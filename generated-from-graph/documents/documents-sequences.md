# Documents - Workflows & Sequences

This document contains sequence diagrams for business workflows in the Documents platform capability.

**Note:** For technical implementation details on malware scanning, blob storage access patterns, and Azure service integrations, see `documents-technical-arch.md`.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

---

## Integration Pattern: Domain API Calling Documents

**What:** A domain API authorizes an upload or download action, then delegates file mechanics to the Documents API  
**When:** Any time a domain feature requires a user to attach, retrieve, or manage a file  
**Who:** Any domain API acting on behalf of an authenticated user (Credentialing, Staffing, Professional Learning, EPP, Audit)
> **This is the canonical integration pattern for all upload and download flows.** The sequences that follow (single upload, bulk upload, staging upload, replacement) are Documents-internal views of what happens after this handoff. When building a domain feature that involves documents, this is the pattern to implement.

```mermaid
---
title: Documents - Integration Pattern: Domain API Calling Documents
---
sequenceDiagram
    actor User
    participant DomainUI as Domain UI (e.g. Credentialing)
    participant DomainAPI as Domain API (e.g. Credentialing API)
    participant DocsAPI as Documents API
    participant BlobStorage as Azure Blob Storage

    User->>DomainUI: Initiate upload (e.g. attach transcript to application)
    DomainUI->>DomainAPI: Request upload token
    Note over DomainUI,DomainAPI: Domain-specific request context

    DomainAPI->>DomainAPI: Validate: user has [domain permission]
    Note over DomainAPI: e.g. credentialing.application.submit<br/>This is the only authorization check for this upload.

    DomainAPI->>DocsAPI: POST /api/documents/upload/request
    Note over DomainAPI,DocsAPI: Authenticated service-to-service call (Managed Identity)<br/>{filename, size, mime_type, category, attachment_type, attachment_id}

    DocsAPI->>DocsAPI: Validate file type and size against category limits
    DocsAPI-->>DomainAPI: {document_id, upload_url}

    DomainAPI-->>DomainUI: {document_id, upload_url}
    DomainUI->>BlobStorage: Upload file via SAS token
```

**Key Decisions:**
- **Single authorization check:** The domain API is the only place business authorization is enforced. Documents trusts authenticated internal callers.
- **Service-to-service auth:** Domain APIs call Documents using Managed Identity. Documents does not accept direct unauthenticated calls from browsers.
- **Technical validation only:** Documents enforces file size, format, and category rules — not business rules about who may upload what.
- **Bulk vs. single is a UX concern only:** There is no separate permission for bulk uploads. If a user is authorized to upload a file for a given purpose, they are authorized to upload multiple files for that same purpose.

**Events Published:**
None — this diagram represents the handoff into Documents; see the per-flow sequences below for what Documents itself publishes.

**Error Scenarios:**
- Domain API authorization fails -> Domain API returns 403 to UI; Documents is never called
- File type not in category allowlist -> Documents returns 400 to domain API; domain API surfaces error to UI
- File size exceeds category limit -> Documents returns 400 to domain API; domain API surfaces error to UI
- Managed Identity auth failure -> Documents returns 401; domain API should alert on this as it indicates a misconfiguration, not a user error

---

## Document Upload with Malware Scanning

**What:** User uploads file, system scans for malware, stores if clean  
**When:** User needs to attach supporting document to credential application, PPR disclosure, or other context  
**Who:** Educator, District Staff, Administrator

