# EPP sequences: change log

Document: design/working-docs/solution-areas/epp/epp-sequences.md
Style guide: design/parity-check/sequences-standardization.md sections 2 and 3.

Counts: 26 sections and diagrams before, 29 after (4 splits adding 5 sections, 1 merge removing 2 sections, net plus 3).

## Changes applied to every diagram

- Conventions block updated: Actor is humans only, participant is everything else, arrows carry APP, SVC, EXT or OUT tags, box grouping explained, standing authorization sentence added.
- All UI to IAM permission check arrows and IAM participants removed (disclosure 3.4 row 1). Permission keys moved to the Who line as "Permission: <key> (scope note)".
- UI participant is now alias UI, label "UI" (was "EPP UI" and "EPP Admin UI").
- Participants declared with box Browser, box MiEdWorkforce (AKS). Canonical names: EPP API, Organizations API, Credentialing API, IAM API, PPR API, Documents API, Event Bus. Aliases EppApi, OrgApi, CredApi, IamApi, PprApi, DocsApi, EventBus.
- Every API request arrow tagged APP or SVC with the verb and path of the matching operationId in the specs (no /api prefixes, no invented paths).
- Plain return arrows and "UI to actor confirmation" arrows removed. Responses kept only where they carry data or an error status code.
- Event consumers in other areas (Communications emails, Credentialing reactions) are no longer drawn as arrows or participants. Each is a Note under the event: "Consumed by ...".
- Each event is drawn once as an async arrow (BulkUploadCompleted and CandidateStatusChanged were drawn in several branches).
- One alt per diagram. Nested error branches moved to the existing Error Scenarios text (no new error text was needed, all were already listed).
- Consecutive "persist" self arrows merged into one self arrow (max 3 self arrows per diagram now).
- "Application #" changed to "Application Number" inside two Notes (the hash character is not valid in Mermaid message text).

## Sections changed (title, what changed)

- Designate EEM Organization as EPP: UI calls Organizations API (was "Organization Reference Data API"); EPP API calls it as SVC. Alt kept for org not found (400). Permission epp.provider.create.
- Update EPP Configuration: same participant and tag changes. Permission epp.provider.edit.
- Configure Pro Prep Catalog Visibility: tags, IAM removed. Permission epp.proprepconfig.manage.
- Accept Candidate Enrollment: Communications API participant removed, consumer Note added. Permission epp.enrollment.verify.
- Reject Candidate Enrollment: same. Permission epp.enrollment.verify.
- View Candidate Enrollment Detail: UI to IAM now APP GET /users/{userId} (getUser), Credentialing path /educators/{educatorId}/credentials. IAM permission check removed. Permission epp.enrollment.view.
- Update Candidate Enrollment Status: one diagram kept. Business Rule Engine participant removed (rules now self arrow on EPP API). Nested validation alts collapsed to one alt. New "Status Rules" table added after Key Decisions (built only from the existing Key Decisions and Error Scenarios). Permission epp.enrollment.edit.
- Exit Candidate from Program: path fixed to PATCH /candidate-enrollments/{enrollmentId} (was /status). Permission epp.enrollment.edit.
- Search Credential Applications for Review: EPP API calls Credentialing API as SVC. Permission epp.applications.view.
- View Credential Application Detail for EPP Review: PPR call now APP GET /disclosures (listDisclosures with educatorId filter, was /educators/{id}/disclosures). Permission epp.applications.view.
- Place Credential Application on Hold, Deny Credential Application, Cancel Credential Application Review: IAM, Communications API and Credentialing API participants removed, consumer Notes added. Permission epp.applications.review.
- Add Internal Remarks to Application Review: tags only. Permission epp.applications.review.
- Search Approval Applications for Review, View Approval Application Detail for EPP Review: tags, IAM removed. Permission epp.approvals.view. In the view diagram the organization read is APP GET /organizations/{organizationCode} on Organizations API.
- Recommend Approval Application, Deny Approval Application: consumer Notes, IAM removed. Permission epp.approvals.review.

## Splits and merges (old title to new titles)

