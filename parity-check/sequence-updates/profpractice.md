# Profpractice sequences: change log

Doc edited: design/working-docs/solution-areas/profpractice/profpractice-sequences.md

Totals: 15 diagrams before, 15 after. 0 splits. 7 missing operations (plus outbound external-system calls noted below). Style guide applied to every diagram: boxes (Browser, MiEdWorkforce (AKS), External), canonical participant names (UI, PPR API, Documents API, Credentialing API, Staffing API, Event Bus, MSP CHRISS Rap Back, Mi-Key, NASDTEC Clearinghouse), actor for humans only, APP / SVC / EXT / OUT tags, no UI-to-IAM arrows, spec paths and verbs.

## Document-wide changes
- Conventions: actor is humans only, participant is every non-human, added tag definitions, added the standing sentence on cached IAM authorization.
- Who line: added "Permission: <key> (scope note)" to all 15 sections.
- Removed the "Verify permission" UI-to-IAM arrows in Manage Account Markers, Configure PPR Worklist and Generate PPR Compliance Report (the 3 profpractice diagrams in 3.4 item 1).
- UI calls to Document Service replaced by UI to PPR API (APP) then PPR API to Documents API (SVC), per 3.4 item 9.
- Wording of Key Decisions, State Changes, Events Published and Error Scenarios was not changed.

## Sections changed
1. Submit Self-Disclosure: boxes, names, tags. Upload now goes UI to PPR API then PPR API to Documents API (`SVC POST /documents/upload/request`, requestDocumentUpload). Permission profpractice.disclosure.submit.
2. Submit Professional Practice Review Response: same as 1. Notes cut from 8 to 5 (dropped the "State: Pending Review" and "ResponseDate" notes, which duplicate State Changes; merged the two "No" branch notes; the LastPPRResponseDate self arrow became a note so self arrows are 4). Permission profpractice.response.submit.
3. Review Disclosure and Update Status: paths aligned to spec (`POST /disclosures/{disclosureId}/add-remark` and `POST /disclosures/{disclosureId}/update-status` replace `POST .../remarks` and `PUT .../status`). Document read moved behind PPR API (`SVC GET /documents/search`, searchDocuments). Arrows cut from 33 to about 22 (removed confirmation responses and the "display available transitions" arrow). The single alt is now the invalid status transition (400 per spec); the edit step became opt. Permission profpractice.status.update plus view, edit and remarks keys.
4. Manage Account Markers: four marker branches replaced by one representative path (set Mandatory Hold) with a note pointing to a new "Marker Actions" table in the section text (all six spec operations, with effects and events). Re-Review reroute kept as an opt with its event. Permission profpractice.markers.manage.
5. Receive Rap Back Notification: MSP CHRISS Rap Back is now a participant in box External (was actor). Kept the separate PPR Webhook Endpoint participant. `EXT POST /webhooks/rapback` (receiveRapBackNotification). Webhook Endpoint to PPR API is `SVC` (see missing operations). Mi-Key is `OUT POST /match-person`. The nested no-match alt became a note (covered by Error Scenarios) so one alt remains. Plain 200 OK and Acknowledgment responses omitted. Existing Communications note kept. Permission: none (external-client).
6. Retrieve Full Rap Sheet: CHRISS SOAP API relabeled MSP CHRISS Rap Back, `OUT SOAP GET_RAPBACK`. `GET /external-checks/{checkId}` and `POST /rapback/retrieve-rapsheet` match spec. Permission profpractice.rapsheet.retrieve.
7. Process NASDTEC Nightly Batch: depth reduced from 3 to 2 (the already-processed check moved into the loop label), self arrows 6 to 4, notes 5 to 4 (merged). Scheduler call is `SVC POST /admin/jobs/process-nasdtec-batch` (processNasdtecBatch). NASDTEC is `OUT GET /v1/clearinghouse/people` (query string moved into the note). Participants relabeled NASDTEC Clearinghouse and Mi-Key.
8. Evaluate PPR Clearance for Credential Application: priority alt chain (8 self arrows, 7 notes) replaced by a two-arrow sequence (`SVC GET /educators/{educatorId}/ppr-clearance`, getPprClearanceAssessment). The rules moved into a "Decision Table" block in the same section, because the domain doc is out of scope for this task (3.3 suggests the domain doc as home).
9. Evaluate Roster Eligibility for Employment: same treatment (`SVC GET /educators/{educatorId}/roster-eligibility`, getRosterEligibilityAssessment).
10. Log Non-System Action: fixed the undeclared UI participant; path now `POST /disclosures/{disclosureId}/log-action` (logNonSystemAction). Permission profpractice.actions.log.
11. Configure PPR Worklist: fixed the broken line `actor Adminparticipant UI as PPR Admin UI` (actor and participant were on one line, invalid Mermaid). Removed IAM arrows. `GET /worklists`, `POST /worklists`, `PUT /worklists/{worklistId}` match spec.
12. View Disclosure History: `GET /educators/{id}/disclosures` relabeled `APP GET /disclosures` (listDisclosures, educatorId is an optional admin filter). Documents now fetched by PPR API (`SVC GET /documents/search`) inside the disclosure detail call.
13. Export Disclosure Data: same relabel for the list call. Export Service kept, now inside the MiEdWorkforce box (see missing operations and questions).
14. Review NASDTEC Disciplinary Record: `POST /external-checks/{checkId}/add-remark` (spec) replaces `.../remarks`. Notes cut from 6 to 5 by merging the two reviewer notes. Query string dropped from the worklist call.
15. Generate PPR Compliance Report: removed IAM and the Reporting Engine participant. Power BI render and export are now one note because a browser-to-Power-BI call fits none of the four tags. `APP GET /reports/data` (getPprReportData). Export alt became opt. Notes 5.

