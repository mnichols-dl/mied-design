# Staffing sequences: change log

File edited: design/working-docs/solution-areas/staffing/staffing-sequences.md

Diagrams before: 9. After: 14. Splits: 4 (nine sections became 14). Missing operations: 5.

Tagging rule used. The tag reflects the caller drawn in the diagram: UI to Staffing API is APP, API to API is SVC. Where the spec x-access differs, the difference is listed under API kind observations and the tag was not changed. No EXT or OUT arrows are drawn: no Staffing sequence has an external-facing endpoint or an outbound call to an external system (Staffing never calls Mi-Key directly).

All sections: Conventions rewritten (Actor is humans only, participant is everything else, API-kind tags, box grouping, the standing authorization sentence). Every participant is declared with `participant Alias as Label` and grouped in `box Browser` and `box MiEdWorkforce (AKS)`. Canonical names: UI, Staffing API, IAM API, PPR API, Credentialing API, Organizations API, Event Bus. Removed: CommAPI, email arrows to roles (AuditorRole, ISDAuditor, SOMAuditor, DistrictUser as email targets), the IAM permission check arrow, plain OK responses, body fields in arrow text. Paths aligned to operationIds. Who lines gained `Permission:` in every section, using x-permissions-required from staffing-api.yml and scope notes from staffing-permissions.md. Diagram checks done by script: all participants declared, blocks balanced, no semicolons, arrows or em dashes in text I added. Measures: largest diagram is 20 arrows, 5 participants, nesting depth 1, one alt at most.

## Sections changed

| Section | What changed |
|---|---|
| Add New Employee | Removed the `POST /permissions/check` arrow (permission moved to Who). Validation alt dropped (already in Error Scenarios), nested Match alt collapsed into an opt. Near Match 202 handling became a Note pointing at the IAM sequence. One alt kept (PPR restriction). Event goes to Event Bus with a Consumed-by Note (Communications, PPR API). PPR path is `/educators/{educatorId}/roster-eligibility`. |
| Update Employee Demographics | Reduced to the update and Match path. Validation alt removed (already in Error Scenarios). Cross-notification alt became a Consumed-by Note. Near Match branch ends with a Note pointing to the new sibling. |
| Request Collection Exception | Added a UI participant for the admin steps (the admin previously called Staffing API directly). `POST /collection-exceptions/request` became `POST /collection-exceptions`. Validation alt removed (already in Error Scenarios), approve versus deny is the one alt. Emails became events with Consumed-by Notes. Admin permission moved to Who. |
| Open New Collection | Scheduler declared in the AKS box. Self arrows merged from 7 to 4. CommAPI replaced by Event Bus with Note "Consumed by Communications and Reporting" (from the domain events table). `GET /collections?schoolYear=...` became `GET /collections`. |
| Create New Position | `POST /positions/create` became `POST /positions`. EEMAPI replaced by Organizations API (review 3.4 item 12). Validation alt removed (Error Scenarios), building invalid is the one alt. |
| Update Position Status | Not split, per review row 8. Five nested status alts replaced by a "Status Transition Rules" table and one diagram with a single rejection path. `PATCH /positions/{positionId}/status` became `POST /positions/{positionId}/status`. The table sits in this section because the domain doc was not in scope to edit. |

Sections that were split are described below.

## Splits

| Old title | New titles |
|---|---|
| Update Employee Demographics | Update Employee Demographics; Update Employee Demographics - Resolve Near Match |
| Assign Employee to Position | Assign Employee to Position - Validate Placement; Assign Employee to Position - Create Assignment; Assign Employee to Position - Create Assignment with Credential Error Justification |
| Certify Collection | Certify Collection - Run Quality Review; Certify Collection - Certify |
| ISD Auditor Review District Submission | ISD Auditor Review District Submission - Request Documentation and Record Findings; ISD Auditor Review District Submission - Finalize ISD Audit |