- Manage EPP Certificate Category Approvals to "Add EPP Certificate Category Approval" and "Remove EPP Certificate Category Approval". Edit described as a text bullet under Key Decisions of the Add section (3.1 row 21, same treatment as row 7).
- Manage EPP Approved Endorsements to "Add EPP Approved Endorsement" and "Remove EPP Approved Endorsement". Edit as text bullet in the Add section.
- Manage Candidate Programs to "Add Candidate Program" (duplicate check) and "Remove Candidate Program" (status rules). Edit as text bullet in the Add section (3.1 row 7).
- Bulk Upload Candidate Tracking Data to "Upload Tracking File" and "Process Tracking File" (3.1 row 23). UI no longer calls Documents API (3.4 row 9): UI calls EPP API initiateBulkUpload, EPP API calls Documents API as SVC.
- Recommend Candidate for Credential to "Load Recommendation Context" (takes the original place) and "Recommend Candidate for Credential" (3.1 row 4). UI no longer fans out to Credentialing API for this flow: the EPP API reads from it as SVC. Business Rule Engine removed, rules to a "Recommendation Rules" table. Alt reduced to the MTTC warning, the other validation failures are in Error Scenarios.
- Merge (3.3 last row, read-only flows): "Search Pending Enrollment Verifications", "Search Enrolled Candidates" and "View Cross-EPP Enrollment History" merged into "Search and View Candidate Enrollments" (these call no other area). Key Decisions and Error Scenarios of the three were combined, no text dropped. Read-only sections that call other areas were kept (View Candidate Enrollment Detail, both credential and approval searches and views).
- Each new section has the full section shape and a "See also:" line.

## Sections left alone and why

- None left completely untouched; every section had IAM arrows, untagged arrows or undeclared box grouping. Text blocks (What, When, Key Decisions, State Changes, Events Published, Error Scenarios) were left as written except where a split or merge required dividing them, plus the added Permission text on the Who line and the new tables noted above.
- Sections 3.2 had no EPP rows.
- No EPP section is inbound external-facing, so no EXT endpoint participant was needed. The public Pro Prep catalog operations (searchProPrepProviders, getProPrepProviderDetail, external-public) have no sequence in this document.

## Missing operations (arrows with no operationId in any spec)

| Area | Tag | Verb and path | Section |
|---|---|---|---|
| EPP | APP | GET /candidate-enrollments/{enrollmentId}/available-transitions | Update Candidate Enrollment Status |
| EPP | APP | GET /epp-providers/{eppCode}/approved-endorsements | Load Recommendation Context |
| Credentialing | APP | GET /applications/{applicationId}/mttc-results | View Credential Application Detail for EPP Review |
| Credentialing | SVC | GET /applications/{applicationId}/mttc-results | Load Recommendation Context, Recommend Candidate for Credential |
| Credentialing | SVC | GET /applications/{applicationId}/requested-endorsements | Load Recommendation Context |
| Credentialing | SVC | GET /applications/approvals | Search Approval Applications for Review |
| Credentialing | APP | GET /applications/approvals/{applicationId} | View Approval Application Detail for EPP Review |
| Documents | SVC | upload of the Synapse error report to staging (no verb and path in the source, uploadToStaging is for import files) | Process Tracking File |

Path adjustments made to match existing operations: PATCH /candidate-enrollments/{enrollmentId}/status became PATCH /candidate-enrollments/{enrollmentId} (updateCandidateStatus). The PUT and DELETE paths for certificate categories, endorsements and candidate programs lost the trailing {id} because the spec carries the id in the body or a query parameter. GET /educators/{candidateId}/disclosures became GET /disclosures (listDisclosures, educatorId filter). GET /users/{candidateId} became GET /users/{userId}.

## API kind observations

