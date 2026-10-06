# Review: Data Collection & Compliance - Staffing Admin (10)

| Field | Value |
|---|---|
| Source Document(s) | `10 - Data Collection & Compliance - Staffing Admin/10 - Data Collection & Compliance Staffing Admin.docx` (main FDD, reviewed via `.md`)<br>`10 - Data Collection & Compliance - Staffing Admin/Staffing Submission Validations.xlsx` (validation-rule matrix, reviewed via `.md`) |
| Related Domain(s) | staffing (primary); credentialing, profpractice, organizations, iam, communications, reporting (cross-domain) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-14 |
| **Overall Status (per document)** | `10 - Data Collection & Compliance Staffing Admin.docx`: **Gaps Identified**. `Staffing Submission Validations.xlsx`: **Boundary Violation**. |

## Summary

This is the first review of the `staffing` domain, and unlike credentialing/payments at
their first reviews, `staffing` already has a comprehensive domain, permissions, and
sequences SDD — `staffing-permissions.md`'s `Admin` category (`manage-collections`,
`manage-validations`, `manage-data-elements`, `process-exceptions`, `view-file-queues`,
`reset-file-processing`, `manage-reports`) already lines up closely with this FDD's admin
feature list, and most cross-domain deferrals (Alerts/Emails, Dashboards, Reports, BRM,
User Auth) match the FDD's own explicit dependency disposition. The main FDD narrative
also contains one genuine **internal contradiction** (its Business Specifications and High
Level Scenario #5 describe the Staffing Data Admin managing file-upload queues — reset,
delete, toggle availability — while its own Addendum Feature 10.14 explicitly disowns that
same capability as "outside scope of an administrative task... developer task"), plus a
handful of real coverage gaps (program-to-staffing EEM cross-validation, evaluation
exemption logic, a User Access/Role-management screen with no clear domain owner). The
companion validation spreadsheet is a different matter: its "REP BRs" sheet reproduces
~100 rows of a legacy MORE system's byte/field-position validation rules verbatim (e.g.
`[Field 7] Social Security Number`, keyed to a fixed-width record layout that has no
meaning in MiEdWorkforce's own data model) — a boundary violation at a larger scale than
prior findings of this kind. Most consequentially for the specific cross-domain question
this review was asked to answer: **`credentialing-domain.md`'s "Mentor Assignment
Requirement for Permits" business rule and `credentialing-sequences.md`'s "Submit Permit
Application" sequence both assume Staffing API integration points that `staffing-api.yml`
does not actually expose** — see the Cross-Domain Question section below.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | `Staffing Submission Validations.xlsx`, "REP BRs" sheet (105 rows, `F02_01`-`F26_6`) | A legacy system's byte/field-position validation catalog — messages literally reference `[Field 2]`, `[Field 7]`, `[Field 12]`, etc., tied to a fixed-width record layout from the predecessor MORE system, each flagged Fatal Error/Error/Warning and cross-referenced to a "MORE?" column | This is a wire-format/legacy-system rule crosswalk, not a functional description of MiEdWorkforce's own validation behavior — the same category as the CEPAS Posting File IDD's byte-position field table (flagged Boundary Violation in the FDD 07 review), but larger in scale (105 rows vs. one field table) and mixed directly into what otherwise reads as a business-rule spreadsheet rather than segregated into its own interface-design document. | Do not port field-position framing into the SDD. The underlying business intent (many of these rows) is legitimate and already substantially captured in `staffing-domain.md`'s Business Rules (e.g. credential/endorsement matching, employment date co-requirements) or newly flagged in Coverage Gaps below; the legacy field-numbering itself has no place in either the domain model or a functional spec. See `staffing-technical-design.md` (newly drafted this review) — "Validation Rule Engine" section — for where this content's *technical* residue (whether a 1:1 rule port is needed) now lives as an Open Technical Question. |
| 2 | `Staffing Submission Validations.xlsx`, "ID Validations" sheet (SSN structure rules: area/group/serial number ranges, specific known-invalid SSNs `078-05-1120`/`721-07-4426`/`219-09-9999`, DOB format variants Mi-Key accepts) | Field-level identity-validation specification that duplicates what should be Mi-Key's (the `identity` domain's) own internal validation logic | `staffing-domain.md`'s own Scope section already correctly excludes this ("Unique ID creation and identity matching logic > owned by `identity` (Mi-Key external service)") — this sheet's SSN/DOB rule detail is Mi-Key's implementation detail leaking into a Staffing-domain FDD artifact. | No `staffing-domain.md` action needed — correctly out of scope already. If an `identity`/Mi-Key FDD or capability doc is reviewed later, flag this sheet as a possible source for validating Mi-Key's own rule catalog. |
| 3 | Main FDD, Feature 10.2.3 / Business Specifications ("auto-generate data definition documents... in a user consumable format (Excel, pdf, etc.)... version controls with a summary of changes") | Describes a document-generation feature (auto-generated data dictionaries from CEDS metadata) at a reasonable business-requirement altitude, but implies a specific generation/export pipeline | Borderline, low severity — this is the kind of content `staffing-technical-design.md` (once populated) would elaborate on (format, storage, versioning mechanics), not a business-model gap. | No SDD action needed now; note for whoever builds this that the pipeline mechanics (similar to `credentialing-technical-design.md`'s "Certificate PDF Generation") belong in `staffing-technical-design.md`, not `staffing-domain.md`. |
| 4 | Main FDD, Feature 10.14 (Queue Management) | Explicitly reclassifies file-queue reset/delete/toggle-availability as a **non-admin, developer-only** technical task — the FDD pushing its *own* content below the functional-design altitude | This is the opposite direction from a typical altitude finding (the FDD is under-claiming scope rather than over-prescribing), but it's worth recording here because it directly determines whether `staffing.admin.view-file-queues`/`staffing.admin.reset-file-processing` (both of which already exist in `staffing-permissions.md`) should exist as *admin-facing* permissions at all. See Discrepancy #2 below — this is an FDD-vs-itself contradiction, not a clean altitude call. | See Discrepancy #2 for resolution — flagged here because the Addendum's reasoning ("technical management and maintenance tasks to be handled by developers") is itself an altitude-style argument the client is making about their own content. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | **SDD vs. itself** | Mentor-credential verification endpoint (Staffing ↔ Credentialing) | `credentialing-domain.md`'s "Mentor Assignment Requirement for Permits" business rule: "Call Staffing API: `GET /educators/{uniqueId}` — Verify educator exists and holds valid, relevant credential." | `staffing-api.yml` has no `GET /educators/{uniqueId}` endpoint. Its closest endpoint, `GET /educators/search` (`operationId: searchEducators`), is a paginated name/Unique-ID search returning `EducatorSearchResult`, whose only credential-related field is a free-text, nullable `activeCredentialSummary: "Brief description of active credentials (for mentor validation)"` — not a structured pass/fail against a specific credential type. Separately, `staffing-domain.md`'s own Dependencies table has the relationship running the *other* direction: **Staffing** calls **Credentialing** (`GET /educators/{educatorId}/credentials`) to validate placement — credential data is credentialing-owned, not staffing-owned. | Real, blocking mismatch — not just a naming drift like the payments `PaymentReceived`/`PaymentCompleted` case, but a question of which domain should even be answering this call, since staffing doesn't own credential validity in the first place. See the full answer under "Cross-Domain Question" below. | **Not fixed unilaterally** (this is a design decision, not a naming typo). Annotated in place: `credentialing-domain.md`'s rule now flags the call `**(unconfirmed — see below)**` with a full explanation, and a new Open Technical Question #8 was added to `credentialing-technical-design.md` alongside Discrepancy #2 below. |
| 2 | **SDD vs. itself** | Employment-assignment verification endpoint (Staffing ↔ Credentialing) | `credentialing-sequences.md`'s "Submit Permit Application" sequence: `AppService->>Staffing: Verify employment assignment (if required)` / `Staffing-->>AppService: Assignment verified`. | No endpoint in `staffing-api.yml` performs this check. The closest candidate, `GET /employee-roster/search`, is entity-scoped (requires `X-Organization-Context` and `staffing.employee-roster.view`), designed for a district user browsing their own roster — not a system-to-system "does Unique ID X have an active employment record at Org Y" lookup credentialing could call generically. No FDD source document reviewed so far (this one included) actually describes this check happening during permit submission — it appears to be an SDD-side assumption with no confirmed FDD origin, the same pattern already seen with the "Issue Temporary Permit — Exceptional Cases (Admin)" sequence. | Genuine gap, same shape as `credentialing-technical-design.md`'s existing Open Technical Question #7 (payment-status endpoint) — a real integration point assumed by one domain's sequence diagram with no confirmed shape (or even confirmed necessity) on the other domain's side. | Annotated in `credentialing-sequences.md` inline (`[endpoint unconfirmed - see credentialing-technical-design.md Open Technical Question #8]`) rather than inventing an endpoint. Folded into the same new Open Technical Question #8 as Discrepancy #1, since both are Credentialing→Staffing integration points surfaced by this same review. |
| 3 | **FDD vs. itself** | Is file-queue management (reset/delete/toggle upload availability) an admin-facing feature? | Main FDD Business Specifications ("the system will have an administrative page for the Staffing Data Admin to manage system configurations, for processing uploaded files and data sets... Reset/reprocess a file in the queue... Delete a file(s) in the queue... turn on/off upload availability") and High Level Scenario #5 (same content, framed as a numbered use case the Staffing Data Admin performs). | Addendum, Feature 10.14: "The requirements for the Staffing Admin to manage file queues (e.g., resetting 'stuck' files) and toggle upload availability should be considered **outside scope of an administrative task**. These functions are considered as technical management and maintenance tasks to be handled by developers." | The Addendum is later in the document and reads as the client's final, considered disposition (it explicitly walks back the earlier framing with reasoning), consistent with how Addendum sections elsewhere in this FDD (10.4-10.7) similarly narrow earlier Business-Specification language to "configuration only, logic lives elsewhere." Treat the Addendum as authoritative over the earlier narrative. | **Resolved by team default, not a client question** — treat file-queue reset/delete/toggle as an internal/developer operational capability, not an admin-facing UI feature. `staffing-permissions.md` already has `staffing.admin.view-file-queues` and `staffing.admin.reset-file-processing` as System-wide Admin permissions; recommend a follow-up note (not made here, since it's a judgment call about whether to remove vs. keep gated behind a more restrictive audience) confirming whether these should stay admin-facing-but-rarely-used or be demoted entirely to an ops-only tool outside the permission model. Not promoted to `client-questions.md` — resolvable with the FDD's own stated final position, doesn't block other work. |
| 4 | FDD vs. SDD (terminology, positive match) | Role name: "Staffing Data Administrator" | Main FDD uses "Staffing Data Administrator" (and "Staffing Administrator") consistently throughout Business Specifications and every High Level Scenario. | `staffing-domain.md`'s Ubiquitous Language table defines "**Staffing Data Administrator**" verbatim, and `staffing-permissions.md`'s Admin category notes are all attributed to the "Staffing Data Admin role." | Exact match — no drift, unlike the "Credential Admin" vs. "System Administrator – Credentialing" inconsistency found in the FDD 05 review. | Aligned — no action needed. Noted here as a positive data point since terminology drift has been a recurring finding in other domains. |
| 5 | FDD vs. SDD | Validation severity model | Main FDD Feature 10.3/10.13: three severity levels — Errors, Warnings, Data Quality. | `staffing-domain.md`'s `QUALITY_REVIEW_RESULTS` ERD entity: `error_count`, `warning_count`, `justification_required_count` with three corresponding payload arrays; `staffing-sequences.md`'s "Certify Collection" sequence treats errors as blocking and justification-required conditions as requiring text entry before certification. | Consistent — "Data Quality" severity in the FDD maps to the SDD's "justification required" category (several FDD Data-Quality-flagged rows, e.g. "apparent overreporting of single Employment Status code," read exactly like conditions needing a justification rather than a hard block). | Aligned — no action needed; worth noting the mapping explicitly if a future document introduces a fourth severity tier. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | `Staffing Submission Validations.xlsx`, Sheet1 rows under "1.0 Employment/Entity" (e.g. "If building has CTE, then CTE assignments should be reported," "...Special Ed...," "...Migrant Programs...," "...Pre-K...," "...Bilingual...," "All buildings providing instruction to students must report a Principal," "All districts... must report a Superintendent," "All schools must report Teachers") | A cross-validation business rule linking EEM-sourced building/district program flags to staffing-data completeness expectations — none of `staffing-domain.md`'s existing Business Rules reference EEM program flags (the closest, "Position Grade Span Must Align with Building Authorization," checks grade span, not program type) | `staffing-domain.md` — new Business Rule (e.g. "Program-to-Staffing Cross-Validation"), enforced by `Collection` during quality review, requiring an EEM program-flag lookup | M | Straightforward to draft directly once the exact EEM program-flag field names are confirmed (not fully enumerated in this FDD — the spreadsheet's own row literally asks "What other staffing groups could be validated against? Are there EEM fields that can help us determine if we can expect groups to be reported?", i.e. even the FDD's author flags this as unresolved) |
| 2 | `Staffing Submission Validations.xlsx`, Sheet1 "6.0 Evaluation" rows ("Evaluation exemptions based on previous eval outcomes (educator must have the applicable qualifying eval outcomes in order to be exempt)") | `staffing-domain.md`'s `EvaluationOutcome` has an appeal-window business rule but no exemption logic — no invariant or rule describes when an evaluation submission can be skipped based on a prior outcome | `staffing-domain.md` — extend `EvaluationOutcome`/`EmployeeRoster` Business Rules with an "Evaluation Exemption" rule | L | Underspecified in the FDD itself ("applicable qualifying eval outcomes" not enumerated) — worth a light client question only if a follow-up document doesn't clarify; not drafted yet given the vagueness |
| 3 | Main FDD, Feature 10.1 (User Access and Role Management: add/edit "Staffing Users" with Role Group/Role Type/Department/Status fields, dropdown values "to be defined") | `staffing-domain.md`'s Scope explicitly defers "User authorization and role assignments" to `iam`, and `staffing-permissions.md` has no per-domain user-role-assignment screen — this FDD describes a Staffing-scoped variant of user/role administration (with a "Department" field not present in `iam-domain.md`'s `RoleDefinition`) that doesn't clearly map to either domain's existing admin surface | `iam-permissions.md` / `iam-domain.md` (most likely) — or a thin Staffing-scoped view over IAM's existing Role+Scope model, pending confirmation that "Department"/"Group" are Staffing-specific concepts rather than generic IAM ones | M | Flag for whoever reviews FDD 01 follow-up work or does a dedicated cross-domain admin-UI pass; this FDD alone doesn't resolve whether "Department" is a new IAM concept or Staffing-specific metadata |
| 4 | `Staffing Submission Validations.xlsx`, "ID Validations" sheet, bottom rows ("Validations on data elements that have changed with 'too much' frequency... last name change more than 2 times in a year... When elements change with 'too much' frequency, user should receive a warning. Updating certain elements may trigger justification required, and/or admin approval.") | `staffing-sequences.md`'s "Update Employee Demographics" sequence models a simple Mi-Key sync with no frequency-monitoring or conditional-approval logic — no threshold, no admin-approval branch | `staffing-domain.md`/`staffing-sequences.md` — extend demographic-update handling with a change-frequency check and admin-approval branch (or confirm this belongs to `identity`/Mi-Key instead, since demographic data ownership sits there) | L | Underspecified ("too much" frequency undefined) — not worth drafting until thresholds are confirmed; note if a Mi-Key/identity FDD surfaces this same requirement independently |
| 5 | Main FDD, Feature 10.12 (Identity Management processing — admin manages identity-processing authorized roles, business rule validations, reports, alerts/communications, and file/API queues, in addition to the ID Resolution worklist) | `staffing-domain.md`/`staffing-sequences.md` already model the narrowest piece (Identity Admin approves/denies new-ID requests via `ID_RESOLUTION_REQUESTS` and the "Near Match Resolution" sequence), but the FDD's broader ask — a dedicated admin config surface for identity-specific validations, reports, and alerts, separate from the general Staffing admin surfaces — isn't distinguished anywhere in the SDD from the general-purpose `staffing.admin.manage-validations`/`manage-reports` permissions | Likely no new SDD action — probably the general-purpose admin permissions already cover this and the FDD is just calling out identity as one category among several a Staffing Data Admin configures, not a functionally distinct capability | L | Not drafted — flagged only in case a future identity/Mi-Key-focused FDD implies otherwise |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs, Feature 10.9-10.11 (Add/modify Collection: Position Roster, Employee Roster, Employee Assignment Details, entity types, open/close dates, category requirements) | staffing | `Collection`/`CollectionDefinition`; Sequence: Open New Collection; `staffing.admin.manage-collections` | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Feature 10.13, Business Specs (assign business rule/validation to Collection/Category/Data Element, severity, justification) | staffing | `QualityReviewResult`; `staffing.admin.manage-validations` | implements (business-rule assignment/config); see Altitude #1 for the underlying rule catalog itself |
| `10 - Data Collection & Compliance Staffing Admin` §Feature 10.8 (Data Element Definition Administration) | staffing | `staffing.admin.manage-data-elements` | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (data Category definitions, entity-type association) | staffing | `staffing.admin.manage-data-elements`; `CategoryRequirement` (Collection aggregate) | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (collection exception date ranges at entity/organization level) | staffing | `CollectionException`; Sequences: Request Collection Exception, Staffing Admin Approve Exception; `staffing.admin.process-exceptions` | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs, Feature 10.15 (data closeout processing, immediate/scheduled to CEPI Data Warehouse) | staffing, reporting | `staffing-technical-design.md` — "Collection-to-Warehouse Batch Processing" | implements (elaborated in new technical-design doc; see Altitude #3) |
| `10 - Data Collection & Compliance Staffing Admin` §Feature 10.14 (Queue Management) | staffing | `staffing.admin.view-file-queues`, `staffing.admin.reset-file-processing` | conflicts with itself (see Discrepancy #3) |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (Alerts & Notifications management, deferred to 6.0) | staffing, communications (unreviewed) | (deferred per FDD's own Dependencies) | informs — out of scope per FDD's own disposition |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (Email Template management, deferred to 6.0) | staffing, communications (unreviewed) | (deferred per FDD's own Dependencies) | informs — out of scope per FDD's own disposition |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (Dashboard config, deferred to 3.0) | staffing, dashboards (unreviewed) | (deferred per FDD's own Dependencies) | informs — out of scope per FDD's own disposition |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (Reports config, deferred to 2.0) | staffing, reporting | `staffing.admin.manage-reports` | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs ("Employment Data Collection Worklist" by entity type/collection) | staffing | `Collection` pre-certification tasks (implied); no explicit `Worklist` aggregate | implements (conceptually) — informal match, not a gap |
| `10 - Data Collection & Compliance Staffing Admin` §Business Specs (Data Quality Review & Processing admin config, DQ Dashboard reports) | staffing | `QualityReviewResult`; Sequence: Run Quality Review | implements |
| `10 - Data Collection & Compliance Staffing Admin` §Feature 10.12 (Identity Management processing admin) | staffing | `ID_RESOLUTION_REQUESTS`; Sequence: Near Match Resolution (admin approve/deny branch) | implements (narrow); gap — broader admin config surface not distinguished (see Coverage Gap #5) |
| `10 - Data Collection & Compliance Staffing Admin` §Feature 10.1 (User Access and Role Management for Staffing users) | staffing, iam (unreviewed for this specific screen) | (no Staffing-scoped user/role admin screen modeled) | gap — not yet modeled (see Coverage Gap #3) |
| `Staffing Submission Validations.xlsx` §Sheet1 (credential/endorsement matching for Administrator/Instructional and Teacher assignments) | staffing, credentialing | "Credential Matching for Administrator/Instructional Positions" and "Endorsement Alignment for Teaching Assignments" business rules; Sequence: Assign Employee to Position | implements — already well modeled |
| `Staffing Submission Validations.xlsx` §Sheet1 (PPR flag restricts employment) | staffing, profpractice | Roster Eligibility Assessment integration; Sequence: Add New Employee | implements — already well modeled |
| `Staffing Submission Validations.xlsx` §Sheet1 (program-to-staffing EEM cross-validation rows) | staffing, organizations | (no equivalent business rule) | gap — not yet modeled (see Coverage Gap #1) |
| `Staffing Submission Validations.xlsx` §Sheet1 (evaluation exemption logic) | staffing | `EvaluationOutcome` | gap — not yet modeled, underspecified (see Coverage Gap #2) |
| `Staffing Submission Validations.xlsx` §Sheet1 ("Warning/flag for employees reported across multiple districts") | staffing | `staffing-domain.md` Open Question #1 (multi-district employment) | informs — corroborates existing open question |
| `Staffing Submission Validations.xlsx` §"REP BRs" sheet (legacy MORE field-position rule catalog) | staffing | `staffing-technical-design.md` — "Validation Rule Engine" | informs — technical detail correctly excluded from domain model (see Altitude #1) |
| `Staffing Submission Validations.xlsx` §"ID Validations" sheet (SSN/DOB field-level rules) | staffing, identity (unreviewed) | (deferred per `staffing-domain.md` Scope to `identity`/Mi-Key) | informs — out of scope per SDD's own disposition (see Altitude #2) |
| **Cross-domain: `credentialing-domain.md`'s "Mentor Assignment Requirement for Permits"** | credentialing, staffing | `GET /educators/{uniqueId}` (assumed, not implemented); `staffing-api.yml`'s actual `GET /educators/search` / `EducatorSearchResult.activeCredentialSummary` | conflicts with (see Discrepancy #1); open — `credentialing-technical-design.md` Open Technical Question #8 |
| **Cross-domain: `credentialing-sequences.md`'s "Submit Permit Application" "Verify employment assignment" call** | credentialing, staffing | No corresponding `staffing-api.yml` endpoint | gap — not yet modeled; open — `credentialing-technical-design.md` Open Technical Question #8 (see Discrepancy #2) |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | What are the exact EEM program/facility flags (CTE, Special Education, Migrant Education, Pre-K, Bilingual, Title programs, etc.) available for the program-to-staffing cross-validation rules in Coverage Gap #1? The FDD's own source spreadsheet asks this same question of itself ("What other staffing groups could be validated against? Are there EEM fields that can help us determine if we can expect groups to be reported?") and doesn't answer it. | Internal (organizations/EEM domain owner) | Open |
| 2 | Is "Department" (Feature 10.1's Add User form) a Staffing-specific concept, or should it be a generic field on `iam-domain.md`'s user/role model? Dropdown values are explicitly "to be defined" in the FDD itself. | Internal | Open |
| 3 | What quantifies "too much" frequency for demographic-change monitoring (ID Validations sheet), and does admin approval gate the change itself or just trigger a warning? | Internal (or Client, if not resolved by the time this becomes buildable) | Open |

Note: none of these three clear the `client-questions.md` judiciousness bar. #1 and #2
are internal cross-domain modeling questions the team can investigate against the EEM/IAM
data model directly rather than asking the client to re-explain their own spreadsheet;
#3 is a minor, non-blocking parameter that doesn't stall other work. The two
cross-domain Credentialing↔Staffing endpoint mismatches (mentor verification, employment
assignment verification) are **not** listed here — they're SDD-vs-itself issues the
engineering team can resolve without the client, tracked instead as
`credentialing-technical-design.md` Open Technical Question #8, consistent with how the
FDD 07 review handled the equivalent `PaymentCompleted`/payment-status-endpoint mismatch.

## Related Documents

- Source document(s): `functional-design-docs/10 - Data Collection & Compliance - Staffing Admin/10 - Data Collection & Compliance Staffing Admin.md`; `functional-design-docs/10 - Data Collection & Compliance - Staffing Admin/Staffing Submission Validations.md`
- SDD documents reviewed against: `solution-areas/staffing/staffing-domain.md`, `solution-areas/staffing/staffing-permissions.md`, `solution-areas/staffing/staffing-sequences.md`, `solution-areas/staffing/staffing-api.yml`, `solution-areas/staffing/staffing-technical-design.md` (newly drafted this review); cross-domain excerpts of `solution-areas/credentialing/credentialing-domain.md` ("Mentor Assignment Requirement for Permits"), `solution-areas/credentialing/credentialing-sequences.md` ("Submit Permit Application"), `solution-areas/credentialing/credentialing-technical-design.md` (Open Technical Questions); light checks of `solution-areas/profpractice/profpractice-domain.md` (Roster Eligibility Assessment Rule — confirmed aligned, no new findings) and `solution-areas/organizations/organizations-api.yml` (`OrganizationType` enum — no Staffing Agency references in this FDD, so no new cross-check needed beyond the existing FDD-17 open question)

---

## Cross-Domain Question: Does `staffing-domain.md`/`staffing-api.yml` expose the endpoint credentialing's mentor/permit rules assume? (asked directly of this review)

**Does staffing expose a `GET /educators/{uniqueId}`-shaped endpoint?** No. `staffing-api.yml`
has exactly one educator-lookup endpoint, `GET /educators/search` (`operationId:
searchEducators`) — a paginated, query-parameter search (`name` or `uniqueId`, both
optional, returning an array with pagination metadata), not a single-resource
path-parameterized GET. It requires `staffing.employee-roster.view` and is explicitly
scoped as a "system-wide identity lookup" with `x-scope-sensitive: false`.

**What does it return, and is it the same thing credentialing needs?** It returns
`EducatorSearchResult`: `uniqueId`, `fullName`, `dateOfBirth`, and a nullable
`activeCredentialSummary` (a free-text string, explicitly documented as existing "for
mentor validation"). This confirms staffing's own API design *anticipated* being used for
mentor validation — but the field it offers is a human-readable summary string, not a
structured "does this Unique ID hold credential type X" boolean/enum credentialing's rule
logic ("Verify educator exists and holds valid, relevant credential... If invalid, return
error") could evaluate programmatically without parsing free text.

**Are "verify mentor holds a valid credential" and "verify employment assignment" the same
capability, or two different ones?** Two different ones, and neither is currently backed by
a real staffing endpoint:
- **Mentor credential verification** (`credentialing-domain.md`'s "Mentor Assignment
  Requirement for Permits") is fundamentally a **credential-validity** question. Credential
  data is credentialing-owned — `staffing-domain.md`'s own Dependencies table has this
  relationship running the *opposite* direction (Staffing calls Credentialing's `GET
  /educators/{educatorId}/credentials` to validate placement, not the reverse). Routing this
  check through Staffing at all may be unnecessary — Credentialing likely should just query
  its own domain data directly for "does Unique ID X hold a valid [permit-relevant]
  credential," with no cross-domain call needed. `staffing-api.yml`'s `activeCredentialSummary`
  field would then be a UI-only convenience (for a human reviewing a search result), not the
  system-of-record answer for an automated eligibility check.
- **Employment assignment verification** (`credentialing-sequences.md`'s "Submit Permit
  Application" — `Staffing: Verify employment assignment (if required)`) is a genuinely
  different question: "is this Unique ID currently employed at this organization?" — which
  *is* staffing-owned data (`EmployeeRoster`/`EMPLOYEE_ROSTER` in `staffing-domain.md`), but
  has no corresponding endpoint today. The closest candidate, `GET /employee-roster/search`,
  is entity-scoped and permission-gated for a district user's own roster browsing, not built
  as a generic cross-domain verification call, and — notably — **no FDD source document
  reviewed across this entire series actually describes this check happening during permit
  submission**, so its necessity itself is unconfirmed, not just its endpoint shape.

Both points are now tracked as `credentialing-technical-design.md` Open Technical Question
#8, and the corresponding lines in `credentialing-domain.md` and `credentialing-sequences.md`
have been annotated in place (not silently left pointing at a nonexistent endpoint, and not
guessed at unilaterally) — the same treatment given to the `PaymentCompleted`/payment-status
mismatch in the FDD 07 review.

## Process/Template Friction Noted

- This FDD (10) turned out to be almost entirely about the **admin/configuration** side of
  Staffing data collection — it barely touches the actual district-submitter workflow
  (adding employees, correcting validation errors, certifying collections), which lives
  instead in `staffing-sequences.md` alone with no FDD source reviewed so far. This mirrors
  the FDD-05 (System Admin - Credentialing) pattern exactly: an admin-facing FDD numbered
  alongside, but structurally separate from, the domain's day-to-day submission FDD(s). If
  a "Staffing" (non-admin) FDD numbered similarly to FDD 16 (relative to FDD 11) exists in
  a later drop, it would be the more direct source for validating `staffing-sequences.md`'s
  "Add New Employee"/"Assign Employee to Position"/"Certify Collection" flows against an
  FDD, rather than this review having done so opportunistically via the validation
  spreadsheet.
- As with the FDD 07 review, the most consequential finding here (the Credentialing↔
  Staffing endpoint mismatches) came from cross-referencing two SDD documents against each
  other, not from anything this FDD directly contradicts — this FDD's own content review
  is closer to "Gaps Identified" territory. Worth reiterating as a standing process note
  now that most core domains have full SDDs: FDD reviews increasingly do their most
  valuable work on cross-domain integration points that have nothing to do with the FDD
  nominally under review.
