# Documents sequences: change log

Doc edited: design/working-docs/solution-areas/documents/documents-sequences.md
Diagrams before: 10. Diagrams after: 16. Splits: 4 (upload, delete, override, staged file).

## Conventions block

Actor is now humans only; the participant line covers every non-human. Added the tag bullet (APP, SVC, EXT, OUT), the box grouping bullet, and the standing sentence on authorization through the cached IAM check. All diagrams now use Mermaid `box` grouping (Browser, MiEdWorkforce (AKS), External). The docs renderer must support Mermaid 10 or later.

## Sections changed

- Integration Pattern: Domain API Calling Documents. Redrawn with tags and boxes; the "Validate: user has [domain permission]" self arrow and its note removed (authorization not drawn); Who gains the Permission line. Key Decisions and Error Scenarios text changed from "Managed Identity" to in-cluster mTLS with service account identity (3.4 item 8). Section left in place (3.3 suggests patterns-and-principles as the home; see Questions).
- Document Download with SAS Token. UI now calls a Domain API (APP), which calls Documents (SVC); caller-already-verified note removed; MetadataDB removed; the three alt branches reduced to two (unavailable or scanning, available) with the status codes on one response arrow. Permission line added. Text blocks unchanged.
- Document Replacement and Versioning. Starts at the replace request through the Domain API; MetadataDB, Archive Container, Defender and EventGrid removed; archive shown as one self step; scan handling referenced to Process Malware Scan Result; single alt for max versions or soft-deleted. The malware branch moved to Error Scenarios (wording of the diagram note appended there). Permission line added.
- Legal Hold Application and Release. Kept as one diagram; MetadataDB removed, permission self arrows removed, soft-deleted alt and DB notes removed (the "eligible for hard delete" note is already covered by the Release doesn't delete decision). Notes reduced from 7 to 4 (including 2 banners and 2 body notes). Who gains Permission line. No alt remains.
- Bulk Document Soft Delete with Validation. MetadataDB removed, permission self arrow removed; Who line now carries `documents.admin.bulk-delete`. The doc's old key `documents.document.bulk-delete` was wrong against the spec and permissions catalog; corrected on the Who line. Single alt (strict versus permissive) kept.
- Bulk Document Upload. See Splits and Questions (kept as a variant, not merged).

## Splits

- Document Upload with Malware Scanning, into Request Upload and Upload File, and Process Malware Scan Result (diagram titles prefixed with "Documents - "). Scan timeout alt moved to Error Scenarios of the second section. Defender and Event Grid merged into one scan callback (Defender participant calls a distinct Scan Webhook Endpoint, tagged EXT).
- Soft Delete and Hard Delete Lifecycle, into Soft Delete Document, and Hard Delete Expired Documents (Nightly Job). Nested nightly and batch loops flattened to one loop with the schedule in a note.
- Administrator Requests Retention Override, into Request Retention Override, Approve Retention Override, Execute Approved Override. Email sends to Notifications removed; replaced with notes "Consumed by Communications (sends email to ...)" on the event arrows. Text blocks distributed by relevance across the three sections.
- Staged File Upload for Synapse Processing, into Stage and Scan Import File, Process Staged File in Synapse, Staging File Cleanup Job. Scan handling in the first is referenced to Process Malware Scan Result rather than redrawn; MetadataDB, Defender and EventGrid removed.
- Not split but reworked: Bulk Document Upload (3.1 row 17 asked to merge it into the single upload). I kept the section as a variant titled "Documents - Request Upload and Upload File - Bulk Variant", starting at the divergence, because bulk has its own operation tracking and endpoint. The base upload section cross references it via See also. Lead to decide whether to delete the section.

## Sections left alone and why

- None of the sections were untouched; all had diagrams that needed tags, boxes or DB removal. Text blocks of unchanged flows (Key Decisions, State Changes, Events, Error Scenarios) were not reworded except where noted (mTLS wording, the Replacement malware scenario, split distribution, and an added expiry error line in Request Retention Override).

## Missing operations (arrows with no operationId in any spec)

Tag, verb and path, section:
- Domain API placeholder routes, all APP, because the domain area owns them and the caller is generic: `APP POST /{domain}/{upload-route}` (Integration Pattern, Request Upload and Upload File), `APP GET /{domain}/{download-route}` (Download), `APP POST /{domain}/{replace-route}` (Replacement), `APP DELETE /{domain}/{document-route}` (Soft Delete Document), `APP POST /{domain}/{bulk-upload-route}` (Bulk Document Upload), `APP POST /{domain}/{staging-upload-route}` (Stage and Scan Import File). Each domain needs its own operation in its spec.
- `EXT POST /webhooks/defender-scan-result` exists as receiveScanResult (internal-service), so not missing, but see API kind observations.
- `OUT` arrows to Azure Blob Storage (PUT with SAS URL, GET with SAS URL, DELETE blob), Staging Container, Data Warehouse and Synapse READ: platform calls, not in any spec by nature.
- Untagged internal handoff `ScanHook to DocsApi: Apply scan result` (Process Malware Scan Result): not an API.

## API kind observations

- receiveScanResult (`/webhooks/defender-scan-result`) is marked `internal-service` in the spec, but its caller is Azure Defender through Event Grid from outside the mesh. I drew it as `EXT` with a separate Scan Webhook Endpoint participant, per the architecture (webhooks are External APIs). The spec text says it is secured by Event Grid signature validation, not mTLS, which supports EXT.
- Operations marked both external and application: none in the Documents spec.
- Documents operations marked `internal-service` (requestDocumentUpload, downloadDocument, requestDocumentReplacement, softDeleteDocument, initiateBulkUpload, uploadToStaging, getBulkOperationStatus) are drawn as SVC from a Domain API, which agrees with the spec.
- searchDocuments is `internal-service`, but Legal Hold draws the Legal Counsel UI calling it directly (`APP GET /documents/search`). Either search also needs an internal-user variant, or Legal Hold search should go through a domain API. Kept as drawn in the original with the APP tag.
- getOverrideRequest, listOverrideRequests, approveOverrideRequest and executeOverrideRequest are `internal-user` and drawn APP from the UI directly, which agrees.
- getBulkOperationStatus is `internal-service`, but the bulk sequence shows progress pushed to the UI (SSE or polling). If the UI polls, it needs an application-facing path.

## Questions

1. Should Integration Pattern: Domain API Calling Documents move to patterns-and-principles (3.3), linked from here? I kept it in place.
2. Bulk Document Upload: merge into single upload (3.1 row 17) or keep as a variant as done here?
3. The spec marks the scan webhook internal-service; confirm it should be EXT (External API) as drawn, and whether the webhook endpoint is a separate service from the Documents API (I drew a Scan Webhook Endpoint participant plus an untagged handoff to the Documents API).
4. Tag for browser or Azure platform calls to Blob Storage, Staging Container, Synapse and the Data Warehouse: none of APP, SVC, EXT, OUT fits exactly. I used OUT. Confirm, or add a fifth tag.
5. Should Azure Synapse, Staging Container, Data Warehouse and Azure Blob Storage sit in the External box? They are Azure platform services, not entries in solution-integrations.md.
6. Events used in the sequences but missing from the capability Domain Events table: OverrideRequestCreated, OverrideRequestExecuted, BulkFileUploaded, BulkFileProcessed (those two appear in the event log entity but not the table). The table has events not drawn: DocumentCategoryCreated, RetentionPolicyUpdated, DocumentArchivedToBlob (no sequence for them).
7. The second approver notification in Approve Retention Override had a direct Notifications call; there is no event for it. I left a note ("second approver is notified by Communications"). Which event carries it?
8. Execute override: the spec lists `documents.retention-override.request` as the permission for executeOverrideRequest; the old doc had no permission shown. Confirm.
9. Download, Replace and Soft Delete Permission lines point to the calling domain's permission (implicit baseline). The permissions catalog has `documents.admin.view-all` only for cross-context viewing; confirm no explicit Documents permission is intended for replace or soft delete.
10. Removed from diagrams but kept in prose only: the 60 second scan timeout branch and the "Document was soft-deleted, eligible for hard delete" note. The timeout job participant is not drawn anywhere; should it have its own diagram?
11. I replaced the Bulk Processor participant with the Documents API, since the webhook endpoint now owns scan callbacks. Confirm Bulk Processor is not a separate deployable.
12. Process Malware Scan Result has a second alt candidate (scan failed or timeout) that I left in Error Scenarios per the one-alt rule.
13. Hard Delete Expired Documents uses the Documents service internal job as a participant; no API operation exists for it, correct as no endpoint is needed.
