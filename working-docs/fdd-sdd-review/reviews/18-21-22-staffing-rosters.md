# Review: Data Collection & Compliance - Roster of Positions, Roster of Employees, Employee Assignment Details (18, 21, 22)

| Field | Value |
|---|---|
| Source Document(s) | `18 - Data Collection & Compliance - Roster of Positions/18 - Collection & Compliance - Position Roster.docx` (reviewed via `.md`)<br>`18 - Data Collection & Compliance - Roster of Positions/StaffPositionsDetails.xlsx` (reviewed via `.md`)<br>`21 - Data Collection & Compliance - Roster of Employees/21 - Collection & Compliance - Employee Roster.docx` (reviewed via `.md`)<br>`22 - Data Collection & Compliance - Employee Assignment Details/22 - Collection & Compliance - Employee Assignment Details.docx` (reviewed via `.md`)<br>`22 - Data Collection & Compliance - Employee Assignment Details/21.7; 22.6 - Collection & Compliance - Employee and Appropriate Placement Audit.docx` (reviewed via `.md`)<br>Out of scope, not converted, noted only: `22 - .../CIP_Program_Endorsements_for_CTE_Instruction_v5_730456_7 (1).pdf`; `22 - .../CTE CIP and Cluster as of October 22 2025.pdf`; `22 - .../Teacher Credential Verification Report_09-12-2025_02-16-31-444.xls` (previously failed conversion, not retried) |
| Related Domain(s) | staffing (primary); credentialing (endorsement/credential mapping, cross-domain) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-14 |
| **Overall Status (per document)** | `18 - Collection & Compliance - Position Roster.docx`: **Aligned — Minor SDD Revisions Needed**. `StaffPositionsDetails.xlsx`: **Aligned** (reference taxonomy only, no discrepancies). `21 - Collection & Compliance - Employee Roster.docx`: **Gaps Identified**. `22 - Collection & Compliance - Employee Assignment Details.docx`: **Gaps Identified**. `21.7; 22.6 - Employee and Appropriate Placement Audit.docx`: **Aligned — Minor SDD Revisions Needed**. |

## Summary