Notes on the splits:
- Diagram titles are "Staffing - " plus the section title. The review proposed the Near Match variant as "Resolve Near Match on Demographic Update"; I used the `<Area> - <Flow> - <Variant>` form from the style guide instead.
- Each new section has What, When, Who, See also, diagram, Key Decisions, State Changes, Events Published and Error Scenarios. The first section of each split keeps the original What and When; the others have short new What and When lines. Original bullets were divided between siblings, not rewritten.
- Assign Employee: the credential and endorsement nested alts became a "Validation Rules" table in the Validate Placement section. "Apply for Temporary Credential" is a Note in Create Assignment. `GET /positions/{positionId}/details` became `GET /positions/{positionId}`.
- ISD Auditor: `GET /audit/{auditItemId}/district-report` became `GET /audit/{auditItemId}`. The undeclared DistrictUser and SOMAuditor are gone (Notes name the Communications consumer).
- Update Demographics resolve and cancel now go UI to Staffing API (APP) then Staffing API to IAM API (SVC), per the staffing-api.yml descriptions. The original had the UI calling IAM directly (see Questions 2).

## Sections left alone and why

- Request Collection Exception: no split row in the review, 19 arrows, one alt. It has two actor stages (request, admin review), so "Request Exception" and "Review Exception" is a candidate split if the owner wants it.
- Open New Collection, Create New Position: no split rows, small diagrams. Changed only for conventions.

## Missing operations (no operationId in any spec)

| Area | Tag | Verb and path | Section |
|---|---|---|---|
| Staffing | APP | GET /employee-roster/search-active | Assign Employee to Position - Validate Placement. listEmployees (`GET /employee-roster`) has a status filter and may be the intended operation. |
| Staffing | APP | GET /audit/{auditItemId}/certification-status | ISD Auditor Review District Submission - Finalize ISD Audit. Staffing spec has getCertificationStatus for collections only. |
| IAM | SVC | POST /identity-resolution/requests/{requestId}/cancel | Update Employee Demographics - Resolve Near Match. iam-api.yml only has `/citizen-identity/requests/{requestId}/cancel`. |
| Organizations | SVC | GET /organizations/{entityCode}/buildings | Create New Position. Organizations API has search, get, hierarchy, by-lead-admin and type operations, no buildings. |
| Staffing | SVC | (no verb or path in the original) Trigger new school-year collection initialization | Open New Collection |

Verified present in a spec: searchEmployees, addEmployee, getEmployeeDemographics, updateEmployeeDemographics, resolveDemographicsNearMatch, cancelDemographicsNearMatch, getPosition, createPosition, updatePositionStatus, validatePlacement, createAssignment, createAssignmentWithJustification, getCertificationStatus, runQualityReview, submitJustifications, certifyCollection, checkExceptionEligibility, requestCollectionException, approveCollectionException, denyCollectionException, listCollections, getAuditItems, getDistrictAuditReport, requestAuditDocumentation, createAuditFinding, finalizeAuditReport, submitBusinessUserIdentityRequest (IAM), getRosterEligibilityAssessment (PPR), getEducatorCredentials (Credentialing).

## API kind observations

The 13 operations marked both internal-user and external-client. In every case the Staffing sequences show only a UI user (district user, or citizen for demographics). No sequence draws an external HR/SIS caller, so the sequences give no evidence of external use. The only evidence for external use is the spec text.

