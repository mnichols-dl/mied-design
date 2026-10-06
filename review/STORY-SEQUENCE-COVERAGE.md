# Story → Sequence → API → Permission coverage review

> **Update (deeper pass + fixes applied to the ttl):** see [Follow-up: external-system
> resolution fixes and corrections to this review](#follow-up-external-system-resolution-fixes-and-corrections-to-this-review)
> at the end of this file. Several findings below (§3's permission gaps, §2/§4's near-miss
> hypotheses, §5a's "confirmed" gaps) turned out, on deeper investigation, to already be
> deliberate and correctly modeled — that section explains what actually changed and retracts
> what didn't hold up.

Front-to-back traceability check for the whole solution graph: for every `sd:UserStory`,
does a user-initiated `sd:Sequence` carry it out, does that sequence show a wireframe and
the API operation(s) it calls, and does each operation have a `sd:PermissionRequirement`?
This is the first pass at this specific chain (prior review passes, tracked in `GAPS.md` and
as `sd:Annotation` instances throughout the domain files, worked FDD-by-FDD from the source
documents rather than by walking this graph shape end to end).

## Method

`solutions/MiEdWorkforce/solution/MiEdWorkforce.ttl` is 4MB / ~26k lines — too large to eyeball
directly. Extraction used `oxigraph` (already a dependency of `editor/`) from a throwaway Node
script: load the full graph into an in-memory store, run ~12 SPARQL `SELECT`s (one per node/edge
type — user stories, `sd:satisfiedBy` edges, sequences, `sd:SequenceReference`s, API operations,
`sd:PermissionRequirement`s, permissions, wireframes, existing `sd:Annotation`s, etc.), dump each
to JSON. A second plain-Node script joined those JSON files in memory and classified every user
story against the chain, using the same rules the existing `validations/*.sparql` queries use
(e.g. only `sd:initiatedBy "user"` sequences count toward "carries out this story"). Both scripts
and the intermediate JSON are throwaway (scratchpad), not committed — this file is the durable
output.

Numbers below are **before** subtracting anything already on record. 510 `sd:UserStory` instances
exist; every one of them nominally "has a gap" under a naive check, because ~121 have no
`sd:Wireframe` reference — expected and not meaningful right now, since only 5 `sd:Wireframe`
instances exist anywhere in the solution (wireframe capture is early/ongoing, per the ontology's
own `sd:Wireframe` comment). Wireframe coverage is **excluded** from the findings below for that
reason; re-run this check once wireframe capture is further along.

## Summary

| Check | Count | Notes |
|---|---|---|
| User stories total | 510 | |
| No `Sequence` in `sd:satisfiedBy` at all | 366 | of which 239 already have an `sd:Annotation` (mostly `coverage-gap`/open) — already tracked, see below |
| ...not yet annotated | **127** | this review's real net-new surface |
| `Sequence` present but none `sd:initiatedBy "user"` | 23 | mostly false positives — see §4 |
| User-initiated sequence with zero `api-call` references | 8 sequences / 27 stories | all IAM — see §1 |
| `api-call` reference that didn't resolve to a modeled operation | 35 (9 "no-such-endpoint", 26 "ambiguous") | see §2 |
| `ApiOperation` with no `sd:PermissionRequirement` | 8 operations | see §3 |

## §1 — Sequences with an existing, matching API operation they don't reference

8 user-initiated IAM sequences have **zero** `sd:SequenceReference` of kind `api-call` — their
mermaid content never shows the actor's own action hitting the platform's own API, only
`external-call`s (to Mi-Key / MiLogin) and `event-emit`s. For 6 of the 8, an `sd:ApiOperation`
that's obviously *the* operation this sequence is about already exists in `iam-domain` — it's
just not drawn in the diagram:

| Sequence | Stories it backs | Existing operation it should reference |
|---|---|---|
| `iam:Seq_IdentityAdministratorResolveRequest` | 15.3.3, 15.3.4, 15.6.2, 15.6.3, 15.9.2, 15.9.3, 15.9.4, 24.5.2 | `iam:Op_adminResolveIdentityRequest` (`POST /identity-resolution/requests/{requestId}/admin-resolve`) |
| `iam:Seq_CitizenUpdateAccountDemographics` | 15.3.2, 15.10.1, 15.10.2, 15.10.4, 15.10.5 | `iam:Op_submitCitizenAccountUpdate` (`POST /citizen-identity/account-update`) |
| `iam:Seq_CitizenCancelPendingUpdate` | 15.10.4 | `iam:Op_cancelOwnCitizenIdentityRequest` (`POST /citizen-identity/requests/{requestId}/cancel`) |
| `iam:Seq_SystemAdminManualGrant` | 1.2.3, 1.4.4, 10.1.2 | `iam:Op_manualGrantAuthorization` (`POST /authorizations/manual-grant`) |
| `iam:Seq_ScopeApproverApproves` | 1.6.4 | `iam:Op_approveAuthorizationRequest` (`POST /authorization-approvals/{token}`) |
| `iam:Seq_BusinessUserInitialAuthorizationRequest` | 1.1.2, 1.6.1, 1.6.2, 1.6.3 | `iam:Op_submitBusinessUserIdentityRequest` (`POST /identity-resolution/requests`) — note also flagged in §3, this operation has no `sd:PermissionRequirement` |

**Judgment: valid gap, cheap fix.** Not a missing-API problem — it's a documentation/modeling
completeness gap in the sequence's own mermaid `sd:content`. Recommend adding the matching
`api-call` step (and a `sd:SequenceReference` for it) to each of these 6 sequences' diagrams.

The remaining 2 — `iam:Seq_CitizenInitialSignIn` and `iam:Seq_MiLoginAuthenticationIntegration` —
are **not** gaps: both are genuinely about redirecting to/round-tripping with the external MiLogin
IdP and Mi-Key, which is exactly what their `external-call` references already capture. No
internal API call is expected there.

## §2 — Unresolved `api-call` references

35 `sd:SequenceReference`s of kind `api-call` carry `sd:resolutionStatus` other than
`"resolved"`. Split by status, with a spot-check against the modeled operations in each
domain:

### "no-such-endpoint" (9) — checked individually against that domain's `sd:ApiOperation` list

| Sequence | Raw reference | Verdict |
|---|---|---|
| `epp:Seq_UpdateCandidateEnrollmentStatus` | `GET /candidate-enrollments/{enrollmentId}/available-transitions` | **Real gap.** `epp:Op_UpdateCandidateStatus` and `epp:Op_GetCandidateEnrollment` exist but nothing exposes the valid-next-states list this read implies. Proposed: add `epp:Op_GetCandidateEnrollmentTransitions` (`GET /candidate-enrollments/{enrollmentId}/available-transitions`), gated by the same permission as `Op_UpdateCandidateStatus`. |
| `epp:Seq_RecommendCandidateForCredential` | `GET /epp-providers/{eppCode}/approved-endorsements` | **Real gap.** `Op_GetApprovedPrograms` and endorsement CRUD exist, but no read endpoint returns an EPP's *approved* endorsement set. Proposed: `epp:Op_GetApprovedEndorsements` (`GET /epp-providers/{eppCode}/approved-endorsements`). |
| `payments:Seq_BulkPaymentProcessing` | `GET /applications/pending-payment?districtId=...` | **Real gap.** Closest existing ops are refund/reconciliation-shaped, not "applications awaiting payment." Proposed: `payments:Op_ListApplicationsPendingPayment` (`GET /applications/pending-payment`). |
| `profpractices:Seq_ConfigurePPRWorklist` | `GET /worklists/{id}` | **Likely real, minor gap.** `profpractices:Op_ListWorklists` and `Op_UpdateWorklist` exist but there's no single-item `GET`. Proposed: add `profpractices:Op_GetWorklist` (`GET /worklists/{worklistId}`). |
| `staffing:Seq_AssignEmployeeToPosition` | `GET /employee-roster/search-active` | **Probably not a real gap — likely a stale/near-miss resolution.** `staffing:Op_SearchEmployees` (`GET /employee-roster/search`) already exists and almost certainly is this call, just missing an `active`-only filter param in its `sd:parameters`. Recommend re-pointing this reference at `Op_SearchEmployees` rather than minting a new operation. |
| `profpractices:Seq_ManageAccountMarkers` | `GET /educators/search` | **Not a real gap — stale resolution.** `staffing:Op_SearchEducators` (`GET /educators/search`) exists with an exact path match; this looks like a cross-context resolution the authoring/resolution pass simply didn't make (this sequence lives in `profpractices`, the operation in `staffing`). Recommend re-pointing the reference. |
| `proflearning:Seq_AddAttendeesToProgram` | `GET /educators/search?query={name}` | Same as above — same `staffing:Op_SearchEducators` match. Re-point, don't mint a new op. |
| `staffing:Seq_BulkRefundProcessing`\* | `GET /payments?status=Paid&districtId=...` | **Probably not a real gap.** `payments:Op_SearchPayments` (`GET /payments/search`) is almost certainly this call with different query-string shape than path shape. Recommend re-pointing / aligning `sd:path`. |
| `documents:...` bulk ops | *(see "ambiguous" below — several document-service refs are genuinely new endpoints, not near-misses; documents' API surface is the least mature of the domains touched here)* | — |

\* sequence lives in `payments:` context — corrected from the raw file naming.

**Net:** of the 9 flagged "no-such-endpoint," 4 are real missing operations (worth adding), 4 are
stale/near-miss resolutions against an operation that already exists under a slightly different
path or in a neighboring context (fix the reference, not the API surface), and the documents
domain's set is covered next.

### "ambiguous" (26)

All 26 are in `documents`, `proflearning`, `epp`, `staffing`, `profpractices`, and `payments`
sequences whose raw text is a `/api/...`-prefixed or slightly-reshaped path relative to how that
domain's `sd:ApiOperation.sd:path` is recorded (e.g. `documents:Seq_DocumentUploadWithMalwareScanning`'s
`POST /api/documents/upload/request` vs. the modeled `documents:Op_RequestDocumentUpload`'s bare
`/documents/upload/request`). These read as a **path-notation inconsistency**, not missing API
surface — `documents` is the newest/least-mature domain in this solution (few operations, no
wireframes yet), so its raw sequence text was authored before its operations were normalized.
Recommend a documents-domain path-normalization pass (either add a consistent `/api` prefix
convention across all `sd:ApiOperation.sd:path` values, or strip it from the sequence text) rather
than treating each one as an individual gap.

## §3 — API operations with no `sd:PermissionRequirement`

| Operation | Verdict |
|---|---|
| `documents:Op_RequestDocumentUpload`, `Op_RequestDocumentReplacement`, `Op_InitiateBulkUpload` | **Real gap.** Uploading/replacing/bulk-uploading a document against a citizen or entity record is exactly the kind of action this solution otherwise always gates — every comparable write elsewhere has a `sd:PermissionRequirement`. Proposed: a `documents`-scoped upload/modify permission (e.g. `Perm_DocumentUpload`, scope matching whatever the containing record's access rule is), applied to all three operations. |
| `documents:Op_ReceiveScanResult` | Not a gap — Azure Defender→DocsAPI service callback, no human actor. |
| `profpractices:Op_ReceiveRapBackNotification` | Not a gap — external-system callback. |
| `payments:Op_ConfirmPayment` | Not a gap — CEPAS redirect/callback, not a direct user action. |
| `proflearning:Op_SearchCatalog` | Not a gap — its own `sd:summary` ("Search **public** program catalog") says this is the intentionally-unauthenticated surface; pair with `sd:requiresAuth false` if not already set. |
| `iam:Op_submitBusinessUserIdentityRequest` | **Worth a look, lower confidence.** Any authenticated business user can apparently call this with no additional permission check. May be intentional (base authentication is the only gate for *starting* an identity request), but confirm with IAM owner — every other identity-resolution write operation in the same file (`adminResolveIdentityRequest`, `escalateIdentityRequest`, `selfResolveIdentityRequest`) does have one. |

## §4 — User-initiated stories mapped only to a non-user-initiated sequence

23 stories `sd:satisfiedBy` a sequence, but none of those sequences are `sd:initiatedBy "user"`.
Reading each: **19 of the 23 are false positives** of this specific check, not real gaps — the
story itself is phrased `"As the MiEdWorkforce System..."` / `"As the system..."`, so an
`internal-system`- or `external-system`-initiated sequence (a batch job, an event-triggered
process) is the *correct* satisfier, exactly as the ontology's own comment on `sd:initiatedBy`
anticipates. Examples: 21.3.1–21.3.3 and 18.3.1–18.3.9 (`Open New Collection`, a scheduled batch
job), all five 8.2.x NASDTEC stories (`Process NASDTEC Nightly Batch`), 7.2.1/7.2.3 (payment
reconciliation batch jobs), 15.12.1/15.11.1 (Mi-Key-Direct admin/system-only path, correctly
external-system-initiated by design per 15.11.1's own statement).

**4 are real mismatches** — a human actor's own action is mapped only to a system/external-initiated
sequence that doesn't actually carry out *their* part of the story:

| Story | Statement (abridged) | Mapped to | Judgment |
|---|---|---|---|
| `profpractices:Req_30_1_1` | Rapback Admin wants to **view** a consolidated list of Rapback records | `Receive Rap Back Notification` (external-system) | That sequence only covers *ingesting* the notification, not the admin's own worklist/list view. Proposed sequence: **"Rapback Admin Reviews Consolidated Rapback Worklist"** (user-initiated) — admin opens worklist, lists/filters Rapback records, drills into one. Needs a `GET /rapback-records` (or similar, list+filter) API operation; check whether `profpractices` already models one before adding — none found in this pass. |
| `staffing:Req_18_3_9` | School District user wants to **see** data-quality indicators before certification | `Open New Collection` (internal-system) | The batch job seeds the collection; a district user viewing DQ indicators on their own roster is a separate, user-initiated read. Proposed sequence: **"District User Reviews Data Quality Indicators"** — likely already partly covered by the `dataquality` domain's own worklist/Action Item sequences; recommend checking there before minting a new one (may just need the `satisfiedBy` edge re-pointed at an existing `dataquality` sequence rather than a wholly new one). |
| `staffing:Req_18_3_4` | School District user wants to **view** last year's carried-forward roster | `Open New Collection` (internal-system) | Same shape as above — the batch job does the carry-forward; viewing it is a separate user action. Proposed sequence: **"District User Views Prior-Year Roster on Collection Open"** (user-initiated), API: likely just a filtered `GET` on the existing employee-roster/position list operations (`staffing:Op_ListPositions` / roster search) scoped to the prior period — probably doesn't need a new operation, just a new sequence documenting the existing read. |
| `iam:Req_15_4_2` | Business/Admin user wants to **view a sortable list** of matching records from their own search | `Identity Resolution - Split/Retire ID (Mi-Key-Direct)` (external-system) | That sequence is the admin-only Mi-Key-direct split/retire path, not a business user's own search-results view. This looks like a mis-linked `satisfiedBy` edge rather than a true gap — `iam:Op_listIdentityResolutionRequests`/`searchUsers` already exist and are plausible satisfiers. Recommend re-pointing `sd:satisfiedBy` at whichever existing user-initiated sequence shows the search-results list (or, if none does yet, mint one), rather than adding new API surface. |

## §5 — 127 stories with no sequence and no existing review annotation

Every one of these 127 already has *some* `sd:satisfiedBy` target (an `ApiOperation`,
`Aggregate`, `BusinessRule`, etc.) — none are satisfied by literally nothing. Split by whether
that existing satisfier is operational:

### 5a. 38 stories with no `ApiOperation`/`BatchJob` among their satisfiers at all

The higher-priority subset — nothing executable backs these yet, only domain-model/business-rule
nodes. Judged individually:

**Genuinely backend/system stories — valid as-is, no sequence needed:**
`staffing:22.3.2` (validation rule), `21.2.3` (system validation), `18.1.4`/`18.3.10`/`18.4.5`
(system-computed warnings/state checks), `22.2.2`/`22.3.3` (backend education-history
processing — `22.3.3`'s "view pre-populated data" is arguably a read the district-roster UI
already shows, not a separate flow), `iam:15.1.2`/`15.1.3` (Mi-Key matching backend),
`dataquality:19.4.1`/`19.3.3`/`19.1.9` (event/rule-driven Action Item mechanics),
`profpractices:9.1.2`/`8.2.7`/`8.2.5` (audit logging / backend security rules),
`proflearning:8.3.1`/`8.3.2`/`8.3.4` (STARR external-data ingestion — backend, likely a
`BatchJob` that just isn't linked via `sd:satisfiedBy` yet), `communications:6.2.3`/`6.1.3`
(email-merge and SendGrid integration — backend). These are correctly satisfied by a
`BusinessRule`/`DomainEvent`/`Aggregate` alone; no action needed beyond, where noted, confirming
the matching `BatchJob` gets linked.

**Real gaps — recommend a new sequence (+ API where noted):**

| Story | Statement (abridged) | Proposed sequence | API note |
|---|---|---|---|
| `fdd32:32.16.8` | Version control when replacing an uploaded document | **"Document Replace Preserves Prior Version"** | `documents:Op_RequestDocumentReplacement` already exists (§1/§3 already flag it for permission); this story is really about that operation's *behavior*, already indirectly covered — low priority, mostly a documentation link, not new API. |
| `fdd32:32.15.1/.2/.4/.5/.6` | Admin creates/manages a worklist scoped to a specific domain (Credential type, Identity mgmt, EPP, Professional Learning, Rapbacks) | **"Administrator Configures Worklist for [Domain]"** (one per domain, or one generic parameterized sequence) | `profpractices:Op_CreateWorklist`/`Op_UpdateWorklist`/`Op_ListWorklists` already exist (found while checking §2) — these stories likely just need `sd:satisfiedBy` pointed at a sequence using that existing API, not new endpoints. |
| `proflearning:28.4.4` | Admin adds/edits program event dates, locations, County, start/end times | **"Administrator Manages Program Event Schedule"** | No matching CRUD operation found for program *events* specifically (`proflearning` ops seen so far are session/attendance/award-shaped) — check `proflearning-domain.ttl` directly; if absent, propose `proflearning:Op_UpdateProgramEvent` (`PATCH /programs/{id}/events/{eventId}`). |
| `iam:15.10.3` | Citizen adds a newly-acquired SSN to complete their profile | **"Citizen Adds SSN to Existing Account"** | No dedicated operation found; `iam:Op_submitCitizenAccountUpdate` may already cover this as a field update — confirm before adding a new endpoint. If it doesn't cover SSN specifically (SSN is often handled as a distinct, more-sensitive field with its own validation/audit), propose `iam:Op_addAccountSsn` (`PATCH /citizen-identity/account/ssn`), permission-gated. |
| `iam:15.9.6` | Identity Admin manages identity-processing system configuration (validations, manuals) | **"Identity Administrator Manages Processing Configuration"** | No configuration-surface operation found in `iam` ops list. Propose `iam:Op_getIdentityProcessingConfig` / `Op_updateIdentityProcessingConfig`, both permission-gated (the one satisfier this story already has is a `Permission`, so the permission side is anticipated — the operation and sequence are the missing half). |
| `profpractices:9.5.4` | User exports reports (PDF/Excel/CSV) | **"User Exports Report"** | Check the `reporting` bounded context first (FDD 02) — this looks like generic cross-cutting export functionality that may already be modeled there under a different story; don't duplicate if so. |
| `profpractices:9.2.4` | PPR Reviewer/Credential Admin views audit log on a worklist item | **"Reviewer Views PPR Worklist Item Audit Trail"** | `profpractices:Op_GetRemarksHistory` exists and is close but is remarks-specific, not a generic audit log; likely needs its own read, e.g. `Op_GetApplicationReviewAuditLog`. |
| `profpractices:9.2.3` | Credentialing Admin manages PPR disclosure questions | **"Administrator Manages PPR Disclosure Questions"** | No question-bank CRUD operation found in the `profpractices` list pulled so far — likely a real gap; propose `profpractices:Op_listDisclosureQuestions`/`Op_updateDisclosureQuestion`. |
| `communications:6.3.4` | System Admin views audit details for email templates | **"Administrator Views Email Template Audit Log"** | No matching read operation found among ops touched in this pass; check `communications-domain.ttl`'s full operation list before adding — this pass only sampled operations reached via sequences, not the full domain file. |

### 5b. 89 stories already satisfied by an `ApiOperation`/`BatchJob` directly, just no sequence diagram

Lower priority — these already have real, executable backing; they just lack a drawn sequence
showing the flow (useful for wireframe/UX documentation, not a functional gap). Grouped by
bounded context:

| Context | Count |
|---|---|
| `staffing` | 30 |
| `proflearning` | 23 |
| `iam` | 8 |
| `profpractices` | 7 |
| `communications` | 6 |
| `organizations` | 4 |
| `fdd32` | 3 |
| `epp` | 3 |
| `payments` | 3 |
| `dataquality` | 1 |
| `reporting` | 1 |

Recommend backfilling sequence diagrams for these opportunistically (e.g. whenever that
operation's domain is next touched for wireframe capture) rather than as a dedicated pass — the
API contract already exists and is the harder half of the work.

## §6 — 239 stories already flagged, no new action needed here

366 − 127 = 239 stories with no `Sequence` satisfier already carry at least one `sd:Annotation`.
Breakdown by `sd:findingType`/`sd:status` (a story can carry more than one annotation):

| findingType / status | Count |
|---|---|
| `coverage-gap` / open | 195 |
| `discrepancy` / open | 18 |
| `unconfirmed-assumption` / open | 14 |
| `out-of-scope` / resolved | 15 |
| `contradiction` / open | 10 |
| *(untyped commentary)* / open | 25 |
| *(untyped commentary)* / resolved | 1 |
| `rejected` / resolved | 1 |

The large `coverage-gap`/open bucket is this exact class of finding, already tracked story-by-story
by whoever ran the FDD-by-FDD review pass — this pass doesn't re-derive or duplicate those; it
only surfaces the **127 that slipped through without one** (§5) plus the sequence/API/permission-level
findings (§1–§4) that a per-story annotation pass wouldn't have caught, since those live one or two
hops further down the chain from the story itself.

## Next steps

1. Fix the 6 IAM sequences in §1 (cheap — add the missing `api-call` reference to each diagram).
2. Re-point the ~5 stale/near-miss `resolutionStatus` references identified in §2 rather than
   minting new operations for them; add the ~4 genuinely new operations identified there.
3. Add `sd:PermissionRequirement` to the 3 `documents` upload operations in §3; confirm the
   `iam` one with the IAM owner.
4. Re-point the `satisfiedBy` edge for `iam:15.4.2` in §4 rather than treating it as missing API
   surface.
5. Work the §5a real-gap table (9 stories) — each needs a sequence minted and, where noted, a
   confirmed-missing operation added; several may turn out to already have partial coverage
   elsewhere once checked directly against the full domain file (this pass only saw operations
   reached via a sequence reference, not a domain's complete `ApiOperation` list).
6. §5b (89 stories) and wireframe coverage generally: backfill opportunistically, not urgently.

## Follow-up: external-system resolution fixes and corrections to this review

A second pass went sequence-by-sequence through every resolved `api-call`/`external-call`
reference to judge whether the operation it points at actually satisfies the spirit of that
sequence step and its user story, and specifically to check that calls to genuine third-party
systems are modeled as `sd:ExternalSystem` references rather than left dangling. That surfaced
one real, structural defect, which is now fixed directly in the ttl — and, in the course of
verifying candidate fixes from §2–§5 of this review against their own already-recorded
reasoning, several were found to be **not** real gaps after all. Both are recorded below.

### Fixed: no `SequenceReference` had ever resolved to an `sd:ExternalSystem`

20 `sd:ExternalSystem` instances exist (Mi-Key, MiLogin, CEPAS, FTS, CEPI, NASDTEC, the RapBack/
CHRISS system, Power BI, SendGrid, and others) and the ontology's own comment on `sd:resolvesTo`
explicitly says its range includes `sd:ExternalSystem` "so a sequence/batch job that calls out to
an external system directly... still gets a resolvable reference, instead of only being
describable in free text." Despite that, **zero** `sd:SequenceReference` anywhere in the graph
actually pointed `sd:resolvesTo` at any `ExternalSystem` node. Every one of the ~33 sequence steps
that call one of these systems had instead been left `resolutionStatus "unresolved"`, each
carrying its own note explaining, correctly, that "CEPAS is an sd:ExternalSystem; no valid
sd:ApiOperation target under current shapes" (or the equivalent for Mi-Key/MiLogin/CEPI/FTS/
NASDTEC/RapBack/PowerBI/SendGrid) — i.e. the authors had already correctly diagnosed *why* no
`ApiOperation` fit, without taking the next step the ontology already permits: pointing
`resolvesTo` at the `ExternalSystem` node itself instead of at an `ApiOperation`.

**Fixed all ~33 of these** — each now has `sd:resolvesTo` set to the correct `ExternalSystem`
instance (matched by the system explicitly named in the reference's own `raw` mermaid text —
Mi-Key, MiLogin, CEPAS, FTS, CEPI, NASDTEC, RapBack/CHRISS, Power BI, or SendGrid) and
`sd:resolutionStatus` changed from `"unresolved"` to `"resolved"`. The old explanatory note on
each is replaced with a short note pointing at this fix, so the reasoning trail isn't lost. One
additional, unrelated resolution miss was fixed alongside these: `epp:Seq_BulkUploadCandidateTrackingData`'s
"Synapse->>IAM: GET /users/by-unique-id/{uniqueId}" step was marked unresolved even though
`iam:Op_GetUserByUniqueId` — an internal-service operation explicitly documented as "Called by
other domain services... not exposed via APIM" — is an exact path match; that's now linked too.

Net effect: `external-call` references resolved from 78/133 to 112/133. The remaining 3 unresolved
`external-call`s and all 9 `no-such-endpoint`/9 `ambiguous` ones are addressed below — every one of
them turned out to already be a deliberately-researched, correctly-flagged gap (see next section),
not something this pass should silently "fix."

Script used: a throwaway Node script did exact-match text substitution directly against the
serialized ttl (not a full parse-and-re-serialize round-trip through oxigraph), specifically to
keep the diff to just the ~3 changed lines per reference and avoid re-formatting the other ~26,000
lines of the file. `git diff --stat` on the result: 102 insertions / 68 deletions across one file.

### Corrected: several §2–§5 findings do not hold up under closer reading

Before touching anything else this review had flagged, each candidate fix was checked against
that node's own existing notes/context first — and it turned out this file's authors had already
done far more rigorous, source-cited verification than this review's first pass had. Specifically:

- **§3's 3 "missing permission" document operations are not a gap.** `documents:Op_RequestDocumentUpload`/
  `Op_RequestDocumentReplacement`/`Op_InitiateBulkUpload` each carry an explicit
  `x-permissions-required: []` note: they're secured by Managed Identity/mTLS at the platform
  level, and the *calling* domain (not Documents) is responsible for the business-authorization
  check before it calls Documents. Adding a `sd:PermissionRequirement` here would misrepresent a
  deliberate architectural boundary. **Not changed.**
- **§5a's 4 "add a new API operation" proposals (`epp` available-transitions/approved-endorsements,
  `payments` applications-pending-payment, `profpractices` GetWorklist) are already
  known, confirmed gaps, each individually cross-checked against its own source API spec** (e.g.
  "No such path in payments-api.yml. Closest modeled operation is pay:Op_SearchPayments... searches
  PaymentTransactions, not credential applications pending payment — a different resource
  entirely. Left unresolved rather than force-matched.") Minting new operations for these without
  the same source-document access those authors had would be lower-confidence than what's already
  recorded. **Not changed** — left as the correctly-flagged `no-such-endpoint` gaps they already were.
- **§2's "stale near-miss" hypotheses were wrong.** This review guessed that
  `staffing:Seq_AssignEmployeeToPosition`'s `GET /employee-roster/search-active` and two
  `GET /educators/search` references (`profpractices:Seq_ManageAccountMarkers`,
  `proflearning:Seq_AddAttendeesToProgram`) were just un-linked near-misses of
  `staffing:Op_SearchEmployees`/`Op_SearchEducators`. On closer reading: the roster one is
  confirmed missing an "active-only" filter parameter that doesn't exist on the real operation
  either (not just a naming mismatch), and the educator-search ones are the same
  already-independently-confirmed gap cited from an FDD acceptance-criteria review ("Confirming,
  not raising, that gap" — traced to BRD 26.5.1's requirement for a *global*, not
  organization-scoped, search, which is a materially different capability than what
  `staffing:Op_SearchEducators` provides). **Not changed.**
- **§4's `iam:Req_15_4_2` "mis-linked satisfiedBy" call was wrong.** Its `satisfiedBy` edge to
  `iam:Seq_SplitRetireIdMiKeyDirect` is deliberate, partial-credit modeling — documented in the
  story's own notes as capturing just the Retired-ID display rule (a real, narrow slice this
  sequence's own key decision covers), while the broader "Person Search" capability it also needs
  is already tracked separately as `iam:Question_PersonSearchNotModeled`, explicitly to avoid
  double-counting the same gap under two different nodes. **Not changed.**

The pattern across all four: this graph's existing "unresolved"/"no-such-endpoint"/`satisfiedBy`
markings are, in the areas checked, already the product of real source-document cross-referencing
— not gaps in care, but gaps this review's first pass mistook for gaps because it worked from
path-text pattern-matching rather than each node's own recorded reasoning. The lesson carried
into the one fix that *was* applied: the external-system non-resolution was safe to fix precisely
because every one of those ~33 notes agreed on the same specific, mechanical cause (`ApiOperation`
can't have `exposedBy` an `ExternalSystem`) rather than disagreeing or hedging — a categorical
tooling gap, not a case-by-case judgment call.
