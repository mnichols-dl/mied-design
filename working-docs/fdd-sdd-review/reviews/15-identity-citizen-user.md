# Review: User Management - Identity Management Integration, Citizen-User/IAM Pass (15, Pass B)

| Field | Value |
|---|---|
| Source Document(s) | `15 - User Management - Identity Management Integration/15.8 - Citizen User Request for ID Requirements.docx`<br>`15 - User Management - Identity Management Integration/15.10 - Citizen User Update Record Requirements.docx`<br>`15 - User Management - Identity Management Integration/15.3 - Citizen User - Near Match Resolution.docx`<br>`15 - User Management - Identity Management Integration/15.4 - Person Search.docx` (iam-facing content only — primarily a staffing/business-user document; see Summary)<br>`15 - User Management - Identity Management Integration/15.13 - Email Comms Requirements.docx`<br>`15 - User Management - Identity Management Integration/15.7; 15.14; 15.15 - Identity Management Integration.docx`<br>`15 - User Management - Identity Management Integration/15.9 - Admin Identity Requirements.docx` (iam-facing content only — staffing-facing content already reviewed in Pass A) |
| Related Domain(s) | iam (primary); staffing (cross-reference only, not re-reviewed); communications (cross-reference only) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-31 |
| **Overall Status (per document)** | 15.8: **Aligned — Minor SDD Revisions Needed** (core Near Match gap fixed directly). 15.10: **Aligned — Minor SDD Revisions Needed** (new sequence drafted directly). 15.3 (Citizen): **Gaps Identified** (Data Precedence Selection on admin resolution not modeled). 15.4 (iam-facing slice): **Discrepancy — Needs Decision** (Request-to-Split-ID contradicts Pass A's resolved finding). 15.13: **Aligned**. 15.7;15.14;15.15: **Not Applicable / Superseded** (confirmed stub — all three features explicitly withdrawn by the client). 15.9 (iam-facing slice): **Gaps Identified** (admin-initiated No-SSN citizen creation not modeled as a distinct actor path). |

## Summary

This is the citizen-facing half of FDD 15's identity-management process, completing Pass
A's staffing/business-user half. The single biggest finding: **`iam-domain.md` had no
aggregate at all for the citizen identity-matching "waiting room"** — the existing "Citizen
User - Initial Sign-In & Identity Resolution" sequence treated Mi-Key matching as one
opaque step with no Near Match branch, despite FDD 15.8/15.10/15.3 requiring one with real
behavioral differences from the staffing-side Near Match path (Account Creation fully
blocks dashboard access on Near Match; Account Update does not, but locks Primary/Secondary
fields while leaving Contact fields editable; the citizen is never shown candidate match
data, unlike nothing-hidden staffing flows). There was also **no sequence at all for citizen
self-service demographic updates** (FDD 15.10) and **no admin-side resolution sequence**
for the "Requires Resolution" worklist entries FDD 15.9 describes. All three are fixed
directly in this review: a new `CitizenIdentityRequest` aggregate (mirroring staffing's
`ID_RESOLUTION_REQUESTS` pattern) was added to `iam-domain.md`, the Initial Sign-In sequence
was extended with the missing Near Match branch, and two new sequences ("Citizen User -
Update Account Demographics" and "Identity Administrator - Resolve Citizen Identity
Request") plus a small "Cancel Pending Update Request" sequence were added to
`iam-sequences.md`, backed by new endpoints in `iam-api.yml`. A near-duplication was caught
and corrected in the process: FDD 15.9's "Update Person Records" and worklist-management
capabilities are a *single* cross-domain Identity Administrator capability that Pass A
already permissioned generically in `staffing-permissions.md` (`staffing.identity-admin.
manage-worklist`'s own description already says "and citizen Requires-Resolution
requests"; `staffing.identity-admin.update-person-record` already reads "a person's
demographic record," not staffing-specific) — this review's new iam-side endpoints require
those existing `staffing.identity-admin.*` permissions rather than defining redundant
`iam.*` ones, keeping endpoint ownership (iam, since the aggregate lives there) separate
from permission ownership (staffing, per Pass A). `15.13` (Email/Comms admin config for
identity) needed no SDD change: `communications-capability.md`'s `EmailTemplate` aggregate
already scopes templates by `functional_area`, and `IAM` is already a listed value in that
enum — this feature is a client-facing view into a capability the SDD already supports.
`15.7;15.14;15.15` is confirmed a stub exactly as flagged in the tracker: all three named
features (Historical Records, ID Migration & Conversion, Citizen File Upload) are
explicitly withdrawn by the client's own text ("determined it was not needed"/"no longer
necessary"). `15.4 - Person Search` is overwhelmingly a staffing/business-user document
(no citizen actor appears anywhere in it) and its core richness gap (SSN/CLN/Application
Number search, retired-ID display, record comparison) is staffing's own open item — per
this review's task scope, that content is left for the staffing domain to resolve rather
than re-opened here. However, 15.4's own Addendum (15.4.5) surfaces a genuine new
discrepancy squarely in scope for this pass: it gives Business/Admin users a "Request to
Split ID" affordance from the Person Detail view, directly contradicting Pass A's resolved
finding (based on FDD 15.11's Addendum 15.11.1) that Splitting is Admin/Mi-Key-backend-only
with no Business-User-facing UI. This is flagged as a new Discrepancy and escalated to
`client-questions.md`.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | `15.8`/`15.3` (Citizen)/`15.10` "Gap Analysis" sections (all three, near-identical NFR lists) | Specific performance thresholds ("page load times under 3s," "10-second performance NFR"), encryption standards ("TLS 1.3/1.2, at-rest"), and named security mechanisms (RBAC implementation detail, XSS/SQL injection input sanitization) | These are technical/non-functional requirements documents embedded inside functional-design "Gap Analysis" addenda — implementation-level detail (specific TLS versions, specific timing budgets) rather than functional business rules. | Not blocking — treat as informing platform-wide NFR baselines already assumed elsewhere in the SDD (e.g., general security/performance conventions), not as new identity-specific technical design. No SDD change needed; these read as a consultant's gap-analysis exercise rather client-authored functional requirements, consistent with how the near-identical NFR boilerplate repeats verbatim across 15.8/15.3/15.10's Gap Analysis sections. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | FDD vs. itself | Whether Business/Admin users can initiate an ID Split request from the UI | `15.6; 15.11; 15.12` Addendum 15.11.1 (reviewed in Pass A): "Business Users (school districts) cannot request an ID split from the MiEdWorkforce UI... When an Admin splits an ID directly in Mi-Key, MiEdWorkforce receives the callback/update." Pass A resolved this internally as the authoritative reading (see `staffing-domain.md` Open Question #10: "Split and Retire... are admin/backend-only... need no Business-User-facing SDD surface"). | `15.4 - Person Search` Addendum 15.4.5: "As a Business or Admin User, I want to initiate administrative actions directly from the Person Detail view... If the user identifies that a single Unique ID is being shared by multiple individuals, a 'Request to Split ID' option is available." | Unlike Pass A's Link/Split self-contradiction (where the main narrative's contradictory bullets were resolved by a clearly more specific, later Addendum in the *same* document), this is a genuine conflict **between two different documents' Addenda** — both are acceptance-criteria-level, both are equally structurally authoritative, and neither reads as an obvious copy/paste artifact of the other. This is not a case the team can resolve by picking the "more specific" reading, unlike Pass A's internal case. | **Needs Client Clarification** — see `client-questions.md` #10. Until resolved, no SDD change made on either side: `staffing-domain.md` Open Question #10's existing framing ("Split... admin/backend-only") is left as-is rather than silently overturned by a single addendum sentence, but flagged as now genuinely contested. |
| 2 | FDD vs. SDD | Whether Citizen Near Match handling is a single flow or has materially different states depending on whether the citizen is creating vs. updating their account | `15.8` §15.8.5: Account Creation Near Match fully blocks dashboard access. `15.10` §15.10.4: Account Update Near Match does **not** block dashboard access, locks only Primary/Secondary fields (not Contact), and the citizen retains a self-cancel option. | Prior to this review, `iam-domain.md`/`iam-sequences.md` had zero representation of either flow — there was nothing to compare against. | FDD is internally consistent (Creation vs. Update genuinely differ); the SDD simply didn't model either. Not a real discrepancy once fixed — recorded here because it explains why the fix required two different state-machine behaviors rather than one shared branch. | **Resolved by drafting directly** (2026-08-31) — see `CitizenIdentityRequest` aggregate and both new sequences in `iam-domain.md`/`iam-sequences.md`. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | `15.3` (Citizen) Gap Analysis, "Missing Admin Action - Data Precedence Selection": "when an admin confirms a 'True Match,' they must have the ability to select which specific user-submitted data elements... should be considered prevalent" to update the Mi-Key master record | The admin resolution action (`CitizenIdentityResolutionAction` in `iam-api.yml`, added this review) currently supports `Match`/`CreateNew`/`Cancel` but has no field-level data-precedence selection when resolving as `Match` — the FDD explicitly calls this out as a documented gap in the FDD's own review process, not just an SDD gap. | `iam-api.yml`'s `CitizenIdentityResolutionAction` schema (extend with a field-selection map, e.g. `precedentFields: string[]`); `iam-sequences.md`'s "Identity Administrator - Resolve Citizen Identity Request" sequence (extend the Match branch) | M | Not drafted — needs a UI/UX decision (side-by-side field comparison control) the team should own deliberately, similar in shape to Pass A's undrafted Link IDs gap. |
| 2 | `15.9` §"Citizen Account Creation (for users with No SSN)" | Admin-initiated Account Creation submission (Identity Administrator fills out the Citizen form on the citizen's behalf, with the "No SSN" checkbox) is a distinct actor path from the self-service `POST /citizen-identity/account-creation` added this review, which is gated by `iam.citizen-identity.submit-request` (Citizen users only per its permission catalog entry). No admin-actor variant exists. | `iam-api.yml` — either a new `x-access`/permission variant on the same endpoint (Identity Administrator can submit on behalf of a not-yet-authenticated citizen) or a dedicated `/citizen-identity/admin-account-creation` endpoint; `iam-permissions.md` needs a corresponding permission (e.g. `staffing.identity-admin.*` extension, per this review's established pattern of routing Identity Administrator actions through the staffing-owned permission category) | M | Not drafted — endpoint/permission shape decision needed (single endpoint with actor-aware behavior vs. separate endpoint) before implementation. |
| 3 | `15.4 - Person Search` (entire document, staffing-facing richness) | SSN/CLN/Application Number search fields, retired-ID display and linkage, multi-record comparison — see Assessment below. | `staffing-domain.md`/`staffing-api.yml` (staffing's own scope; see FDD 24 review's Open Question #3, which named this document as the one to resolve it) | — | **Not re-opened per this review's scope** (staffing-facing content, cross-referenced only). Resolution note: 15.4 describes this as a broader identity search "for search result inclusion, the identity record... must be established within the MiEdWorkforce system" — not restricted to a business user's own employee roster the way `staffing-api.yml`'s current `GET /employee-roster/search` is (`x-scope-sensitive: true`, "returns only employees within the organization specified by X-Organization-Context"). This resolves FDD 24 review's Open Question #3's framing question (extension of employee-roster search vs. separate capability) with a partial answer — it is broader than the existing roster search, not identical to it — but the actual endpoint/schema work remains staffing's to do. iam's own involvement is limited to field-level visibility restriction ("Data visibility... restricted based on the user's specific role and permissions" — already covered generically by the existing RBAC/permission-check model, no new iam gap). |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `15.8 - Citizen User Request for ID Requirements` §"Detailed Requirements for Citizen Account Creation Process"; Addendum 15.8.1-15.8.5 | iam | `CitizenIdentityRequest` aggregate (added); Sequence: Citizen User - Initial Sign-In & Identity Resolution (Near Match branch added); `POST /citizen-identity/account-creation` (added) | implements — core gap fixed directly this review |
| `15.10 - Citizen User Update Record Requirements` §"Detailed Requirements for Citizen Account Updates Process"; Addendum 15.10.1-15.10.5 | iam | `CitizenIdentityRequest` aggregate; Sequence: Citizen User - Update Account Demographics (added); Sequence: Citizen User - Cancel Pending Update Request (added); `POST /citizen-identity/account-update`, `POST /citizen-identity/requests/{requestId}/cancel` (added) | implements — no sequence previously existed at all; fixed directly |
| `15.10` §"If the Unique ID is associated with active employment, the Primary Demographic fields will remain read-only" | iam, staffing (cross-domain) | New "Citizen Near Match Resolution" business rule + "Update Account Demographics" sequence field-locking logic in `iam-domain.md`/`iam-sequences.md`; cross-references `staffing-domain.md`'s `EMPLOYEE_ROSTER` employment status, not re-modeled there | implements — thin cross-domain read dependency, no staffing-side change needed |
| `15.3 - Citizen User - Near Match Resolution` §"Detailed Requirements for Near Match Review User Process"; Gap Analysis (Cancel action, Data Precedence Selection, Matching Score) | iam | Sequence: Identity Administrator - Resolve Citizen Identity Request (added); `CitizenIdentityResolutionAction` schema (added) | implements (Match/CreateNew/Cancel outcomes); gap — Data Precedence Selection not modeled (Coverage Gap #1) |
| `15.9 - Admin Identity Requirements` §"Requires Resolution Requests" | iam | `CitizenIdentityRequest`; `GET /citizen-identity/worklist` (added, gated by `staffing.identity-admin.manage-worklist` per the cross-domain permission decision below) | implements — resolves the iam-side half of Pass A's deferred worklist scope |
| `15.9` §"Update Person Records" | iam, staffing (shared capability) | `PATCH /citizen-identity/person-records/{userId}` (added, gated by `staffing.identity-admin.update-person-record` — the existing staffing-owned permission, not a new iam one) | implements — corrected a near-duplication before it happened (see Summary) |
| `15.9` §"Citizen Account Creation (for users with No SSN)" | iam | `NoSSNStatus` value object on `CitizenIdentityRequest` (added); admin-actor submission path | gap — admin-initiated variant not modeled (Coverage Gap #2) |
| `15.13 - Email Comms Requirements` (entire document) | iam, communications (cross-domain) | `communications-capability.md`'s `EmailTemplate` aggregate, `functional_area` enum (already includes `IAM`); `EmailResend` capability | implements/aligned — no gap; the capability this feature configures already exists generically |
| `15.4 - Person Search` §Detailed Requirements, Addendum 15.4.1-15.4.4 | staffing (primary, not re-reviewed) | `staffing-api.yml` `GET /employee-roster/search` (existing, narrower scope) | gap — staffing's own open item (FDD 24 review Open Question #3); cross-referenced only, not resolved here (Coverage Gap #3) |
| `15.4` Addendum 15.4.5 ("Request to Split ID" from Person Detail view) | staffing (primary) | Contradicts Pass A's resolution of `staffing-domain.md` Open Question #10 | discrepancy — FDD vs. itself (Discrepancy #1); escalated to client |
| `15.7; 15.14; 15.15 - Identity Management Integration` (entire document) | iam | n/a — no aggregate; document explicitly withdraws all three features | Not Applicable / Superseded — no SDD action |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Does Business/Admin User "Request to Split ID" (FDD 15.4 Addendum 15.4.5) actually exist as a UI feature, or is it a documentation artifact carried over from the structurally-identical Link ID option? This directly contradicts FDD 15.11's Addendum 15.11.1 ("Business Users... cannot request an ID split from the MiEdWorkforce UI"), which Pass A relied on to conclude Split needs no Business-User-facing SDD surface. | Client | Open — see `client-questions.md` #10 |
| 2 | What is the numeric auto-cancellation timeframe for an unresolved Citizen Near Match (Account Creation or Update)? Mirrors `staffing-domain.md` Open Question #9 for the identical Business User-path question. | Internal | Open — see `iam-domain.md` Open Question #11 |
| 3 | Is the "Identity Administrator Worklist" a single cross-domain read-model spanning `staffing.ID_RESOLUTION_REQUESTS` and `iam.CITIZEN_IDENTITY_REQUESTS`, or two separately-owned queues composed together in the UI? | Internal | Open — see `iam-domain.md` Open Question #12 |
| 4 | Should the Identity Administrator be able to submit a Citizen Account Creation request on the citizen's behalf (FDD 15.9's No-SSN admin path), and if so, via the same endpoint with an admin actor variant or a dedicated admin endpoint? | Internal | Open — see Coverage Gap #2 |

## Related Documents

- Source documents: see header table above (all under `functional-design-docs/15 - User Management - Identity Management Integration/`)
- SDD documents reviewed against and modified: `solution-areas/iam/iam-domain.md` (new `CitizenIdentityRequest` aggregate, ERD entity, domain events, two business rules, Open Questions #11-#12 added), `solution-areas/iam/iam-permissions.md` (three citizen-only self-service permissions added; explicit note added declining to duplicate `staffing.identity-admin.*` permissions), `solution-areas/iam/iam-sequences.md` (Near Match branch added to Initial Sign-In sequence; three new sequences added: Update Account Demographics, Cancel Pending Update Request, Identity Administrator - Resolve Citizen Identity Request), `solution-areas/iam/iam-api.yml` (7 new endpoints + 5 new schemas under a new "Citizen Identity Resolution" section)
- SDD documents reviewed but NOT modified (staffing-facing content, cross-referenced only per this review's scope): `solution-areas/staffing/staffing-domain.md`, `staffing-permissions.md`, `staffing-sequences.md`, `staffing-api.yml`
- SDD documents reviewed, no gap found: `solution-areas/communications/communications-capability.md` (Email Comms, 15.13)
- Prior review this builds on: [reviews/15-identity-business-user.md](15-identity-business-user.md) (Pass A) — its deferred citizen-side scope (15.3 Addendum, 15.9's citizen worklist entries) is completed here; its Open Question #10 (`staffing-domain.md`, Split/Retire admin-only framing) is reopened by this review's Discrepancy #1, not silently resolved
- Also builds on: [reviews/24-request-for-id.md](24-request-for-id.md) — Open Question #3 (Person Search richness, "properly FDD 15.4's call") partially answered here (broader than roster search) but not fully resolved; remains staffing's open item