| Operation | What the sequences show | Evidence and reading |
|---|---|---|
| addEmployee | UI user only. Starts with an interactive search (searchEmployees is internal-user only), then add, then possibly a Near Match screen. | Add New Employee. An external system could post, but could not finish a Near Match without the interactive operations below. |
| resolveNearMatch | Not drawn in Staffing. Add New Employee only points to the IAM Near Match sequence. IAM selfResolveIdentityRequest is internal-user only. | Interactive review of potential matches by a person. Looks application only. |
| requestNewId | Not drawn in Staffing. IAM escalateIdentityRequest is internal-user only. | A district user rejects all matches and asks for a new ID. Looks application only. |
| updateEmployeeDemographics | UI user (district user or citizen). A citizen is never an external HR system. | Update Employee Demographics. Plausibly both (HR/SIS pushing updates), but its Near Match response cannot be handled without the UI-only operations. |
| resolveDemographicsNearMatch | UI user on a comparison screen with an attestation. | Resolve Near Match. Looks application only. |
| cancelDemographicsNearMatch | UI user selects Cancel. The spec also says IAM invokes it automatically on timeout, but the sequence shows IAM publishing IdentityRequestCancelled, not calling Staffing. | Resolve Near Match and its Error Scenarios. Application only, or the automatic invocation implies a service-kind operation (a third option). |
| updateEmploymentStatus | Not used in any Staffing sequence. | No evidence either way. |
| createPosition | UI user only. | Create New Position. |
| updatePosition | Not used in any Staffing sequence. | No evidence either way. |
| updatePositionStatus | UI user only. | Update Position Status. The rules apply equally to a system caller. |
| createAssignment | UI user, always after a UI-driven validatePlacement. A credential failure is handled by choosing justification or temporary credential in the UI. | Assign Employee to Position. A system caller has no justification route because createAssignmentWithJustification is internal-user only, so an external client would hit a dead end on credential errors. |
| updateAssignment | Not used in any Staffing sequence. | No evidence either way. |
| endAssignment | Not drawn as a call. The effect (end the assignment at the effective date) happens inside updatePositionStatus. | Update Position Status. |

Other observations:
- validatePlacement is internal-service in the spec, but the sequence has the UI calling it (tagged APP). Either the spec should say internal-user, or the UI should not call it directly.
- getEducatorCredentials (Credentialing) allows internal-user and internal-service. The Staffing call is made inside a user flow, which the Credentialing spec says uses the user token pattern. Tagged SVC here because it is an API to API arrow. Owner to confirm.
- selfResolveIdentityRequest (IAM) is internal-user, but the drawn call from Staffing API is API to API (SVC). Same for the proxy of `/resolve`.
- submitBusinessUserIdentityRequest (internal-service) and getRosterEligibilityAssessment (internal-service) match the SVC arrows drawn.
- External HR/SIS callers (review 3.4 item 7) are not drawn anywhere, because no sequence behaviour exists for them.

## Questions

1. Should each of the 13 dual-kind operations be application only, external only, or two operations? Suggested reading from the sequences: application only for resolveNearMatch, requestNewId, resolveDemographicsNearMatch and cancelDemographicsNearMatch (interactive), and a decision on external HR/SIS scope for addEmployee, updateEmployeeDemographics, updateEmploymentStatus, createPosition, updatePosition, updatePositionStatus, createAssignment, updateAssignment and endAssignment.
2. Near Match resolve and cancel: does the UI call Staffing API, which proxies to IAM (per staffing-api.yml, now drawn), or the UI call IAM directly (original diagram)?
3. Which IAM operation does Staffing call to cancel a business-user demographic Near Match? None exists in iam-api.yml.
4. Is the buildings lookup an Organizations API operation, a synced local replica (staffing-domain.md Dependencies says "Synced local replica"), or EEM? No operation exists today.
5. Is the Open New Collection rollover an in-process scheduled job of Staffing API or a call from a separate Scheduler? No operation exists.
6. The Who line of Update Employee Demographics says Citizen User (self-only), but staffing.employee-roster.update-demographics has Entity scope only and the operation lists no citizen permission or self scope.
7. Events drawn but absent from the Domain Events table in staffing-domain.md: DemographicDataUpdated (only in the entity events list), PositionCreated, PositionStatusChanged, DocumentRequestCreated, CollectionExceptionRequested, CollectionExceptionApproved, CollectionExceptionDenied. The consumed event IdentityRequestCancelled is also not in the Dependencies row for IAM.
8. In the original Add New Employee, it was unclear whether the PPR check still runs after IAM returns a Near Match. The Near Match path is now a Note and behaviour is unchanged.
9. Should the Status Transition Rules and Validation Rules tables move to staffing-domain.md? They were kept in the sequences doc only because that was the permitted file.
10. Open Question #11 of iam-domain.md is still referenced in Key Decisions of the Resolve Near Match section (prose left untouched). No open-question remarks were found inside the original diagrams.
