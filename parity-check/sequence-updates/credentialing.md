# Credentialing sequences: change log

Doc edited: `design/working-docs/solution-areas/credentialing/credentialing-sequences.md`. Original copy kept only in the session scratchpad. Style guide: `design/parity-check/sequences-standardization.md` section 2, items from section 3.1 to 3.4.

Counts: 18 diagrams before, 23 after, 5 splits (each into 2 sections). All 23 diagrams were checked for declared participants, closed alt/loop/box blocks, no semicolons in message text, no arrow or em dash characters inside diagrams. Largest diagram is 24 arrows (Configure Endorsement Definition). Every diagram has at most one alt block.

## Changes applied to every section

- Conventions list rewritten: actor is humans only, participant is everything else, arrows carry the API kind tag (APP, SVC, EXT, OUT), box grouping explained, and the standing sentence about cached IAM authorization added.
- Participants renamed to canonical labels and aliases (Credentialing API as CredApi, PPR API, EPP API, Staffing API, Documents API, Professional Learning API, IAM API, Event Bus). "Application Service", "Professional Practices API", "Educator Prep API", "Document Service", "Identity & Access API", "Reference Data API" and all UI variants ("Credentialing UI", "Credentialing Admin UI", "Credential Processor UI") are gone. One participant `UI` in `box Browser`; services and Event Bus in `box MiEdWorkforce (AKS)`.
- All "Verify permission" arrows to IAM, and their "Permission confirmed" answers, removed (9 diagrams). The permission key moved to the Who line as `Permission: <key> (scope note)` in every gated section (scope notes taken from credentialing-permissions.md). System-process sections say "Permission: none".
- Every request arrow between participants now starts with a tag and uses the verb and path of the spec (query strings dropped, `/api` not used). Responses untagged.
- Plain confirmations ("Application deleted", "Credential suspended", "Approval confirmed" and similar) dropped from API responses. UI to human confirmations kept.
- Business Rule Engine participant removed from all diagrams and replaced by self arrows or notes labelled "(Business Rule Engine)" (see Questions).
- Reference Data API participant removed: its calls are Credentialing's own endorsement and assessment definition endpoints (disagreement 13).
- Direct UI to Document Service arrows replaced by UI to Credentialing API (APP) then Credentialing API to Documents API (SVC), because the Documents upload, staging, download and search operations are `internal-service` (disagreement 9).
- Open questions and editorial remarks taken out of diagrams (see Questions): payment status "NOTE not yet resolved" arrow text, "Wait for PaymentCompleted ... not the previously-referenced PaymentReceived" note, "[endpoint unconfirmed ...]" and "(NOT YET DEFINED in credentialing-api.yml ...)" text in arrows. The "wait for PaymentCompleted" statement was moved into the new decision table. The other remarks are still present in the existing Key Decisions text.

## Sections changed (not split)

- Add Endorsement to Certificate: tags, canonical names, Reference Data calls folded into Credentialing API notes, self arrows folded into 4 notes to meet the self arrow limit (was over the arrow limit, now 17). Permission added: `credentialing.endorsement.add`.
- Application Manual Review and Approval: added actor "Credential Processor" and the `UI` participant (the old diagram used the UI as the only caller). Document list now fetched by Credentialing API from Documents API (SVC `GET /documents/search`), not by the UI. Plain confirmation responses dropped. Permission line added.
- Suspend or Revoke Credential: IAM participant and arrows removed, search and detail mapped to `GET /credentials` and `GET /credentials/{credentialId}`. Consumers note kept.
- Nullify Endorsement: IAM removed, search mapped to `GET /credentials`.
- Bulk Renewal Initiation: `GET /credentials/expiring` mapped to `GET /credentials` (searchCredentials has `credentialType` and `expiringBefore` filters).
- Print Certificate: `GET /credentials?educatorId&status=approved` mapped to `GET /credentials`. Document Generation Service kept as a participant (aliased PdfGen) with an SVC arrow that has no operation.
- Delete Pending Application: list mapped to `GET /applications`, status check mapped to `GET /applications/{applicationId}`.
- Configure Credential Definition, Configure Endorsement Definition, Configure Assessment Definition: IAM and Reference Data removed, nested effective-date alt in credential definition replaced by a note (one alt only). Definition list calls match the spec. Arrow counts 22, 24, 20 (were over the arrow limit for the first two).
- Import Assessment Results: IAM removed, Document Service replaced by Documents API behind Credentialing API (staging upload request, download), validation self arrows collapsed to one.
- Validate Assessment Results: depth and notes reduced. The nested loop with per-assessment alts replaced by one self arrow plus a Decision Table (per required assessment) in the text. "Assessment Service" participant replaced by self arrows (see Questions). One alt kept (all met or not).
- View Definition History: IAM removed. The `GET /{definition-type}` placeholder paths replaced by the credential-definition paths, with a note that the same calls apply to the endorsement and assessment definition paths. Who line gets the permission, including `credentialing.definition.compare-versions` which the spec requires for the history operations.

