# Proflearning sequences: change log

Doc edited: design/working-docs/solution-areas/proflearning/proflearning-sequences.md. Diagrams before: 10. After: 10. Splits: 0.

## Applied to every diagram

- Participants declared with aliases, grouped in `box Browser` (UI) and `box MiEdWorkforce (AKS)` (services, Event Bus). Humans only as `actor`.
- Canonical names: UI (replaces ProfLearningUI, ProfLearningAdminUI, AdminUI, CatalogUI), Professional Learning API, IAM API, Documents API, Credentialing API, Event Bus (replaces ServiceBus).
- Every request arrow tagged APP, SVC or EXT. Verbs and paths changed to the spec's (drop query strings and put the filter in the label).
- Authorization checks removed (UI to IAM_API arrows in Program Application Submission and Add Attendees). Permission keys added to each Who line.
- Events drawn as `--)` arrows with the bare event name. CommunicationsWorker and CommunicationsAPI participants removed; the consumer is described in a Note on the Event Bus.
- Plain "Success" responses dropped.
- Conventions text updated (actor is humans only, tag line, standing authorization sentence).

## Sections changed

1. Program Application Submission and Approval, diagram 1: UI no longer calls Documents API; agenda upload now goes UI to Professional Learning API (APP) then Professional Learning API to Documents API (`SVC POST /documents/upload/request`). Who line gained permissions.
2. Same section, diagram 2 (Admin Approves): agenda fetch is now Professional Learning API to Documents API (`SVC GET /documents/{documentId}/download`). Coordinator notification arrow removed in favour of Notes.
3. Add Attendees: IAM calls retargeted to `SVC GET /users/by-unique-id/{uniqueId}` and `SVC GET /users/search`; permission check removed.
4. Adjust SCECH Awards and Certify: paths fixed (`/sessions/{sessionId}/attendees`, `/session-attendees/{attendeeId}/scech-award`); error response now "400" per certifyAttendance spec; CommunicationsWorker removed.
5. Sponsor Requests SCECH Correction: roster path fixed.
6. Admin Reviews Correction: tags and events only.
7. Configure Evaluation Template: tags; UI call to Question Set API kept (see Questions).
8. Manage Program Categories: deactivate is `POST .../deactivate` (was PATCH); error shows "400" per spec.
9. Apply College Course: paths `/my/college-courses/eligible` and `/my/college-courses`; notification via Note.
10. Public Catalog Search: `EXT public` tags for search, program and sponsor detail; bookmark is APP.

## Splits

None. Proflearning has no rows in 3.1, 3.2 or 3.3. Section 3.4 rows 1 and 9 were applied as above.

## Sections left alone and why

No section was left untouched. Structure, prose, Key Decisions, State Changes, Events Published and Error Scenarios text are unchanged. Not split (not flagged, within size limits): Manage Program Categories (create, subcategory and deactivate in one alt) and Adjust SCECH Awards and Certify (nested error alt retained).

## Missing operations (no operationId in any spec)

| Area | Tag | Verb and path | Section |
|---|---|---|---|
| proflearning | APP | GET /educators/search | Add Attendees |
| proflearning | APP | agenda upload request (no path invented) | Program Application Submission |
| proflearning | APP | GET /correction-requests/{correctionRequestId} (spec has list, submit, approve, deny only) | Admin Reviews Correction |
| credentialing | SVC | GET /credentials/{educatorId}/type | Adjust SCECH Awards and Certify |
| credentialing | SVC | GET /credentials/{educatorId}/last-issuance | Apply College Course |
| question sets | APP | GET /question-sets | Configure Evaluation Template |

Closest existing: `getEducatorCredentials` (`GET /educators/{educatorId}/credentials`, internal-user and internal-service) could serve both Credentialing calls.

## API kind observations

- No operation is marked both external and application. `getProgram` and `getSponsor` are `internal-user, external-public`; the catalog diagram uses them from the public page, so tagged `EXT public`. If an authenticated educator uses them the tag would be APP.
- `searchUsers` (IAM) is `internal-user`, but Add Attendees calls it from Professional Learning API; a service-kind operation is needed or the educator search should be reworked.
- `getEducatorCredentials` is dual access (internal-user, internal-service); the sequences would use the service form.
- `searchCatalog` is external-public and drawn from the UI, which is a browser call; the kind is not one of the three named.
- Documents operations are all internal-service, which matches the new SVC wiring.

## Questions

- Agenda upload: no Professional Learning operation exists for it. Confirm the endpoint that requests the SAS token and hands the document reference back to the UI.
- Question Set API has no spec or owner in the architecture; should the UI call it directly or through Professional Learning API?
- Educator search: should it call IAM `searchUsers` as a service call, and is `GET /educators/search` meant to be a new Professional Learning operation?
- The original Admin Approves diagram showed a single combined notification step for all three decisions; it is now three Notes. Confirm Communications consumes ProgramApproved, ProgramRejected and InfoRequested.
- Admin approve action is gated by `program.approve` for reject and request-info as well (per spec); permissions doc lists only approve.
- Certify: the "Evaluation Required" check shows a 400 under a nested alt; confirm whether to flatten to one alt.
- Key Decisions text still says the Admin UI accesses endpoints directly, which is consistent with APP tags.