```mermaid
---
title: Documents - Document Upload with Malware Scanning
---
sequenceDiagram
    actor User
    participant UI
    participant DocsAPI as Documents API
    participant BlobStorage as Azure Blob Storage
    participant Defender as Azure Defender
    participant EventGrid
    participant MetadataDB as SQL Metadata DB
    participant EventBus

    User->>UI: Select file for upload
    UI->>DocsAPI: POST /api/documents/upload/request
    Note over UI,DocsAPI: {filename, size, mime_type, category, attachment_type, attachment_id}

    Note over DocsAPI: Calling domain has already authorized this action.<br/>Documents validates technical constraints only.
    DocsAPI->>DocsAPI: Validate file type against allowable formats
    DocsAPI->>DocsAPI: Validate file size against category limit

    DocsAPI->>DocsAPI: Generate document_id (UUID)
    DocsAPI->>DocsAPI: Sanitize original filename
    DocsAPI->>DocsAPI: Build blob path
    Note over DocsAPI: documents-{category}/{year}/{month}/<br/>{attachment-type}/{attachment-id}/{doc-id}

    DocsAPI->>MetadataDB: Create Document record
    Note over MetadataDB: Status: Scanning<br/>blob_path, original_filename
    MetadataDB-->>DocsAPI: document_id

    DocsAPI->>BlobStorage: Generate upload SAS token (write-only, 15 min)
    BlobStorage-->>DocsAPI: SAS URL

    DocsAPI-->>UI: 200 OK {document_id, upload_url, status: Scanning}

    UI->>BlobStorage: Upload file directly to blob storage via SAS
    BlobStorage-->>UI: Upload complete
    UI-->>User: "File uploaded, scanning for malware..."

    Note over Defender: Azure Defender detects blob upload<br/>and scans in-place
    Defender->>BlobStorage: Scan blob for malware
    Defender->>BlobStorage: Write blob index tags

    alt Clean Scan Result
        Defender--)EventGrid: MalwareScanningResult {scanResultType: "No threats found"}
        EventGrid->>DocsAPI: Webhook: /api/webhooks/defender-scan-result

        DocsAPI->>MetadataDB: Update Document
        Note over MetadataDB: Status: Available

        DocsAPI--)EventBus: DocumentUploaded
        DocsAPI->>UI: Notification (SSE/webhook)
        UI-->>User: "File uploaded successfully"

    else Malware Detected
        Defender--)EventGrid: MalwareScanningResult {scanResultType: "Malicious", threatName}
        EventGrid->>DocsAPI: Webhook: Malware detected

        DocsAPI->>BlobStorage: Delete infected blob

        DocsAPI->>MetadataDB: Update Document
        Note over MetadataDB: Status: Infected<br/>threat_details

        DocsAPI--)EventBus: DocumentMalwareDetected
        DocsAPI->>UI: Notification (SSE/webhook)
        UI-->>User: "File rejected due to security concerns"

        Note over DocsAPI: Alert security team

    else Scan Timeout/Failure
        Note over DocsAPI: Background job detects no webhook<br/>received within 60 seconds

        DocsAPI->>BlobStorage: Delete blob
        DocsAPI->>MetadataDB: Update Document
        Note over MetadataDB: Status: Scan Failed
        DocsAPI->>UI: Notification (SSE/webhook)
        UI-->>User: "Upload failed, please try again"
    end
```

