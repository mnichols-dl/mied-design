# IAM sequences: change log

File edited: design/working-docs/solution-areas/iam/iam-sequences.md. Diagrams before: 20. After: 23 (3 splits). Nothing else was edited.

## Applied to every diagram
- Actor is humans only; every non-human is a participant. Boxes added: Browser (UI), MiEdWorkforce (AKS), External.
- Canonical participants: UI, IAM API (IamApi), Organizations API (OrgsApi), Staffing API (StaffingApi), Event Bus (EventBus), Mi-Key (MiKey), MiLogin. The old "Notification", "CommAPI", "Identity Resolution", "Public Website" and "Mi-Key (external)" participants are gone.
- Every request arrow between participants is tagged APP, SVC, EXT or OUT with verb and path from the spec. Human to UI arrows and responses are untagged.
- Authorization checks are not drawn. Permission keys added to the Who line of gated sections. Conventions text updated (actor humans only, tag legend, box grouping, standing authorization sentence).
- Direct Notification and CommAPI sends replaced by event arrows plus a Note "Consumed by Communications (...)". Where the old diagram had no matching event, the Note sits on the Event Bus or IAM (see Questions).
- One alt per diagram. Extra failure branches removed (the Error Scenarios text already covered them). Self arrows merged.
- Plain OK responses dropped; responses kept where they carry data or a status code.
- "Check existing authorizations" is drawn as APP GET /authorizations/my-authorizations (getMyAuthorizations). Browser redirects to MiLogin are tagged OUT with the browser (UI) as caller.

