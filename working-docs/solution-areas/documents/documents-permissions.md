# Documents - Permissions Catalog

This document defines all atomic permissions for the Documents platform capability.

---

## Permissions

| Category                | Permission ID                          | Description                                                                        | Applicable Scopes | Notes                                                                                             |
| ----------------------- | -------------------------------------- | ---------------------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------- |
| **Legal Hold**          | `documents.legal-hold.apply`           | Apply legal hold to one or more documents                                          | System-wide       | Legal Counsel, Compliance Officer; requires justification; audit logged                           |
| **Legal Hold**          | `documents.legal-hold.release`         | Release legal hold from document(s)                                                | System-wide       | Legal Counsel, Compliance Officer; requires case closure justification; audit logged              |
| **Legal Hold**          | `documents.legal-hold.view`            | View documents with active legal holds                                             | System-wide       | Legal Counsel, Compliance Officer, Document Admin                                                 |
| **Retention Override**  | `documents.retention-override.request` | Request early deletion before retention expiry                                     | System-wide       | Document Admin; requires business justification; audit logged                                     |
| **Retention Override**  | `documents.retention-override.approve` | Approve a retention override request                                               | System-wide       | Business Owner, Compliance Officer; dual approval required for high-value documents; audit logged |
| **Admin Operations**    | `documents.admin.view-quarantine`      | View documents flagged by malware scanner                                          | System-wide       | Security Admin, Document Admin; regular users receive only a generic upload failure message       |
| **Admin Operations**    | `documents.admin.override-scan`        | Mark a quarantined document as a false positive                                    | System-wide       | Security Admin only; requires justification; audit logged                                         |
| **Admin Operations**    | `documents.admin.force-hard-delete`    | Immediately hard-delete a document outside the normal retention cycle              | System-wide       | System Admin only; requires approved override request; irreversible; audit logged                 |
| **Admin Operations**    | `documents.admin.bulk-delete`          | Soft-delete multiple documents in a single batch operation                         | System-wide       | Document Admin; retention and legal hold status validated per item before executing               |
| **Admin Operations**    | `documents.admin.view-all`             | View and download any document regardless of attachment context                    | System-wide       | Document Admin, Compliance Officer; for audit and compliance investigations                       |
| **Category Management** | `documents.category.view`              | View document category configurations                                              | System-wide       | Document Admin, Compliance Officer                                                                |
| **Category Management** | `documents.category.create`            | Create a new document category                                                     | System-wide       | System Admin, Document Admin                                                                      |
| **Category Management** | `documents.category.edit`              | Modify category configuration (display name, allowable formats, file size limit)   | System-wide       | System Admin, Document Admin; retention period and storage tier changes are backend config only   |
| **Category Management** | `documents.category.deprecate`         | Mark a category as deprecated; prevents new uploads, existing documents unaffected | System-wide       | System Admin only                                                                                 |
| **Audit & Reporting**   | `documents.audit.view`                 | View and export document access logs and audit trail                               | System-wide       | Document Admin, Compliance Officer, Auditor                                                       |
| **Audit & Reporting**   | `documents.reports.view`               | View all document reports (storage usage, upload activity, retention compliance)   | System-wide       | Scope-aware reporting                                                                             |

---

## Scope Definitions

**System-wide:** Permission applies across the entire system with no organizational scoping. All documents permissions are system-wide as document lifecycle management and compliance controls are not organizational-scope-filtered.

**Self-only:** Implicit baseline capabilities (see below) apply only to documents the user personally uploaded.

---

## Implicit Baseline (All Authenticated Users)

The following capabilities are available to any authenticated user without an explicit permission grant. They are enforced by the Documents capability automatically, but only after the calling domain or UI has already established that the user is authorized to perform the action that requires the document.

> **Important:** Documents does not independently verify whether a user "owns" an attachment or has the right to upload to a given context. That determination belongs entirely to the calling domain. These baseline capabilities describe what Documents will honor once a legitimate, authenticated request arrives.

| Capability                       | Notes                                                                                                                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| View and download a document     | Documents generates a time-limited SAS token; assumes the caller has already verified access rights                                                                                         |
| Replace an uploaded document     | Documents enforces technical constraints only (max versions, not soft-deleted, format/size); calling domain or UI is responsible for determining whether the user may replace this document |
| Soft-delete an uploaded document | Blob preserved until retention period expires; admin can restore before hard delete occurs                                                                                                  |

---

## Upload Permission Model

Documents is platform infrastructure. It owns the *mechanics* of file storage, scanning, and lifecycle management — not the business rules about *when* or *why* a file may be uploaded.

**Upload access is governed by the source domain.** If a user has the right to perform an action that requires a document (submitting a credential application, uploading a staffing roster, responding to an audit finding), that domain permission is the gate. The documents capability then enforces upload mechanics: file size limits, format validation, and malware scanning.

Viewing documents follows the same principle — if you can view a credential application, you can view its attachments. Documents does not maintain a parallel permission layer for viewing files within normal workflow contexts.

| Functional Area       | Upload Context                       | Controlling Permission             | Domain        |
| --------------------- | ------------------------------------ | ---------------------------------- | ------------- |
| Credentialing         | Supporting documents for application | `credentialing.application.submit` | credentialing |
| Professional Practice | Court records for self-disclosure    | `profpractice.disclosure.submit`   | profpractice  |
| Staffing              | Employee roster bulk import          | `staffing.roster.upload`           | staffing      |
| Professional Learning | Program agenda for SCECH session     | `proflearning.session.edit`        | proflearning  |
| Professional Learning | Attendee list bulk import            | `proflearning.attendees.upload`    | proflearning  |
| EPP                   | Candidate tracking bulk import       | `epp.candidates.upload`            | epp           |
| Audit                 | Additional documentation response    | `audit.finding.respond`            | audit         |

> **Note:** Storage tier transitions, malware scan engine configuration, and retention period durations are managed via backend configuration (Azure App Configuration / infrastructure-as-code) and are not exposed as application permissions.