- No EPP operation is marked both external and application. getEducatorCredentials (Credentialing) and the Organizations operations are marked "internal-user, internal-service": the UI calls them as APP in View Candidate Enrollment Detail and View Approval Application Detail, and the EPP API calls the Organizations operations as SVC. Both uses are consistent with the dual marking.
- getApprovedPrograms (GET /epp-providers/{eppCode}/approved-programs) is x-access internal-service but the UI calls it (APP) in Add Candidate Program. Synapse calls it as SVC in Process Tracking File. Suggests internal-user and internal-service.
- listApplications (Credentialing, internal-user) is called by the EPP API as SVC in Search Credential Applications for Review. getUser (IAM, internal-user) and getApplication (Credentialing, internal-user) are called by the UI as APP, consistent. If EPP API is to call internal-user operations of other areas on its own behalf, those operations need an internal-service form (or the EPP API forwards the user token, which the sequence does not show).
- getOrganization was tagged SVC from the EPP API (internal-user, internal-service in the Organizations spec, consistent). The EPP spec's getEppProvider and similar internal-user operations are only called by the UI.
- completeBulkUpload, bulkCreateEnrollments (EPP), getUserByUniqueId (IAM), uploadToStaging (Documents) are internal-service and called as SVC, consistent. The EPP spec describes completeBulkUpload and bulkCreateEnrollments as "Managed Identity / mTLS"; the architecture says mTLS and service accounts.
- getCandidateEnrollmentStatus (EPP, internal-service) is not used in any EPP sequence.

## Questions

1. Permission keys differ between the old IAM checks and the spec: Search Pending Verification checked epp.enrollment.verify (spec search operation needs epp.enrollment.view, both are in the Who line); Search and View Credential Applications and View Credential Application Detail checked epp.applications.review (spec: epp.applications.view); Recommend Candidate checked epp.applications.review (spec: epp.applications.recommend); Search and View Approval Applications checked epp.approvals.review (spec: epp.approvals.view). The Who lines now use the spec keys. Confirm.
2. Read steps inside a gated section need a different key from the gating action (for example getEppProvider needs epp.provider.view inside Update EPP Configuration). Only the action key is on the Who line. Confirm whether read keys should also be listed.
3. Business Rule Engine is not defined in solution-architecture.md. It was removed as a participant and treated as logic inside the EPP API. Confirm.
4. Organization source: the diagrams called "Organization Reference Data API" and the text still says org-reference-data and EEM. Confirm that all EPP reads of organizations go through Organizations API (3.4 row 12) and whether the Key Decisions wording "org-reference-data" should change.
5. Load Recommendation Context shows the EPP API reading requested endorsements and MTTC results from Credentialing and returning them to the UI. The EPP spec has no operation that returns these (getApplicationReview is the closest). Should EPP expose them, or should the UI call Credentialing directly (as the other EPP view diagrams still do, APP)? UI fan-out to Credentialing API, IAM API, PPR API and Organizations API remains in View Candidate Enrollment Detail, View Credential Application Detail, View Approval Application Detail and Add EPP Approved Endorsement (endorsement-definitions). The style guide does not say whether these stay.
6. Upload Tracking File follows the EPP spec (initiateBulkUpload receives the file, EPP API stages it through Documents API). The old diagram had the UI get a SAS token from Documents, upload to blob, and Documents call EPP. The old SAS upload and Documents' BulkFileUploaded event (Documents spec: published on clean scan, triggers the pipeline) are not drawn. Who triggers Synapse (EPP API or the Documents event) needs a decision. The EPP-to-Synapse trigger is drawn as an untagged async arrow and Synapse sits in its own box "Synapse (Azure)", which is not in the style guide.
7. listApplications in the Credentialing spec has no eppCode filter, and the spec has no way to list approval applications by EPP. Search Credential Applications for Review and Search Approval Applications for Review assume one.
8. The View Candidate Enrollment Detail and View Credential Application Detail diagrams read demographics (SSN, DOB) through IAM getUser, which requires iam.user.view. The EPP permissions doc does not carry that permission. Confirm intended access for EPP coordinators.
9. The old Recommend diagram showed the MTTC check twice (a UI read and an EPP API read). Kept the EPP API read only, in the Recommend section.
10. Deleting or editing certificate categories, endorsements and programs: spec paths have no {id}; the diagrams show the spec path. Confirm the docs should follow the spec.
11. Event consumers in other areas are now Notes. Confirm that Credentialing's reaction to CredentialApplicationDenied, CredentialApplicationRecommended, ApprovalApplicationRecommended and ApprovalApplicationDenied is documented in the Credentialing sequences (the EPP diagrams no longer show it).
12. The old sections for the three merged search and view flows and the split flows are renamed. Any references (feature maps, graph nodes) to the old titles need updating on re-ingest.