## Sections changed (title, what changed)
- Citizen User - Initial Sign-In & Identity Resolution: split (see Splits).
- Business User - Initial Authorization Request: UI added, org search via APP then SVC to Organizations API, submit is APP POST /authorization-requests, Lead Admin lookup is SVC GET /organizations/{organizationCode}, emails become Note on AuthorizationRequested. Permission iam.authorization.request.
- Lead Administrator Bootstrap: depth reduced (outer alt on Business/Worker removed, kept as Note; one alt on Lead Admin yes/no, loop kept). SVC GET /organizations/by-lead-admin. Welcome email is a Note. No permission line (automatic grant, see Questions).
- Scope Approver - Approves Authorization Request: APP GET and POST /authorization-approvals/{token}; opt Modify roles folded into the approve step; three error branches merged into one else. Permission iam.authorization.approve.
- Scope Approver - Rejects Authorization Request: APP GET and POST /authorization-denials/{token}. Permission iam.authorization.deny.
- Business User - Withdraws Pending Authorization Request: APP GET /authorization-requests, APP POST /authorization-requests/{requestId}/withdraw. Permission iam.authorization.withdraw.
- Authorization Request - Link Expiration: Scheduler as participant, SVC POST /admin/jobs/expire-requests. Permission iam.inactivity.run (the spec's key for this job).
- Business User - Request Authorization Update: APP/SVC org search, APP POST /authorization-requests. Permission iam.authorization.request.
- Business User - Self-Remove Authorization: APP DELETE /authorizations/{authorizationId}. Permission iam.authorization.remove-own.
- External Party - Request User Authorization Removal: Public Website replaced by UI (Browser) plus a separate "IAM External API" participant; EXT POST /removal-requests (public). Permission line says none (public).
- System Admin - Reviews External Authorization Removal Request: APP GET /removal-requests, approve and deny calls (missing operations). Permission iam.removal-request.review.
- Automated Inactivity-Based Account Deactivation: SVC POST /admin/jobs/deactivate-inactive-users. Permission iam.inactivity.run.
- System Admin - Manual Authorization Grant: APP GET /users/search, GET /users/{userId}, org search, APP POST /authorizations/manual-grant, SVC GET /organizations/{organizationCode}. Permission iam.authorization.grant.
- Citizen User - Update Account Demographics: depth reduced (employment alt and validation alt removed; employment locking kept in Note text and Key Decisions; validation failure is in Error Scenarios). APP POST /citizen-identity/account-update, OUT to Mi-Key. The Staffing participant is gone; the district notification is a Note on PersonRecordUpdated. Permission iam.citizen-identity.submit-request. PersonRecordUpdated is now drawn once (it was drawn twice).
- Citizen User - Cancel Pending Update Request: APP GET /citizen-identity/requests/my-requests and APP POST .../cancel. Permission iam.citizen-identity.cancel-own-request.
- Business User - Request Link ID: UI added; Staffing call is APP POST /employee-roster/{employeeId}/link-id-requests (missing); StaffingApi to IAM is SVC POST /identity-resolution/requests; unused Identity Resolution and CommAPI participants removed; follow-ups are Notes on the events. Permission taken from the spec's keys on that operation.
- Identity Resolution - Split/Retire ID (Mi-Key-Direct): Mi-Key callback drawn into its own "IAM External API" participant, tagged EXT; IAM External API publishes the events; Staffing reaction is a Note.
- Integration Flows: heading note added saying it is reference material, suggested home solution-integrations.md (MiLogin, Mi-Key) and the Organizations capability doc. Tables and prose untouched. The MiLogin diagram was redrawn (UI as browser, tags, one alt with three branches, nested alts flattened into combined steps).
- Notes (UI Context Management): note added saying it is a standard, suggested home iam-domain.md or patterns-and-principles/authentication.md. Text untouched.

## Splits (old title to new titles)
- Citizen User - Initial Sign-In & Identity Resolution to "Citizen User - Sign-In and Authorization Check" (diagram "IAM - Citizen Sign-In - Authorization Check") and "Citizen User - Account Creation (Mi-Key Match)" (diagram "IAM - Citizen Sign-In - Account Creation (Mi-Key Match)"). Both headings were renamed; the old heading no longer exists.
- Identity Administrator - Resolve Identity Request to the same heading (diagram "IAM - Resolve Identity Request - Match, Create New, Deny") and new section "Identity Administrator - Resolve Link ID Request" (diagram "IAM - Resolve Identity Request - Link ID"). Origin based follow-ups moved to a table in Key Decisions area ("Follow-ups by origin"); the two citizen follow-ups are still drawn as opt blocks so their events appear once.
- Business User - Request New ID (Near Match Escalation) to the same heading (diagram "IAM - Request New ID - Submit and Match") and new section "Business User - Request New ID - Near Match Self-Resolve or Escalate" (diagram "IAM - Request New ID - Near Match Self-Resolve or Escalate"). This split was not in the 3.1 table; I made it because 3.2 flags the diagram for arrow count and there is a natural divergence point.

## Sections left alone and why
None of the diagram sections were left untouched. Text blocks (Key Decisions, State Changes, Events Published, Configuration, Post-Approval) were left as written except where a split required distributing them. Integration Flows prose and tables, and the Identity Resolution Integration subsection, were left as written (only a heading note added).

## Missing operations (no operationId in any spec)
| Area | Tag | Verb and path | Section |
|---|---|---|---|
| iam | APP | GET /authorization-approvals/{token} (view request details from link) | Approves; Rejects |
| iam | APP | GET /removal-requests (pending queue) | System Admin Reviews Removal Request |
| iam | APP | POST /removal-requests/{referenceId}/approve (proposed path) | System Admin Reviews Removal Request |
| iam | APP | POST /removal-requests/{referenceId}/deny (proposed path) | System Admin Reviews Removal Request |
| iam | EXT | POST /identity-resolution/callbacks (proposed path, Mi-Key split and retire callback) | Split/Retire ID |
| staffing | APP | POST /employee-roster/{employeeId}/link-id-requests (proposed path) | Request Link ID |
| iam | OUT | Mi-Key REST calls (submit demographics, resolve near match, confirm match, retire and merge): drawn without a path because solution-integrations.md defers to the Mi-Key Swagger and has CONFIRM markers | Account Creation; Update Demographics; both Resolve sections; New ID sections |
| iam | OUT | OIDC authorize and token (MiLogin) | sign-in sections, MiLogin Integration |

Operations that exist in the spec but have no arrow: listPendingApprovals, getRole, listRoles, reports, getEffectivePermissions, adminUpdatePersonRecord, getUserByUniqueId, getRemovalRequestStatus. Not a diagram problem, noted for the permission traceability check (iam.identity-admin.update-person-record has no sequence).

## API kind observations
- No IAM or Organizations operation is marked both external and application. Staffing add-employee, resolve-near-match and request-new-id are marked internal-user plus external-client; the IAM New ID sequence shows only the internal path.
- selfResolveIdentityRequest and escalateIdentityRequest are internal-user in iam-api.yml and drawn here as UI calls to IAM (APP). staffing-api.yml says Staffing proxies these (resolveNearMatch, requestNewId, with an external-client access). Evidence in the sequence: "Show near-match resolution screen (backed by IAM)" and the district user calling IAM directly. Likely they should be SVC from Staffing API, or the sequence should draw the Staffing proxy.
- submitBusinessUserIdentityRequest is internal-service and drawn SVC from Staffing API. Consistent.
- searchOrganizations and getOrganization appear in iam-api.yml as internal-user (UI calls IAM, IAM proxies) and in organizations-api.yml as internal-user plus internal-service. The sequences draw both hops (APP then SVC).
- getOrganizationsByLeadAdmin is internal-user plus internal-service in the Organizations spec but is only used as SVC by IAM; internal-user looks unnecessary.
- triggerExpireRequests and triggerDeactivateInactiveUsers are internal-service and need a human permission (iam.inactivity.run); the sequence shows the Scheduler as caller.
- submitRemovalRequest is external-public. Tagged EXT (public) per the style guide.

## Questions
1. Lead Administrator Bootstrap and the sign-in flows: no operation matches "validate token and auto-grant on first login". getMyAuthorizations is used as the trigger, and the auto-created authorizations are a side effect. Is there a login or session operation to add to iam-api.yml?
2. Bootstrap has no permission key (automatic grant). Leave the Who line without one?
3. Mi-Key callback (Split/Retire) and the public removal form are drawn as "IAM External API". Is this a separate deployment from IAM API, and what is the callback path? The diagram shows it publishing events directly because the style guide says IAM publishes after the callback.
4. UniqueIdAssigned: the domain doc and the Integration Flows subsection say IAM subscribes to a Mi-Key or Identity Resolution event. Per the style guide IAM now publishes it after the Mi-Key response. The old self arrow "Receive UniqueIdAssigned / IdentityResolutionCompleted" was dropped. Confirm who consumes UniqueIdAssigned.
5. Events with no listed home for notifications: removal request denied (requestor notified), inactivity summary report to System Admins, and "FYI to Lead Admin" cases. Drawn as Notes. Should events be added to the domain doc?
6. Removal request review: the spec has no review operations and no detail view. The list response is assumed to carry requestor info and target authorizations. Confirm paths, and whether approving reuses revokeAuthorization internally.
7. Request Link ID: the permission is the pair of Staffing keys on the IAM submit operation, with no Link ID specific key. Add one? Also confirm the Staffing endpoint for Link ID.
8. Update Account Demographics: no operation returns the citizen's own demographic fields (SSN masked) for the form, and active-employment field locking is decided somewhere not drawn. Removed the alt on active employment; behaviour kept in text. Confirm an operation and who checks employment.
9. Link expiration and inactivity: permission for triggerExpireRequests is iam.inactivity.run in the spec. Correct or should there be an iam.approval-link key?
10. MiLogin Integration diagram overlaps the sign-in sections. Candidate for removal once solution-integrations.md has the MiLogin flow.
11. Mi-Key OUT arrows have no paths; the integrations doc still has CONFIRM markers for the resolve call.
12. Header X-Organization-Context in Notes: confirm destination doc.