## Splits (old title to new titles)

1. Submit New Certificate Application to "Submit New Certificate Application - Check Eligibility" and "Submit New Certificate Application - Submit Application". Hold path (application created in Professional Practice Hold) stays in Check Eligibility because that is where the original drew it; Blocked is also there. "Not eligible" branch moved from the diagram to the existing Error Scenarios text. Submit Application holds documents and submission.
2. Submit Permit Application to "... - Check Eligibility" (educator search, PPR stop check, permit types and questions) and "... - Submit Application" (answers, Hold alt, validation, employment check, creation). The hard stop alt moved to the existing Error Scenarios text. Submit Application cannot re-draw the PPR call without inventing a second call, so it carries a note that the PPR status comes from Check Eligibility (see Questions).
3. Issue Temporary Permit - Exceptional Cases (Admin) to "Issue Temporary Permit (Exceptional) - Check Eligibility" and "Issue Temporary Permit (Exceptional) - Submit Application". Titles use "(Exceptional)" so the title keeps the three part shape. In the original, the PPR check happens after the POST, so Check Eligibility is only the lookup phase (educator search, permit types) and is short (7 arrows). The unconfirmed capability status note is kept on the first section only. The nested requirements-not-met alt moved to existing Error Scenarios.
4. Renew Credential to "Renew Credential - Check Eligibility" (renewable list, requirements, SCECH hours) and "Renew Credential - Submit Application" (PPR check, validation, creation). The validation-fails alt moved to Error Scenarios.
5. Application Auto-Approval to "Auto-Approve Application" (happy path, with the Decision Table) and "Auto-Approval Held for Manual Review" (PPR Hold or Blocked, ConditionalClearance, rule failure). Pending Payment and Pending Documents exist only in the decision table, not drawn. The title "Application Auto-Approval" no longer exists; other docs that link to it need updating.

Heading renames affect cross references in other docs (for example "Submit Permit Application", "Application Auto-Approval", "Renew Credential", "Issue Temporary Permit — Exceptional Cases (Admin)"). Prose inside this doc still uses the flow-level names and was not rewritten.

## Sections left alone and why

None left fully alone: every diagram needed at least tags, canonical names and the Permission line. Text blocks (What, When, Key Decisions, State Changes, Events, Error Scenarios) are unchanged except where a diagram moved between sibling sections, where the new "See also" lines and Decision Tables were added, and the Permission additions to Who lines.

## Missing operations

Arrows with no operationId in any spec. Tag, verb and path as drawn, section.

Credentialing:
1. APP GET /certificate-types, Submit New Certificate Application - Check Eligibility
2. APP GET /eligibility-questions, same section
3. APP POST /validate-eligibility, same section
4. APP POST /upload, Submit New Certificate Application - Submit Application and Import Assessment Results (the original drew UI to Documents API `POST /upload`; no Credentialing operation fronts the Documents upload request)
5. APP GET /educators/{educatorId}/search, Submit Permit Application - Check Eligibility
6. APP GET /permit-types, Submit Permit Application - Check Eligibility and Issue Temporary Permit (Exceptional) - Check Eligibility
7. APP GET /permit-questions, Submit Permit Application - Check Eligibility
8. SVC "Verify employment assignment (if required)" to Staffing API, no verb or path, Submit Permit Application - Submit Application (closest operation is validatePlacement, `POST /assignments/validate-placement`, internal-service, but it validates credential to position alignment, not employment, so it was not substituted)
9. APP POST /applications/permits/admin-issue/exceptional, Issue Temporary Permit (Exceptional) - Submit Application (the original already flagged it as not defined)
10. APP GET /endorsements/eligible, Add Endorsement to Certificate
11. APP GET /assessment-requirements, Add Endorsement to Certificate
12. APP GET /credentials/renewable, Renew Credential - Check Eligibility
13. APP GET /renewal-requirements, Renew Credential - Check Eligibility
14. APP POST /applications/{applicationId}/on-hold, Application Manual Review and Approval (permission `credentialing.application.place-hold` exists, no endpoint)
15. APP GET /endorsements/{endorsementId}/details, Nullify Endorsement
16. SVC Generate certificate PDF to Document Generation Service, no verb or path, Print Certificate
17. APP GET /credential-categories, Configure Credential Definition
18. APP GET /credential-definitions/{definitionId}, Configure Credential Definition (spec has only PUT and `/history` for this path)
19. APP GET /endorsement-definitions/{definitionId}, Configure Endorsement Definition
20. APP GET /assessment-definitions/{definitionId}, Configure Assessment Definition
21. APP GET /credential-definitions/{definitionId}/versions/{versionId} (and the endorsement and assessment equivalents), View Definition History

Professional Learning:
22. APP GET /scech-hours, UI to Professional Learning API, Renew Credential - Check Eligibility (already noted as an open item in the Key Decisions text; `proflearning-api.yml` has no match)