**Key Decisions:**
- **Browser-based upload:** File uploaded directly from browser to blob storage via SAS token (no proxy through API)
- **Scan in-place:** Azure Defender scans blob in target container (no quarantine/copy required)
- **Async notification:** UI uses SSE or polling for scan result (don't block user during scan)
- **Timeout handling:** Background job marks as `Scan Failed` if no webhook received within 60 seconds

**State Changes:**
- Document status: `None` -> `Scanning` -> `Available` OR `Infected` OR `Scan Failed`

**Events Published:**
- `DocumentUploaded` - Scan clean, document available
- `DocumentMalwareDetected` - Malware found, security alert

**Error Scenarios:**
- Invalid file type -> 400 Bad Request, upload rejected before SAS generation
- File size exceeds limit -> 400 Bad Request
- Blob storage quota exceeded -> 503 Service Unavailable, alert admins
- Scan timeout (>60 seconds) -> Mark as `Scan Failed`, delete blob

---

## Document Download with SAS Token

**What:** User requests document, system generates time-limited access URL  
**When:** User views document from credential application, a filtered pending-items list view, or document library  
**Who:** Educator, District Staff, Administrator

```mermaid
---
title: Documents - Document Download with SAS Token
---
sequenceDiagram
    actor User
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant BlobStorage as Azure Blob Storage
    participant EventBus

    Note over UI,DocsAPI: Caller has already verified the user may access<br/>this document. Documents does not re-check business authorization.

    User->>UI: Click "View Document"
    UI->>DocsAPI: GET /api/documents/{document_id}/download

    DocsAPI->>MetadataDB: Get Document metadata
    MetadataDB-->>DocsAPI: {blob_path, status, original_filename}

    alt Document not available
        DocsAPI-->>UI: 404 Not Found OR 410 Gone (if soft-deleted)
        UI-->>User: "Document not available"
    else Document still scanning
        DocsAPI-->>UI: 202 Accepted {status: Scanning}
        UI-->>User: "Document is being processed, please try again shortly"
    else Document available
        DocsAPI->>BlobStorage: Generate download SAS token (read-only, 15 min)
        BlobStorage-->>DocsAPI: SAS URL

        DocsAPI->>MetadataDB: Log access
        Note over MetadataDB: DocumentAccessLog:<br/>{document_id, user_id, action: view, timestamp}

        DocsAPI--)EventBus: DocumentViewed

        DocsAPI-->>UI: 200 OK {sas_url, original_filename}
        UI->>BlobStorage: Direct download via SAS URL
        BlobStorage-->>UI: Blob content
        UI-->>User: Browser downloads file as original_filename
    end
```

**Key Decisions:**
- **Direct blob access:** UI downloads directly from blob storage using SAS token (no proxy through API)
- **Time-limited tokens:** 15-minute expiry prevents URL sharing
- **Access logging:** All views logged for compliance audit
- **Status check:** Prevent download while still scanning

**Events Published:**
- `DocumentViewed` - Track document access for analytics

**Error Scenarios:**
- Document soft-deleted -> 410 Gone
- Document still scanning -> 202 Accepted (try again)
- SAS token expired -> UI requests new token

---

## Document Replacement and Versioning

**What:** User replaces existing document, old version archived  
**When:** User uploads wrong file or needs to update supporting document  
**Who:** Educator (own documents), Administrator (any document in scope)

```mermaid
---
title: Documents - Document Replacement and Versioning
---
sequenceDiagram
    actor User
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant BlobStorage as Azure Blob Storage
    participant ArchiveContainer as Archive Container
    participant Defender as Azure Defender
    participant EventGrid
    participant EventBus

    User->>UI: Upload new version of document
    UI->>DocsAPI: POST /api/documents/{document_id}/replace/request
    Note over UI,DocsAPI: {filename, size, mime_type}

    DocsAPI->>MetadataDB: Get current Document
    MetadataDB-->>DocsAPI: {current_version, blob_path, category}

    DocsAPI->>DocsAPI: Validate: max 10 versions not exceeded
    DocsAPI->>DocsAPI: Validate: document not soft-deleted

    DocsAPI->>DocsAPI: Generate new document_id for version
    DocsAPI->>DocsAPI: Increment version number (e.g., 1.0 -> 2.0)

    Note over DocsAPI: Archive old version
    DocsAPI->>BlobStorage: Read current blob
    BlobStorage-->>DocsAPI: Blob content
    DocsAPI->>ArchiveContainer: Copy to archive
    Note over ArchiveContainer: document-versions/{original_doc_id}/v1.0

    DocsAPI->>MetadataDB: Create DocumentVersion record
    Note over MetadataDB: {original_document_id, version: 1.0,<br/>archived_blob_path, replaced_at}

    DocsAPI->>MetadataDB: Update original Document
    Note over MetadataDB: current_version = false<br/>soft_deleted = true<br/>replaced_by_version_id = new_doc_id

    Note over DocsAPI: Upload new version (follows standard upload flow)
    DocsAPI->>DocsAPI: Build new blob path
    DocsAPI->>MetadataDB: Create new Document record
    Note over MetadataDB: {document_id: new_doc_id, version: 2.0,<br/>replaces_version_id: original_doc_id,<br/>status: Scanning}

    DocsAPI->>BlobStorage: Generate upload SAS token
    BlobStorage-->>DocsAPI: SAS URL

    DocsAPI-->>UI: 200 OK {new_document_id, upload_url, version: 2.0}

    UI->>BlobStorage: Upload new file via SAS
    BlobStorage-->>UI: Upload complete

    Note over Defender: Scan new version in-place
    Defender->>BlobStorage: Scan blob
    Defender--)EventGrid: MalwareScanningResult

    alt Clean scan
        EventGrid->>DocsAPI: Webhook: Clean
        DocsAPI->>MetadataDB: Update new Document status = Available
        DocsAPI--)EventBus: DocumentReplaced
        DocsAPI->>UI: Notification
        UI-->>User: "Document updated successfully"
    else Malware detected
        EventGrid->>DocsAPI: Webhook: Malware
        DocsAPI->>BlobStorage: Delete infected blob
        DocsAPI->>MetadataDB: Update new Document status = Infected
        DocsAPI->>UI: Notification
        UI-->>User: "Replacement file rejected - original version remains"
        Note over DocsAPI: Original version restored as current
    end
```

**Key Decisions:**
- **New document ID for new version:** Each version is a separate Document record
- **Archive old version:** Old blob moved to archive container, not deleted
- **Version numbering:** Incremental (1.0, 2.0, 3.0) for simplicity
- **Malware scan new version:** Replacement goes through full scan-in-place workflow
- **Rollback on failure:** If new version fails scan, old version remains current

**State Changes:**
- Original Document: `Available` -> `Soft Deleted` (current_version = false)
- New Document: `None` -> `Scanning` -> `Available` OR `Infected`

**Events Published:**
- `DocumentReplaced` - Includes old and new document IDs (only if new version scan clean)

**Error Scenarios:**
- Max 10 versions exceeded -> 400 Bad Request "Maximum versions reached"
- Document soft-deleted -> 400 Bad Request "Restore document before replacing"
- New file infected -> Upload fails, old version remains current

---

## Soft Delete and Hard Delete Lifecycle

**What:** User deletes document, system soft-deletes immediately, hard-deletes after retention period  
**When:** User removes incorrect upload or document no longer needed  
**Who:** Educator (own documents), Administrator (any document in scope)

```mermaid
---
title: Documents - Soft Delete and Hard Delete Lifecycle
---
sequenceDiagram
    actor User
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant EventBus
    participant HardDeleteJob as Hard Delete Job (nightly)
    participant BlobStorage as Azure Blob Storage

    Note over User,UI: SOFT DELETE (User-Initiated)
    User->>UI: Click "Delete Document"
    UI->>DocsAPI: DELETE /api/documents/{document_id}

    DocsAPI->>MetadataDB: Get Document
    MetadataDB-->>DocsAPI: {status, retention_expiry_date, legal_hold}

    DocsAPI->>MetadataDB: Update Document
    Note over MetadataDB: soft_deleted = true<br/>soft_deleted_at = NOW()<br/>soft_deleted_by_user_id = user_id<br/>Status: Soft Deleted

    DocsAPI--)EventBus: DocumentSoftDeleted
    DocsAPI-->>UI: 200 OK
    UI-->>User: "Document deleted"

    Note over MetadataDB: Document hidden from user queries<br/>but blob remains in storage

    Note over HardDeleteJob: HARD DELETE (System-Automated)
    loop Nightly at 2:00 AM EST
        HardDeleteJob->>MetadataDB: Query eligible documents
        Note over MetadataDB: WHERE soft_deleted = true<br/>AND soft_deleted_at + retention_days < NOW()<br/>AND legal_hold = false
        MetadataDB-->>HardDeleteJob: List of eligible documents

        loop For each document (batch 100)
            HardDeleteJob->>BlobStorage: Delete blob
            BlobStorage-->>HardDeleteJob: Success

            HardDeleteJob->>MetadataDB: Update Document
            Note over MetadataDB: hard_deleted = true<br/>hard_deleted_at = NOW()<br/>Status: Hard Deleted<br/>(metadata retained, blob removed)

            HardDeleteJob--)EventBus: DocumentHardDeleted
        end

        HardDeleteJob->>HardDeleteJob: Generate summary report
        Note over HardDeleteJob: Total evaluated, deleted,<br/>skipped (legal hold), errors
    end
```

**Key Decisions:**
- **Soft delete is immediate:** User action hides document instantly
- **Hard delete is automated:** No user permission to force hard delete
- **Metadata preserved:** Even after hard delete, metadata remains for audit

**State Changes:**
- Document status: `Available` -> `Soft Deleted` (user action) -> `Hard Deleted` (job)

**Events Published:**
- `DocumentSoftDeleted` - User deletes document
- `DocumentHardDeleted` - Blob permanently removed (nightly job)

**Error Scenarios:**
- User tries to delete document with legal hold -> 403 Forbidden "Document has active legal hold"
- Hard delete blob fails -> Retry next night (max 3 attempts), alert admin after 3 failures

---

## Legal Hold Application and Release

**What:** Administrator applies legal hold preventing deletion, later releases after legal matter concludes  
**When:** Litigation, investigation, or audit requires document preservation  
**Who:** Legal Counsel, Compliance Officer

```mermaid
---
title: Documents - Legal Hold Application and Release
---
sequenceDiagram
    actor LegalCounsel as Legal Counsel
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant EventBus

    Note over LegalCounsel,UI: APPLY LEGAL HOLD
    LegalCounsel->>UI: Search documents for case
    UI->>DocsAPI: GET /api/documents/search?attachment=application-12345
    DocsAPI->>MetadataDB: Query documents
    MetadataDB-->>DocsAPI: List of documents
    DocsAPI-->>UI: Document list

    LegalCounsel->>UI: Select documents, "Apply Legal Hold"
    UI->>DocsAPI: POST /api/documents/legal-hold/apply
    Note over UI,DocsAPI: {document_ids[], justification,<br/>case_number, hold_reason}

    DocsAPI->>DocsAPI: Validate: user has documents.legal-hold.apply

    loop For each document_id
        DocsAPI->>MetadataDB: Update Document
        Note over MetadataDB: legal_hold = true<br/>legal_hold_applied_at = NOW()<br/>legal_hold_applied_by = user_id<br/>legal_hold_justification = text<br/>legal_hold_case_number = case_number

        DocsAPI--)EventBus: LegalHoldApplied
    end

    DocsAPI-->>UI: 200 OK {applied_count}
    UI-->>LegalCounsel: "Legal hold applied to X documents"

    Note over MetadataDB: Documents cannot be hard-deleted<br/>even if retention period expires

    Note over LegalCounsel,UI: RELEASE LEGAL HOLD (Later)
    LegalCounsel->>UI: View legal hold documents
    LegalCounsel->>UI: Select documents, "Release Legal Hold"
    UI->>DocsAPI: POST /api/documents/legal-hold/release
    Note over UI,DocsAPI: {document_ids[], release_reason,<br/>case_closure_date}

    DocsAPI->>DocsAPI: Validate: user has documents.legal-hold.release

    loop For each document_id
        DocsAPI->>MetadataDB: Update Document
        Note over MetadataDB: legal_hold = false<br/>legal_hold_released_at = NOW()<br/>legal_hold_released_by = user_id<br/>legal_hold_release_reason = text

        DocsAPI--)EventBus: LegalHoldReleased

        alt Document was soft-deleted
            Note over DocsAPI: Document now eligible for hard delete<br/>if retention period expired
        end
    end

    DocsAPI-->>UI: 200 OK {released_count}
    UI-->>LegalCounsel: "Legal hold released for X documents"
```

**Key Decisions:**
- **Legal hold overrides retention:** Documents with hold cannot be hard-deleted regardless of age
- **Justification required:** Must include case number and reason for audit trail
- **Release doesn't delete:** Releasing hold allows retention policy to resume, but doesn't immediately delete

**Events Published:**
- `LegalHoldApplied` - Per document, includes case number
- `LegalHoldReleased` - Per document, includes release reason

**Error Scenarios:**
- Non-legal user attempts hold -> 403 Forbidden
- Release without justification -> 400 Bad Request "Release reason required"

---

## Bulk Document Upload

**What:** Administrator uploads multiple files simultaneously, system processes batch with malware scanning  
**When:** District uploads employee roster CSV, EPP uploads candidate tracking file, Sponsor uploads attendee list  
**Who:** District Admin, EPP Admin, Professional Learning Sponsor

```mermaid
---
title: Documents - Bulk Document Upload
---
sequenceDiagram
    actor Admin
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant BlobStorage as Azure Blob Storage
    participant Defender as Azure Defender
    participant EventGrid
    participant BulkProcessor as Bulk Processor (async)
    participant EventBus

    Admin->>UI: Select multiple files, click "Upload All"
    Note over UI,DocsAPI: Bulk vs. single upload is a UX distinction only.<br/>Authorization is governed by the same source domain permission<br/>that controls single uploads for this context.
    UI->>DocsAPI: POST /api/documents/bulk-upload/request
    Note over UI,DocsAPI: {files: [{filename, size, mime_type}],<br/>category, attachment_type, attachment_id}

    DocsAPI->>DocsAPI: Validate: all files match allowable formats
    DocsAPI->>DocsAPI: Validate: all files within size limits

    DocsAPI->>MetadataDB: Create BulkOperation
    Note over MetadataDB: {operation_type: upload, status: Queued,<br/>total_count: file count,<br/>initiated_by_user_id}
    MetadataDB-->>DocsAPI: operation_id

    loop For each file
        DocsAPI->>DocsAPI: Generate document_id, build blob path
        DocsAPI->>MetadataDB: Create Document (status: Scanning)
        DocsAPI->>MetadataDB: Create BulkOperationItem
        Note over MetadataDB: {operation_id, document_id, filename, status: Queued}
        DocsAPI->>BlobStorage: Generate upload SAS token
    end

    DocsAPI-->>UI: 200 OK {operation_id, upload_urls[]}
    UI-->>Admin: "Upload started, processing X files..."

    loop For each file (parallel, max 10 concurrent)
        UI->>BlobStorage: Upload file via SAS token
        BlobStorage-->>UI: Upload complete

        Defender->>BlobStorage: Scan blob in-place
        Defender--)EventGrid: MalwareScanningResult

        alt Scan clean
            EventGrid->>BulkProcessor: Webhook: Clean
            BulkProcessor->>MetadataDB: Update Document status = Available
            BulkProcessor->>MetadataDB: Update BulkOperationItem status = Succeeded
            BulkProcessor--)EventBus: DocumentUploaded
        else Scan infected or failed
            EventGrid->>BulkProcessor: Webhook: Malware/Error
            BulkProcessor->>BlobStorage: Delete blob
            BulkProcessor->>MetadataDB: Update Document status = Infected/Scan Failed
            BulkProcessor->>MetadataDB: Update BulkOperationItem
            Note over MetadataDB: status: Failed<br/>error_message: scan result
        end

        BulkProcessor->>UI: Progress update (SSE/polling)
        UI-->>Admin: "12 of 50 files processed..."
    end

    BulkProcessor->>MetadataDB: Update BulkOperation
    Note over MetadataDB: status: Completed or Partially Failed<br/>succeeded_count, failed_count

    BulkProcessor--)EventBus: BulkOperationCompleted
    BulkProcessor->>UI: Completion notification
    UI-->>Admin: "Upload complete: 48 succeeded, 2 failed"
```

**Key Decisions:**
- **Browser-based bulk upload:** Each file gets its own SAS token, uploaded directly from browser
- **Scan in-place:** Files scanned in target containers (no quarantine)
- **Parallel uploads:** Max 10 concurrent to avoid overwhelming scan service (2,000 files/min limit)
- **Individual item tracking:** Each file has success/failure status
- **Partial success allowed:** Some files can fail without failing entire operation

**State Changes:**
- BulkOperation status: `Queued` -> `In Progress` -> `Completed` OR `Partially Failed`
- BulkOperationItem status: `Queued` -> `Succeeded` OR `Failed`

**Events Published:**
- `DocumentUploaded` - Per successfully uploaded file
- `BulkOperationCompleted` - Summary of entire operation

**Error Scenarios:**
- Malware detected in 1 file -> That file fails, others continue
- All files fail scan -> Operation status `Failed`
- User cancels mid-upload -> Operation status `Cancelled`, uploaded files remain

---

## Bulk Document Soft Delete with Validation

**What:** Administrator marks multiple documents for soft delete, system validates retention policies  
**When:** Cleanup of obsolete documents, removing test data, or bulk document management  
**Who:** Document Administrator, District Admin (scoped to their documents)

```mermaid
---
title: Documents - Bulk Document Soft Delete with Validation
---
sequenceDiagram
    actor Admin
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant EventBus

    Admin->>UI: Select multiple documents, click "Delete Selected"
    UI->>DocsAPI: POST /api/documents/bulk-delete
    Note over UI,DocsAPI: {document_ids[], mode: permissive}

    DocsAPI->>DocsAPI: Validate: user has documents.document.bulk-delete

    Note over DocsAPI: Pre-validation phase
    DocsAPI->>MetadataDB: Query documents
    MetadataDB-->>DocsAPI: Document list with retention/legal hold status

    DocsAPI->>DocsAPI: Validate retention and legal holds
    Note over DocsAPI: Separate into:<br/>- Eligible (no restrictions)<br/>- Ineligible (legal hold or retention not expired)

    alt Strict mode AND any ineligible
        DocsAPI-->>UI: 400 Bad Request
        Note over UI: "Cannot delete: 2 documents have legal holds"
        UI-->>Admin: Error message with ineligible document list
    else Permissive mode (default)
        DocsAPI->>MetadataDB: Create BulkOperation
        Note over MetadataDB: {operation_type: delete, status: In Progress,<br/>total_count, eligible_count, ineligible_count}
        MetadataDB-->>DocsAPI: operation_id

        loop For each eligible document
            DocsAPI->>MetadataDB: Update Document
            Note over MetadataDB: soft_deleted = true<br/>soft_deleted_at = NOW()<br/>soft_deleted_by_user_id = admin_id

            DocsAPI->>MetadataDB: Create BulkOperationItem
            Note over MetadataDB: {document_id, status: Succeeded}

            DocsAPI--)EventBus: DocumentSoftDeleted
        end

        loop For each ineligible document
            DocsAPI->>MetadataDB: Create BulkOperationItem
            Note over MetadataDB: {document_id, status: Skipped,<br/>skip_reason: legal hold or retention}
        end

        DocsAPI->>MetadataDB: Update BulkOperation
        Note over MetadataDB: status: Completed or Partially Failed<br/>succeeded_count, skipped_count

        DocsAPI--)EventBus: BulkOperationCompleted
        DocsAPI-->>UI: 200 OK {operation_id, succeeded, skipped}
        UI-->>Admin: "98 documents deleted, 2 skipped (legal hold)"
    end
```

**Key Decisions:**
- **Pre-validation required:** Check all retention/legal hold before starting
- **Permissive mode default:** Skip ineligible documents, delete eligible ones
- **Strict mode optional:** Admin can require all-or-nothing deletion

**Events Published:**
- `DocumentSoftDeleted` - Per successfully deleted document
- `BulkOperationCompleted` - Summary with skipped count

**Error Scenarios:**
- All documents have legal holds -> Operation status `Failed` (0 succeeded)
- User lacks scope permission -> 403 Forbidden
- Strict mode with ineligible docs -> 400 Bad Request, operation blocked

---

## Administrator Requests Retention Override

**What:** Admin requests exception to retention policy, business owner approves  
**When:** Need to delete documents before retention expiry (e.g., GDPR request, legal mandate, storage emergency)  
**Who:** Document Administrator (requestor), Business Owner (approver)

```mermaid
---
title: Documents - Administrator Requests Retention Override
---
sequenceDiagram
    actor DocAdmin as Document Admin
    actor BusinessOwner as Business Owner
    participant UI
    participant DocsAPI as Documents API
    participant MetadataDB as SQL Metadata DB
    participant EventBus
    participant NotificationService as Notifications

    Note over DocAdmin,UI: REQUEST OVERRIDE
    DocAdmin->>UI: Select documents blocked by retention
    DocAdmin->>UI: Click "Request Deletion Override"
    UI->>DocsAPI: POST /api/documents/override-requests
    Note over UI,DocsAPI: {document_ids[], justification,<br/>urgency, business_reason}

    DocsAPI->>DocsAPI: Validate: user has documents.retention.override.request
    DocsAPI->>DocsAPI: Determine approval requirements
    Note over DocsAPI: High-value docs (legal hold history,<br/>>5 years retention) = dual approval<br/>Standard = single approval

    DocsAPI->>MetadataDB: Create OverrideRequest
    Note over MetadataDB: {status: Pending, requested_by,<br/>document_ids[], approval_count_required,<br/>justification, expires_at: NOW() + 7 days}
    MetadataDB-->>DocsAPI: request_id

    DocsAPI--)EventBus: OverrideRequestCreated
    DocsAPI->>NotificationService: Notify Business Owner(s)
    NotificationService-->>BusinessOwner: Email: "Override request pending review"

    DocsAPI-->>UI: 200 OK {request_id}
    UI-->>DocAdmin: "Override request submitted, awaiting approval"

    Note over BusinessOwner,UI: APPROVE OVERRIDE
    BusinessOwner->>UI: Review override request
    UI->>DocsAPI: GET /api/documents/override-requests/{request_id}
    DocsAPI->>MetadataDB: Get OverrideRequest with documents
    MetadataDB-->>DocsAPI: Request details
    DocsAPI-->>UI: Request + document list + retention info

    BusinessOwner->>UI: Review justification, click "Approve"
    UI->>DocsAPI: POST /api/documents/override-requests/{request_id}/approve
    Note over UI,DocsAPI: {approval_comments}

    DocsAPI->>DocsAPI: Validate: user has documents.retention.override.approve

    DocsAPI->>MetadataDB: Create OverrideApproval
    Note over MetadataDB: {request_id, approved_by, approved_at,<br/>comments}

    DocsAPI->>MetadataDB: Update OverrideRequest
    Note over MetadataDB: approval_count += 1

    alt Sufficient approvals received
        DocsAPI->>MetadataDB: Update OverrideRequest
        Note over MetadataDB: status: Approved
        DocsAPI--)EventBus: OverrideRequestApproved
        DocsAPI->>NotificationService: Notify requester
        NotificationService-->>DocAdmin: Email: "Override approved, execute within 7 days"
    else More approvals needed (dual approval)
        DocsAPI->>NotificationService: Notify second approver
        Note over DocsAPI: Awaiting second approval
    end

    DocsAPI-->>UI: 200 OK
    UI-->>BusinessOwner: "Override approved"

    Note over DocAdmin,UI: EXECUTE OVERRIDE (Within 7 days)
    DocAdmin->>UI: View approved override, click "Execute"
    UI->>DocsAPI: POST /api/documents/override-requests/{request_id}/execute

    DocsAPI->>MetadataDB: Get OverrideRequest
    MetadataDB-->>DocsAPI: {status: Approved, document_ids[], expires_at}

    DocsAPI->>DocsAPI: Validate: not expired (< 7 days since approval)

    loop For each document_id
        DocsAPI->>MetadataDB: Force soft delete (bypass retention check)
        Note over MetadataDB: soft_deleted = true<br/>deletion_override_request_id = request_id
        DocsAPI--)EventBus: DocumentSoftDeleted {override: true}
    end

    DocsAPI->>MetadataDB: Update OverrideRequest
    Note over MetadataDB: status: Executed, executed_at, executed_by

    DocsAPI--)EventBus: OverrideRequestExecuted
    DocsAPI-->>UI: 200 OK
    UI-->>DocAdmin: "Override executed, documents deleted"
```

**Key Decisions:**
- **Dual approval for high-value:** Documents with legal hold history or >5 year retention require 2 approvals
- **7-day execution window:** Approved overrides expire if not executed within 7 days
- **Justification mandatory:** All overrides require business reason for audit trail
- **Cannot be reversed:** Once executed, deletion is permanent (standard retention applies to hard delete)

**State Changes:**
- OverrideRequest: `Pending` -> `Approved` -> `Executed`
- Documents: Bypass retention, immediate soft delete eligibility

**Events Published:**
- `OverrideRequestCreated` - Admin submits request
- `OverrideRequestApproved` - Business owner approves
- `OverrideRequestExecuted` - Admin executes deletion
- `DocumentSoftDeleted` - Per document, flagged as override deletion

**Error Scenarios:**
- Override expired (>7 days) -> 400 Bad Request "Override expired, request new approval"
- Insufficient approvals -> 403 Forbidden "Awaiting second approval"
- Request already executed -> 409 Conflict "Override already executed"
- Approver is same as requester -> 403 Forbidden "Cannot approve own request"

---

## Staged File Upload for Synapse Processing

**What:** User uploads bulk import file to staging area, Synapse pipeline processes  
**When:** District uploads employee roster CSV, EPP uploads candidate enrollment file  
**Who:** District Admin, EPP Admin

```mermaid
---
title: Documents - Staged File Upload for Synapse Processing
---
sequenceDiagram
    actor Admin
    participant UI
    participant DocsAPI as Documents API
    participant BlobStorage as Staging Container
    participant Defender as Azure Defender
    participant EventGrid
    participant MetadataDB as SQL Metadata DB
    participant EventBus
    participant Synapse as Azure Synapse
    participant DataWarehouse as Data Warehouse

    Admin->>UI: Upload bulk import file (CSV/XLSX)
    UI->>DocsAPI: POST /api/documents/staging/upload/request
    Note over UI,DocsAPI: {filename, size, mime_type,<br/>functional_area: staffing,<br/>import_type: employee_roster}

    DocsAPI->>DocsAPI: Validate: file format (CSV, XLSX only)
    DocsAPI->>DocsAPI: Validate: file size (max 500 MB)

    DocsAPI->>DocsAPI: Generate upload_id
    DocsAPI->>DocsAPI: Build staging path
    Note over DocsAPI: documents-bulk-import-staging-staffing/<br/>{upload_id}_{timestamp}_{filename}

    DocsAPI->>MetadataDB: Create StagingFile record
    Note over MetadataDB: {upload_id, blob_path, functional_area,<br/>import_type, status: Scanning,<br/>uploaded_by, uploaded_at}

    DocsAPI->>BlobStorage: Generate upload SAS token
    BlobStorage-->>DocsAPI: SAS URL

    DocsAPI-->>UI: 200 OK {upload_id, upload_url, status: Scanning}

    UI->>BlobStorage: Upload file via SAS token
    BlobStorage-->>UI: Upload complete
    UI-->>Admin: "File uploaded, scanning..."

    Note over Defender: Scan staging file in-place
    Defender->>BlobStorage: Scan blob for malware
    Defender--)EventGrid: MalwareScanningResult

    alt Scan clean
        EventGrid->>DocsAPI: Webhook: Clean
        DocsAPI->>MetadataDB: Update StagingFile status = Uploaded
        DocsAPI--)EventBus: BulkFileUploaded
        Note over EventBus: {upload_id, functional_area,<br/>import_type, blob_path}
        DocsAPI->>UI: Notification
        UI-->>Admin: "File uploaded, processing will begin shortly"

        Note over Synapse: Synapse subscribes to BulkFileUploaded
        EventBus->>Synapse: BulkFileUploaded event

        Synapse->>Synapse: Trigger appropriate pipeline
        Note over Synapse: Based on functional_area + import_type

        Synapse->>BlobStorage: Read file (using Managed Identity)
        BlobStorage-->>Synapse: File content

        Synapse->>Synapse: Parse and validate data

        alt Processing successful
            Synapse->>DataWarehouse: Insert/update records
            Synapse--)EventBus: BulkFileProcessed
            Note over EventBus: {upload_id, status: Success,<br/>records_processed}

            EventBus->>DocsAPI: BulkFileProcessed event
            DocsAPI->>MetadataDB: Update StagingFile
            Note over MetadataDB: status: Processed,<br/>processed_at, records_count

            DocsAPI->>UI: Notification (webhook/SSE)
            UI-->>Admin: "Import complete: 150 records processed"

        else Processing failed
            Synapse--)EventBus: BulkFileProcessed
            Note over EventBus: {upload_id, status: Failed,<br/>error_details}

            EventBus->>DocsAPI: BulkFileProcessed event
            DocsAPI->>MetadataDB: Update StagingFile
            Note over MetadataDB: status: Failed,<br/>error_message

            DocsAPI->>UI: Notification
            UI-->>Admin: "Import failed: invalid data format on row 23"
        end
    else Scan infected/failed
        EventGrid->>DocsAPI: Webhook: Malware/Error
        DocsAPI->>BlobStorage: Delete infected/failed blob
        DocsAPI->>MetadataDB: Update StagingFile
        Note over MetadataDB: status: Scan Failed / Infected
        DocsAPI->>UI: Notification
        UI-->>Admin: "Upload failed due to security concerns"
    end

    Note over BlobStorage: Auto-cleanup after 7 days
    loop Daily cleanup job
        DocsAPI->>MetadataDB: Query old staging files
        Note over MetadataDB: WHERE uploaded_at < NOW() - 7 days
        MetadataDB-->>DocsAPI: Old file list

        loop For each old file
            DocsAPI->>BlobStorage: Delete blob
            DocsAPI->>MetadataDB: Update StagingFile
            Note over MetadataDB: blob_deleted: true
        end
    end
```

**Key Decisions:**
- **All staging files scanned:** No exceptions for "trusted internal sources"
- **Browser-based upload:** File uploaded via SAS token directly to staging container
- **Scan in-place:** Defender scans staging files where they sit
- **Event-driven decoupling:** Documents domain doesn't know about Synapse pipelines
- **Synapse reads directly:** Synapse uses Managed Identity to access staging containers
- **Auto-cleanup:** Files deleted after 7 days regardless of processing status

**State Changes:**
- StagingFile status: `Scanning` -> `Uploaded` -> `Processed` OR `Failed`

**Events Published:**
- `BulkFileUploaded` - File scanned clean, ready for processing
- `BulkFileProcessed` - Synapse publishes after processing (consumed by Documents)

**Error Scenarios:**
- Invalid file format -> 400 Bad Request before upload
- Malware detected -> File deleted, status `Infected`
- Synapse pipeline failure -> Status `Failed`, file remains for retry
- Synapse timeout (>30 min) -> Alert admins, file remains for manual investigation