These five documents are exactly the day-to-day district data-submission FDD set the FDD 10
review identified as missing: FDD 18 (positions), FDD 21 (employees), and FDD 22 (assignment
details) describe individual districts entering, bulk-uploading, and API-submitting their own
roster data — a different scope than FDD 10's admin/configuration-only content. The core
domain model holds up very well: `staffing-domain.md`'s `PositionRoster`, `EmployeeRoster`,
`Assignment`, `Collection`, and `AuditWorkItem` aggregates, and their business rules for
credential/endorsement matching, PPR restriction, and audit-worklist routing, all match this
FDD set closely and in some cases almost verbatim. The most consequential findings are: (1)
a well-specified school-year rollover/carry-forward process (FDD 18 §18.3, FDD 21 §21.3) that
`staffing-domain.md` referenced (`Collection` aggregate's "Referenced In: Sequence: Open New
Collection") but never actually modeled — drafted directly as a new sequence and business
rule; (2) a fully-described cross-entity employee "push/share" workflow (FDD 21 §21.4.5,
Feature 21.1.1) with no home anywhere in the SDD; (3) an Education History sub-model
(source tracking, CEPI/STARR-NSC read-only provenance, required-for-position-type business
rule) that was previously a single under-specified value object — extended directly; and (4)
confirmation that this FDD set's "New Teacher Mentor" verification (a name lookup) is
unrelated to the still-open Credentialing↔Staffing mentor-*credential*-verification gap
flagged in the FDD 10 review — the two are different concepts entirely, addressed directly in
the domain model's field description. The Appropriate Placement Audit doc (21.7/22.6)
confirms the `AuditWorkItem` model closely but, per the specific question this review was
asked to check, does **not** provide enough detail to draft the credentialing-side
SCED/Grade/CIP-to-endorsement mapping catalog FDD 12's review recommended — it only consumes
that mapping (deferred explicitly to FDD 12.1 in its own Dependencies section), it doesn't
define it.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | FDD 18, Addendum Features 18.1-18.2 (exact modal step sequence, stat-card labels, "red state indicator" for FTE mismatch, specific button/link text like "See Recommendations") | Screen-by-screen wireframe detail: modal steps, specific UI components (stat cards), exact banner/button text | Same low-severity pattern as prior reviews' wireframe-column findings (FDD 05, 11, 13, 29) — field/screen-inventory detail, not new business logic once abstracted to the underlying `Assignment`/`PositionRoster` model. | No SDD action needed; underlying business rules already captured in `staffing-domain.md`. |
| 2 | FDD 21, Addendum 21.1.2 (exact tab list: "Demographic Information / Credentials / Employment / Professional Development / Education"; exact field list for demographic form) | Exhaustive UI field/tab inventory for the employee detail page | Same pattern as #1 — legitimate underlying data (identity, credentials, employment, PL, education) is already modeled across `staffing-domain.md` and cross-domain APIs; the specific tab layout is presentation detail. | No SDD action needed. |
| 3 | FDD 22, Business Specifications ("college/university field will be defined by the Staffing Data Administrator (EEM IHE and NSC as base?)") | A specific data-source implementation question posed as a requirement | Borderline — reads as the FDD author's own open technical question rather than a settled requirement (note the "(EEM IHE and NSC as base?)" phrasing). Not prescriptive enough to flag further; the underlying need (a controlled institution list) is a legitimate business requirement regardless of source. | No SDD action needed; if a future EEM/reference-data FDD clarifies the institution list source, note it in `staffing-domain.md`'s `EducationHistory` extension made in this review. |
| 4 | FDD 22, Business Specifications ("The Education History component data will calculate and store as the 'Highest Level of Education Completed' within the K12Staff CEDS Domain") | Names the specific CEDS domain/element the calculated value must be stored against | Mild — CEDS alignment is a stated project-wide constraint (see FDD 18/21/22's shared "Assumptions" section), not a novel implementation choice specific to this feature, so this is closer to a legitimate business/compliance requirement than an implementation prescription. | No SDD action needed; captured as a computed property in the `EducationHistory` extension made in this review (see Coverage Gap #3 / direct fix). |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | SDD vs. itself — **fixed directly (2026-08-14)** | Missing "Open New Collection" sequence | `staffing-domain.md`'s `Collection` aggregate lists "Referenced In: Sequence: Open New Collection." | `staffing-sequences.md` (prior to this review) contained no such sequence — it jumped straight from certification/audit flows to position creation, with no rollover/initialization flow at all. | Genuine internal SDD gap: a cross-reference pointing at a sequence that didn't exist. FDD 18 §18.3 (Feature 18.3, stories 18.3.1-18.3.10) and FDD 21 §21.3 (Feature 21.3, stories 21.3.1-21.3.4) both describe the rollover logic in unambiguous, matching detail (carry forward records without an end date; exclude records with both an end date and separation/cancellation reason; recalculate data-quality indicators; log outcome). | **Fixed directly.** Added "Open New Collection" sequence to `staffing-sequences.md` and a new "Collection Rollover Carries Forward Active Records" business rule to `staffing-domain.md`, drafted from FDD 18 §18.3 and FDD 21 §21.3 (which agree with each other in full). |
| 2 | FDD vs. SDD — **fixed directly (2026-08-14)** | Tri-state vs. boolean placement fitness | FDD 18 §18.1.6/18.1.7: assigned employees display a "Meets Requirements" indicator of "Yes," "Partially," or "No" — with a distinct recommendation banner and "Appropriate Placement Recommendations" modal specifically for "Partially" (e.g., holds the credential but a different endorsement). | `staffing-api.yml`'s `PlacementValidationResult.meetsRequirements` was a plain boolean — no way to express "partially" (only a `validationMessages` array with `Error`/`Warning` severities). | Real, well-evidenced expressiveness gap in the API contract, unambiguous to fix — the FDD is specific and internally consistent about the three-state UI. | **Fixed directly.** Added `meetsRequirementsLevel: enum [Yes, Partially, No]` to `PlacementValidationResult` in `staffing-api.yml`, keeping the existing boolean for backward-compatible checks. |
| 3 | FDD vs. SDD — clarifying (not a conflict) | Is FDD 21/22's "New Teacher Mentor" verification the same as Credentialing's permit-mentor credential check flagged missing in the FDD 10 review? | FDD 22 §"New Teacher"/Addendum 22.1.4: "The user will be able to enter a Unique ID, the system will return the name associated with the Unique ID for verification prior to submission." No credential check is described anywhere in FDD 21 or 22 for this mentor. | `staffing-domain.md`'s `NEW_TEACHER_MONITORING.mentor_verified` field previously read "True if mentor Unique ID validated via Credentialing API," implying a credential-validity check. `credentialing-domain.md`'s "Mentor Assignment Requirement for Permits" rule (a *different* mentor concept, for EDSP/EEDSP/SAP/SSWP permit applicants) is the one with the actual unresolved `GET /educators/{uniqueId}` cross-domain gap from the FDD 10 review. | The two "mentor" concepts are unrelated: this FDD's New Teacher Mentor (MCL 380.1526, tracked in `NewTeacherMonitoring`) is a Staffing-owned, first-three-years mentorship requirement verified by a simple name lookup (most plausibly `staffing-api.yml`'s own `GET /educators/search`, which already returns `fullName` for a given `uniqueId`) — not a credential check, and not staffing calling credentialing. The FDD 10-flagged gap (Credentialing calling Staffing to verify a *permit* mentor's credential) remains open and untouched by this FDD set. | **Fixed directly, narrowly.** `staffing-domain.md`'s `mentor_verified` field description corrected to describe a name-lookup only, and explicitly cross-referenced to the unrelated, still-open permit-mentor gap so the two are never conflated again. No change made to `credentialing-domain.md` — that gap is unaffected by this review's source material (see Priority Question #2 below). |
| 4 | FDD vs. itself — minor, not resolved | Editable position fields after creation | FDD 18 main Business Specifications: "The system will allow changes to the positions in the org chart... a MiEdWorkforce user with the School District role can manage the position data by: Add / Remove / **Rename** / Assign Employee." | FDD 18 Addendum 18.2.1 ("Edit Position Details"): "the system displays editable fields for **Approved FTE, Building and Status**" — no mention of title/rename. | Same pattern noted in the 10, 13 reviews: an Addendum section describing a narrower field set than the main narrative. `staffing-api.yml`'s `UpdatePositionRequest` already includes `title`, `localJobCategory`, `localJobFunction`, and `approvedFte` — consistent with the main narrative's "Rename" capability, not the Addendum's narrower list. | Not resolved — low stakes, doesn't block anything since the SDD's existing (broader) `UpdatePositionRequest` already covers both readings. Recorded as an internal Open Question rather than a client question, consistent with how the same Addendum-narrows-main-narrative pattern was handled in the FDD 10 and FDD 13 reviews. |
| 5 | FDD vs. SDD — corroborates, no change needed | Credential/endorsement matching at assignment time | FDD 22 Business Specifications and Addendum 22.1.1-22.1.3: Administrator/Instructional positions require a valid credential; Teacher positions with course details require a valid endorsement covering SCED code and grade level; if missing, user is directed to Temporary Credential Application or must provide a required, free-form Credential Error Justification, which is included in Appropriate Placement Audit reviews and not viewable to the citizen. | `staffing-domain.md`'s "Credential Matching for Administrator/Instructional Positions" and "Endorsement Alignment for Teaching Assignments" business rules; `staffing-sequences.md`'s "Assign Employee to Position" sequence (justification path creates assignment with status `Justified`, alerts SOM Auditor). | Near-exact match, already flagged as well-modeled in the FDD 10 review's tagging table — this review independently confirms it from the actual submission-side FDD. | Aligned — no action needed. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | FDD 21, Business Specifications and Addendum 21.1.1/21.4.5: "the system will allow a user to 'send' or 'push' submitted employee data to entities within the established hierarchy, including PSA Education Management Companies, PSA Charter Authorizer and ISD/RESA hierarchy. The receiving entity will receive an Action Item in the worklist to confirm the employee sent... Upon confirmation, the employee is successfully added to the Employee Roster with an indicator that the data has been received from another entity." | A cross-entity employee-sharing workflow (e.g., a shared speech therapist or daily substitute reported once and pushed to multiple district rosters) with no aggregate, event, permission, or endpoint anywhere in `staffing-domain.md`/`staffing-permissions.md`/`staffing-api.yml` today. | `staffing-domain.md` — likely a new capability on `EmployeeRoster` (e.g. a `RosterShareRequest` value object/sub-entity) plus a new event pair (e.g. `EmployeeRosterShareRequested`/`EmployeeRosterShareConfirmed`) and worklist integration; `staffing-permissions.md` needs a new permission (push vs. confirm are different actors). | H | Not drafted directly — genuinely new scope (new permission model for the pushing vs. receiving entity, and interaction with the still-thin `worklists` platform capability) rather than an unambiguous extension. Flagged for whoever next does a dedicated pass on `staffing-domain.md` or `worklists`. |
| 2 | FDD 21/22 (both, repeated verbatim across `21.4.1`, `21.7.2`, `22.6.2`): District/ISD/SOM audit report sections "Required Positions with No Assignments" (positions with no staff) and "Buildings with No Appropriate Positions" (EEM-open buildings with no assigned positions/staff) | `staffing-api.yml`'s `DistrictAuditReport.sections` previously had only `employeesNotAppropriated`, `employeesNoAssignment`, and `changesSummary` — missing these two report sections, named identically across three separate source documents (strong, repeated evidence). Notably, "Buildings with No Appropriate Positions" is the same EEM open-building-no-staff concept as the FDD 10 review's Coverage Gap #1 (program-to-staffing EEM cross-validation), now confirmed as a concrete report requirement, not just a validation rule. | `staffing-api.yml` — `DistrictAuditReport.sections` | H | **Fixed directly.** Added `requiredPositionsNoAssignments`, `buildingsNoAppropriatePositions`, and a separated `changesToPositions` section (distinct from the existing employee-roster-focused `changesSummary`) to `DistrictAuditReport` in `staffing-api.yml`, citing the FDD sections verbatim. |
| 3 | FDD 22, Business Specifications ("Education History... If no historical data exists... the system will allow the School District (and individual) to perform updates... The Degree element will be required, all other elements will be optionally submitted... The School District and citizen user(s) are not able to edit or remove Education History that is sourced from the CEPI Data Warehouse... Cannot remove Education History data submitted by the citizen [as a district user]... Can view all updates made by a citizen.") | `staffing-domain.md`'s `EducationHistory` value object was a flat degree/institution/year/field-of-study record with no provenance tracking (CEPI Data Warehouse/STARR-NSC vs. district-entered vs. citizen-entered), no read-only-by-source flag, and no "who can remove what" rule. | `staffing-domain.md` — `EducationHistory`/`EDUCATION_HISTORY` | M | **Fixed directly.** Extended the `EDUCATION_HISTORY` ERD entity with `source`, `education_verification_method`, `is_read_only`, `added_by`, and `added_by_role` fields, and added a required-for-position-type business rule and a Key Invariant note, all cited to FDD 22. |
| 4 | FDD 21 §"Employee Collection Cert Level" validations (Business Specifications table) and FDD 21.4.3: warning conditions — "Warning/flag for employees reported across multiple districts," "No new employees added since last 30-day certification," "No terminations reported since last 30-day certification," "Apparent overreporting of single Employment Status code," "All terminated records reported with same Date of Termination," "Summary patterns where the same Employment Separation Reason is reported for all separating employees" | `staffing-domain.md`'s `QualityReviewResult` models the three-tier severity shape (errors/warnings/justification-required) correctly at the aggregate level, but none of these specific collection-level warning conditions are individually named anywhere in the domain model or `staffing-technical-design.md`'s "Validation Rule Engine" categorization. | `staffing-technical-design.md` — "Validation Rule Engine," under the existing "Historical/trend anomaly detection" and "Cross-record / collection-level" categories (Open Technical Questions #2/#3 already cover this shape generically) | L | Not drafted — these are concrete instances of the same open technical questions already tracked in `staffing-technical-design.md` from the FDD 10 review; no new open question needed, just additional evidence for the existing ones. |
| 5 | FDD 21.7/22.6, Business Specifications: three district-side acknowledgment actions at assignment time when a placement is inappropriate ("Acknowledge placement as inappropriate" / "...and wants to correct" / "Update assignment to place an employee appropriately"), each explicitly assigning a task to the **ISD Auditor's** worklist and sending an ISD Auditor alert immediately at assignment-save time (not waiting for collection certification). | `staffing-sequences.md`'s "Assign Employee to Position" sequence only models the SOM Auditor alert on justification submission (matching FDD 22 Addendum 22.1.1 exactly); `AuditWorkItem`/`staffing-domain.md`'s "ISD Auditor Worklist Population After District Certification" rule creates ISD audit items only in bulk, after full collection certification — not per-assignment, in real time, as this document describes. | `staffing-domain.md`/`staffing-sequences.md` — extend `AuditWorkItem` creation to also fire per-assignment (not just per-certification) when a district acknowledges an inappropriate placement, in addition to the existing SOM alert | M | Not drafted — this is a genuine timing/granularity gap (real-time per-assignment ISD worklist items vs. today's single certification-triggered item) rather than a simple field addition; recommend for whoever next revises the "Assign Employee to Position" sequence or the Audit aggregate. |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `18 - Collection & Compliance - Position Roster` §Business Specs (org chart, Add/Remove/Rename/Assign Employee, position statuses) | staffing | `PositionRoster` aggregate; Sequences: Create New Position, Update Position Status | implements |
| `18 - ...` §Addendum 18.1 (Add New Position, spot workflow, FTE health indicator) | staffing | `PositionRoster`/`Assignment` aggregates; Sequence: Assign Employee to Position | implements |
| `18 - ...` §Addendum 18.2 (Edit/Remove Position) | staffing | `PositionRoster`; `staffing.position-roster.update`/`.delete` | implements; minor Addendum-vs-main discrepancy noted (Discrepancy #4) |
| `18 - ...` §Addendum 18.3 (Create New Collection / rollover) | staffing | `Collection` aggregate; new Sequence: Open New Collection; new Rule: Collection Rollover Carries Forward Active Records | implements — drafted directly this review (Discrepancy #1) |
| `18 - ...` §Addendum 18.4 (Position Roster Certification) | staffing | `Collection` aggregate; Sequence: Certify Collection; `staffing.position-roster.certify` | implements |
| `18 - ...` §Addendum 18.5 (Data Input via API and File Upload) | staffing | `staffing.position-roster.create`/`.update`; bulk/API ingestion (no dedicated sequence beyond `staffing-technical-design.md`) | implements |
| `18 - ...` §Feature 18.6 (Email/Comm) | staffing, communications | Deferred to Business Specification #6 per FDD's own disposition | informs — out of scope for staffing, consistent with FDD's own text |
| `StaffPositionsDetails.xlsx` (Education Job Type/Local Job Category/Local Job Function reference taxonomy) | staffing | `PositionRoster.EducationJobType`/`LocalJobCategory`/`LocalJobFunction` value objects | implements — reference data source, no discrepancies |
| `21 - Collection & Compliance - Employee Roster` §Business Specs (employment data elements, statuses, validations) | staffing | `EmployeeRoster` aggregate; Sequence: Add New Employee, Update Employee Demographics | implements |
| `21 - ...` §PPR validation table and Addendum 21.1.4 (Reviewed-Listed hard block, Reviewed-Felony halt, Under Review/Document Hold/PPR Hold alerts) | staffing, profpractice | Roster Eligibility Assessment integration (`GET /educators/{id}/roster-eligibility`); Sequence: Add New Employee | implements — richer detail than currently modeled (informs, not a gap requiring action; SDD's boolean eligibility check is a reasonable simplification of the FDD's graduated messaging) |
| `21 - ...` §Addendum 21.3 (Create a New Collection / rollover) | staffing | `Collection`; new Sequence: Open New Collection | implements — drafted directly this review (Discrepancy #1) |
| `21 - ...` §Addendum 21.4 (Employee Roster Certification, warning conditions) | staffing | `Collection`; `QualityReviewResult`; Sequence: Certify Collection | implements; specific warning conditions inform `staffing-technical-design.md` Open Technical Questions #2/#3 (Coverage Gap #4) |
| `21 - ...` §Addendum 21.4.5 (Push/send employee data to hierarchy entities) | staffing | (no equivalent aggregate; the receiving entity's "worklist" Action Item is staffing's own filtered list endpoint per the Worklist/Pending-Items working assumption, not a separate platform capability — see solution-architecture.md) | gap — not yet modeled (Coverage Gap #1) |
| `21 - ...` §Feature 21.5 (Data Input via API and File Upload) | staffing | `staffing.employee-roster.bulk-upload`; bulk/API ingestion | implements |
| `21 - ...` §Feature 21.6 (Email/Comm) | staffing, communications | Deferred to Business Specification #6 per FDD's own disposition | informs — out of scope for staffing |
| `22 - Collection & Compliance - Employee Assignment Details` §Business Specs (Assignment Details, credential/endorsement validation, Education History) | staffing | `Assignment` aggregate; `EducationHistory` value object (extended); Sequences: Assign Employee to Position, Report Course Details | implements; `EducationHistory` extended directly this review (Coverage Gap #3) |
| `22 - ...` §Business Specs (New Teacher Mentor lookup) | staffing | `NewTeacherMonitoring.mentor_verified` | implements — field description corrected this review (Discrepancy #3) |
| `22 - ...` §Business Specs (Evaluation Outcomes, citizen/district appeal) | staffing | `EvaluationOutcome` value object; "Evaluation Outcome Appeal Window" business rule | implements — exact match, including the 5-year window and citizen-any-time appeal asymmetry |
| `22 - ...` §Addendum 22.1.3 (Course-level data elements: SCED, grade span, CTE Career Cluster/CIP, Special Ed age group/service category, specialized funding) | staffing | `CourseDetail`/`SpecializedFunding` value objects; `ASSIGNMENT_COURSE_DETAILS` ERD entity | implements |
| `22 - ...` §Addendum 22.2/22.3 (Non-Instructional/Student Support Staff assignment details) | staffing | `Assignment`/`AssignmentDetail` | implements |
| `21.7; 22.6 - Employee and Appropriate Placement Audit` §Business Specs (ISD Auditor worklist, Audit Finding form, document requests, SOM Auditor oversight, finalize/de-certify) | staffing | `AuditWorkItem` aggregate; Sequence: ISD Auditor Review District Submission; `staffing.audit.*` permissions | implements — near-exact match, `AuditType` enum (`SchoolSafety`/`AppropriatePlacementVerification`) matches FDD's two audit types precisely |
| `21.7; 22.6 - ...` §Dependencies ("mappings of credentials to assignments... defined by the MiEdWorkforce System Admin within Managing Credential Use of Assignment Codes (12.1)") | staffing, credentialing | (no aggregate — deferred to FDD 12.1, not defined here) | informs — does **not** provide sufficient detail to draft the credentialing-side mapping aggregate FDD 12's review recommended (see Priority Question #1 below); confirms only that this FDD *consumes*, not defines, that mapping |
| `21.7; 22.6 - ...` §Addendum 21.7/22.6 (real-time ISD Auditor alert on district acknowledgment of inappropriate placement) | staffing | `AuditWorkItem` | gap — not yet modeled at this granularity (Coverage Gap #5) |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Should `UpdatePositionRequest` continue to allow renaming a position's title/job category/function after creation (per FDD 18's main narrative), or should the Addendum's narrower "Approved FTE, Building, Status only" reading govern (Discrepancy #4)? | Internal | Open |
| 2 | Should `AuditWorkItem` creation happen per-assignment in real time (as FDD 21.7/22.6's three acknowledgment paths describe, each routed to the ISD Auditor) in addition to the existing per-certification bulk creation, or is the FDD describing the same underlying event at a different level of narrative detail (Coverage Gap #5)? | Internal | Open |
| 3 | What EEM data source populates `buildingsNoAppropriatePositions` (FDD's "Buildings with No Appropriate Positions" report section) — the same EEM program/facility flags already flagged as unresolved in the FDD 10 review's Open Question #1? | Internal (organizations/EEM domain owner) | Open |

Note: none of these three clear the `client-questions.md` judiciousness bar. #1 and #2 are
internal modeling/granularity questions resolvable by the team without the client (the SDD
already has a safe, working default in both cases — the broader field set, and
certification-time audit item creation — that doesn't block other work while unconfirmed).
#3 is the same underlying EEM data-source question already tracked as an internal open
question in the FDD 10 review, not a new one requiring the client.

## Related Documents

- Source document(s): `functional-design-docs/18 - Data Collection & Compliance - Roster of Positions/18 - Collection & Compliance - Position Roster.md`; `functional-design-docs/18 - Data Collection & Compliance - Roster of Positions/StaffPositionsDetails.md`; `functional-design-docs/21 - Data Collection & Compliance - Roster of Employees/21 - Collection & Compliance - Employee Roster.md`; `functional-design-docs/22 - Data Collection & Compliance - Employee Assignment Details/22 - Collection & Compliance - Employee Assignment Details.md`; `functional-design-docs/22 - Data Collection & Compliance - Employee Assignment Details/21.7; 22.6 - Collection & Compliance - Employee and Appropriate Placement Audit.md`
- SDD documents reviewed against: `solution-areas/staffing/staffing-domain.md` (updated), `solution-areas/staffing/staffing-permissions.md`, `solution-areas/staffing/staffing-sequences.md` (updated), `solution-areas/staffing/staffing-technical-design.md`, `solution-areas/staffing/staffing-api.yml` (updated); cross-domain excerpts of `solution-areas/credentialing/credentialing-domain.md` ("Mentor Assignment Requirement for Permits" rule, endorsement mapping scope)
- Related prior reviews: [10-staffing-admin](10-staffing-admin.md) (established the `staffing` domain baseline, the Credentialing↔Staffing mentor/employment-verification endpoint gaps, and the EEM program-flag cross-validation open question this review's Coverage Gap #3/Open Question #3 corroborate), [12-system-admin-users](12-system-admin-users.md) (source of the recommended `credentialing-domain.md` endorsement-mapping aggregate this review's Priority Question #1 addresses)

---

## Answers to This Review's Priority Questions

**1. Does this FDD set describe the SCED/Grade/CIP-to-endorsement mapping and "appropriate
placement" alignment-audit process in enough detail to draft the `credentialing-domain.md`
aggregate FDD 12's review recommended?** **No.** Both FDD 22's main narrative ("The
appropriate credential to position mappings are defined by the MiEdWorkforce Admin (12.1)";
"The appropriate endorsements to assignment mappings are defined by the MiEdWorkforce Admin
(12.1)") and the Appropriate Placement Audit doc's own Dependencies section ("The
Appropriate Placement Audit will utilize the mappings of credentials to assignments as
defined by the MiEdWorkforce System Admin within Managing Credential Use of Assignment Codes
(12.1)") consistently and explicitly defer the mapping's *definition* to FDD 12.1 — they only
describe *consuming* it (at assignment-time validation, and during audit review). This
FDD set does confirm two things worth recording: the mapping catalog is genuinely
cross-domain (used by Staffing's `Assignment` validation and by the Staffing-owned
`AuditWorkItem`/audit-finding process, not just a Credentialing-internal concern), and the
"appropriate placement" audit workflow itself (worklist, alerts, acknowledgment paths,
finding forms) is already well-modeled in `staffing-domain.md`'s `AuditWorkItem` aggregate —
it's specifically the mapping catalog itself (SCED/Grade-Setting/CIP → Endorsement crosswalk)
that remains undrafted in `credentialing-domain.md`, exactly as the FDD 12 review left it.
No new draft was attempted here; drafting it still requires FDD 12.1's own content.

**2. Does it describe or use the mentor-verification or employment-assignment-verification
endpoints already flagged as missing in the FDD 10 review, and does it add clarity on what
these should look like?** **No, on both counts, but it does resolve an adjacent point of
confusion.** FDD 21/22's "New Teacher Mentor" (MCL 380.1526, tracked in
`NewTeacherMonitoring`) is a different concept from Credentialing's permit-mentor
requirement (EDSP/EEDSP/SAP/SSWP, `credentialing-domain.md`'s "Mentor Assignment Requirement
for Permits" rule) — this FDD's mentor verification is a simple Unique-ID-to-name lookup with
no credential check described anywhere ("the system will return the name associated with the
Unique ID for verification prior to submission"), most plausibly served by staffing's own
`GET /educators/search` endpoint. It does not touch, and provides no clarity on, the two
still-unresolved cross-domain endpoints from the FDD 10 review (`GET /educators/{uniqueId}`
for mentor credential validity, and the employment-assignment-verification check assumed by
`credentialing-sequences.md`'s "Submit Permit Application" sequence). Fixed directly:
`staffing-domain.md`'s `mentor_verified` field description previously implied a credential
check via Credentialing API — corrected to describe the name-lookup-only mechanism this FDD
actually documents, and cross-referenced to the unrelated, still-open gap so the two are not
conflated in future reviews.

**3. Is this the "day-to-day staffing data submission" FDD the FDD 10 review said was
missing?** **Yes, unambiguously.** FDD 18, 21, and 22 describe exactly the individual-district
online-entry, bulk-file, and API submission workflows for positions, employees, and
assignment details (add/edit/remove records, run quality review, resolve errors, certify
collections) that FDD 10 review's "Process/Template Friction Noted" section predicted would
exist in "a later drop" — the review explicitly named `staffing-sequences.md`'s "Add New
Employee"/"Assign Employee to Position"/"Certify Collection" flows as needing exactly this
kind of FDD to validate against, which this review has now done directly, confirming a close
match with only the gaps and one internal fix (the rollover sequence) noted above.

## Process/Template Friction Noted

- This is the first review in the series with **five source documents spanning three FDD
  folders** reviewed together as one closely-related set (per the template's allowance for
  "a main FDD plus its interface-design sub-docs and supporting spreadsheets" — extended here
  to three sibling folders describing one continuous workflow: position → employee →
  assignment → audit). The single-review-file-covers-multiple-tracker-rows mechanic worked
  cleanly; worth confirming this is the intended scaling pattern as more multi-folder FDD sets
  are picked up (e.g., a future combined review of FDD 19/20, which are themselves two
  side-by-side Data Quality Review folders).
- Two of this review's three fixes (the missing "Open New Collection" sequence, and the
  `PlacementValidationResult` tri-state gap) were found by cross-referencing the FDD against
  the SDD's own internal "Referenced In" pointers and UI-description language, respectively —
  not by reading the FDD in isolation. This continues the pattern noted in the FDD 10 review:
  once a domain has a mature SDD, the highest-value findings increasingly come from checking
  whether the SDD's own cross-references and UI-facing contracts actually hold up against a
  freshly-read FDD, rather than from spotting net-new business rules.
- The three out-of-scope, unconverted files in FDD 22's folder (two CIP/CTE crosswalk PDFs and
  the still-unparseable Teacher Credential Verification `.xls`) are exactly the kind of source
  material FDD 12.1's endorsement-mapping catalog would need once that gap is picked up — worth
  flagging for whoever does that work that these files exist and may be directly relevant, even
  though they weren't converted or read for this review.