## Splits
None. The profpractice rows in 3.1 (row 22) and 3.3 ask for consolidation and relocation, not splits. No other profpractice diagram was flagged for splitting. Row 22 was done as one generic marker diagram plus a table.

## Sections left alone and why
No section was left completely unchanged; all got the Who permission line and the style changes. Not restructured: Submit Self-Disclosure and Submit PPR Response (only notes trimmed; the second still has two business alts, kept because they are happy-path branches, not error branches), Configure PPR Worklist (create and edit alt kept; it was not on the split list and is a business branch).
Left in place: the stray line "You're right on both counts! Let me add those two missing sequences:" before Review NASDTEC Disciplinary Record. It is a leftover conversational line outside any diagram. I did not rewrite prose, so please decide whether to delete it.

## Missing operations (no operationId in any spec)
| Area | Tag | Verb and path | Section |
|---|---|---|---|
| profpractice | APP | POST /upload (document upload request through PPR API; Documents has requestDocumentUpload but PPR has no front door for it) | Submit Self-Disclosure; Submit PPR Response |
| profpractice | APP | GET /educators/search | Manage Account Markers |
| profpractice | APP | GET /worklists/{worklistId}/external-checks (was with ?source=NASDTEC) | Review NASDTEC Disciplinary Record |
| profpractice | APP | PUT /external-checks/{checkId}/status | Review NASDTEC Disciplinary Record |
| profpractice | APP | GET /worklists/{worklistId} (spec has only PUT on this path) | Configure PPR Worklist |
| profpractice | SVC | Process Rap Back notification (Webhook Endpoint to PPR API, no path in spec) | Receive Rap Back Notification |
| export | APP | POST /export/disclosures (Export Service, in no spec) | Export Disclosure Data |

Outbound calls to external systems are not in any internal spec by nature: OUT POST /match-person (Mi-Key, 2 sections), OUT SOAP GET_RAPBACK (CHRISS), OUT GET /v1/clearinghouse/people (NASDTEC). The Mi-Key match path is taken from the old diagram; the IAM spec has no matching operation.

## API kind observations
- No operation is marked both external and application. receiveRapBackNotification is external-client only and matches the EXT use here.
- All Documents operations used (requestDocumentUpload, searchDocuments) are internal-service, which agrees with calling them from PPR API (SVC). The old diagrams had the browser call Documents directly. requestDocumentUpload describes a SAS token for browser upload to blob storage, so the browser-to-blob step is not an API call and is not drawn.
- getPprClearanceAssessment and getRosterEligibilityAssessment are internal-service but list x-permissions-required (clearance.view, eligibility.view) with scopes (system-wide, entity, district, ISD). A service call carries no user token, so those scopes cannot be evaluated; the sequences treat them as caller permission notes only.
- processNasdtecBatch is internal-service and lists profpractice.nasdtec.view although the caller is a scheduler. Same point.
- getDisclosure returns documentIds only. The sequences show PPR API composing document metadata from Documents API (`SVC GET /documents/search`), which is an assumption (see Questions).

## Questions
1. Permission keys in the old Who lines and diagrams do not exist in the permissions catalog: profpractice.disclosure.update-status (catalog: status.update), profpractice.rapback.retrieve-rapsheet (rapsheet.retrieve), profpractice.disclosure.log-non-system-action (actions.log), profpractice.reports.disclosure-metrics.view and profpractice.reports.compliance.view (report.view), and account-marker.set-mandatory-hold and similar (markers.manage). The old Who text was kept and the catalog key added after it. Should the old text be replaced? Review NASDTEC Disciplinary Record used profpractice.disclosure.view, while the spec asks nasdtec.view.
2. Permission Segregation (Key Decisions of Manage Account Markers) says set and clear use different permissions, but the catalog and spec have only profpractice.markers.manage for all six actions.
3. Clear Enhanced Monitoring and Clear Re-Review exist in the spec but were never in the diagram; their events are unspecified.
4. Evaluate PPR Clearance listed "Other reviewed disclosures" as priority 5 but had no branch or outcome for it. Kept as a row marked unspecified.
5. Where do the Evaluate decision tables live? They are in this doc for now; 3.3 suggests the domain doc.
6. Who owns the PPR upload entry point (POST /upload) and the document list for a disclosure? Neither exists in profpractice-api.yml.
7. Export Service is not in the architecture or any spec. Is it part of PPR API, Reporting API or a separate service? What permission does export need?
8. Compliance report: the old diagram had the browser call a "Reporting Engine (Power BI)" directly; Reporting API has generateEmbedToken (reporting.embed-token.generate). Should this flow use the Reporting API embed token instead? Power BI render is currently a note only.
9. Rap Back: how does the Webhook Endpoint hand off to PPR API (SVC route, event, queue)? No operation exists. The Rap Back acknowledgment (200 OK) is no longer drawn but remains in Error Scenarios.
10. Mi-Key match-person: which Mi-Key operation and path is correct (solution-integrations refers to the Mi-Key Swagger)?
11. The Mermaid renderer must support `box` (Mermaid 10 or later). No renderer was available here, so the diagrams were checked by hand only.