Total: 22 (counting the three definition `GET {definitionId}` calls and the `versions` call separately; 21 distinct arrows if the three `versions` variants are counted as one).

Operations that did match and were used: submitApplication, getApplication, deleteApplication, listApplications, approveApplication, denyApplication, submitRenewal, initiateBulkRenewals, submitEndorsementAddition, adminIssuePermit, searchCredentials, getCredential, suspendCredential, revokeCredential, printCredential, nullifyEndorsement, list/create/update credential, endorsement and assessment definitions, getCredentialDefinitionHistory, importAssessmentResults, plus other areas: getPprClearanceAssessment, getCandidateEnrollmentStatus, requestDocumentUpload, uploadToStaging, downloadDocument, searchDocuments, searchUsers.

## API kind observations

- No operation in credentialing-api.yml is marked both external and application. Only two kinds of mixed marking exist: `getEducatorCredentials` is `internal-user, internal-service` (not drawn in any credentialing sequence, so no evidence either way here) and the two `/public/educator-credentials` operations are `external-public` (not drawn; "Public Portal" appears only in a Note).
- Documents operations used by Credentialing (requestDocumentUpload, uploadToStaging, downloadDocument, searchDocuments) are `internal-service`. The original diagrams had the UI calling them. They are now SVC calls from Credentialing API. This implies Credentialing needs its own `APP` endpoints to front upload, staging upload and the document list; none exist in the spec (see missing operations 4 and the Questions on getApplication).
- `getPprClearanceAssessment` and `getCandidateEnrollmentStatus` are `internal-service`, consistent with the SVC arrows from Credentialing API.
- `searchUsers` (IAM) is `internal-user`, so the UI calling it directly (Exceptional Check Eligibility) is a legitimate APP call, unlike the removed permission checks.
- `GET /scech-hours` has no operation. It is drawn from the UI as APP (as in the original). Given the domain doc expects Credentialing to track SCECH by events, this call may need to disappear or become an SVC call from Credentialing API.
- `getEndorsement`-style reads (`GET /endorsements/{endorsementId}/details`) and definition reads by id are not in the spec but are plainly APP (UI driven).
- `approveApplication` carries two permissions (`approve` and `bypass-validation`) in the spec; the Who line lists only the permissions the sequence needs. See Questions.

## Questions

1. Business Rule Engine: is it a library inside Credentialing API or a separate service? It is not in solution-architecture.md. It was drawn as a participant 10 times; this pass turned it into "(Business Rule Engine)" self arrows or notes on Credentialing API. Revert if it is a service.
2. Assessment Service in Validate Assessment Results: no such service in the architecture. AssessmentResult is a Credentialing aggregate, so the `GET /assessment-results?educatorId={id}` call became a self arrow. There is also no `GET /assessment-results` operation in the spec (only import). Confirm.
3. Document Generation Service in Print Certificate: technical design says a PDF generation service consumes `CredentialIssued` and stores the PDF in Documents, while the sequence has Credentialing call it synchronously on print. Which is right, and what is the call?
4. Documents access from the UI: should Credentialing expose its own upload endpoint and a document list (or include documents in `getApplication`)? The Manual Review diagram assumes `GET /applications/{applicationId}` makes Credentialing call `GET /documents/search`; the spec does not say that.
5. Submit Permit Application - Submit Application: the PPR clearance call is drawn only in Check Eligibility, and the Hold branch in Submit uses a note saying the status comes from there. Does the submit endpoint re-evaluate PPR clearance server side? If yes, add the SVC arrow.
6. Exceptional permit split: Check Eligibility is only a lookup phase because the original did the PPR check after the POST. Keep two sections or merge back to one?
7. Submit New Certificate Application: Hold path creates an application during eligibility validation (before documents and payment), as in the original. Confirm this is the intended point at which the application aggregate is created.
8. Open question removed from the Auto-Approval diagram: which endpoint or event backs the "locally cached PaymentStatus" check (technical design Open Technical Question #7). Still stated in the Key Decisions text of Auto-Approve Application.
9. Open question removed from the Submit Permit diagram: employment assignment verification endpoint (Open Technical Question #8). Still stated in the Key Decisions text only as "Not yet modeled"; the endpoint text lived in the arrow and is now only in this log. Consider adding it to the tracking file.
10. Manual review permissions: spec lists `approve` plus `bypass-validation` for approveApplication, but the Who line follows the catalog (approve, deny, place-hold). Is `bypass-validation` required to approve or only to override rules?
11. Submit New Certificate Application permission: the spec says submitApplication requires `credentialing.application.submit` only, while the catalog says admins use `manual-submit`. The Who line mentions both; confirm.
12. `GET /credentials` used for the Bulk Renewal expiring list and the Print list. Confirm searchCredentials (with `expiringBefore`, `status` filters) is the intended operation.
13. IAM Who-line wording for View Definition History: `credentialing.definition.compare-versions` is required by the history operations in the spec but not listed in the original Who line. Added; confirm.
