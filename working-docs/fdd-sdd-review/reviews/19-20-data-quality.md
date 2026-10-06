# Review: Data Quality Review & Processing; Data Quality Review & Processing - Comments & Flags (19, 20)

| Field | Value |
|---|---|
| Source Document(s) | `19 - Data Quality Review & Processing/19 - Data Quality Review & Processing.docx` (reviewed via `.md`)<br>`19 - Data Quality Review & Processing/Copy of EOY 2024 REP Comparison and Review - Final.xlsx` (reviewed via `.md`)<br>`19 - Data Quality Review & Processing/Workgroup Materials/REP_DQ Log.xlsx` (reviewed via `.md`)<br>`20 - Data Quality Review & Processing - Comments & Flags/20.0 - Data Quality Review & Processing - Comments and Flags.docx` (reviewed via `.md`)<br>Out of scope, not converted, noted only: `19 - .../Workgroup Materials/4.23.25 MiEdWorkforce DQ Dashboard Meeting Notes.pdf`; `4.24.25 ... Meeting Notes.pdf`; `4.29.25 ... Meeting Notes.pdf`; `MiEd DQ Dashboard Regroup_1.pptx` |
| Related Domain(s) | staffing (primary); communications, credentialing, profpractice, epp, proflearning, iam (cross-domain, for FDD 20's system-wide comment/flag framework) — note: "worklists" is no longer tracked as a separate platform-capability domain (its former `solution-areas/worklists/` folder was removed; see `solution-architecture.md`'s Worklist/Pending-Items working assumption) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-14 |
| **Overall Status (per document)** | `19 - Data Quality Review & Processing.docx`: **Aligned — Minor SDD Revisions Needed**. `Copy of EOY 2024 REP Comparison and Review - Final.xlsx`: **Aligned** (corroborating evidence only, no new gaps). `Workgroup Materials/REP_DQ Log.xlsx`: **Aligned** (corroborating evidence only, no new gaps). `20.0 - Comments and Flags.docx`: **Gaps Identified** (the "Flag" half of the concept; the "Comment" half is **Aligned**). The four out-of-scope PDFs/pptx remain `Not Started`. |

## Summary

FDD 19 describes a Data Quality (DQ) Administrator role that defines, schedules, and runs
data-validation checks against submitted staffing data, with results surfaced via a
district-facing "Data Quality Dashboard," Action Items on a worklist, and email
notifications. This maps closely onto `staffing-domain.md`'s existing `Collection`/
`QualityReviewResult` model and `staffing-technical-design.md`'s "Validation Rule Engine"
section — FDD 19 is best read as corroborating and enriching that already-modeled concept
(admin configuration of checks, categories, aggregation levels, thresholds), not as
evidence for a separate "Data Quality" domain concept. The two companion spreadsheets (EOY
REP Comparison, REP_DQ Log) reinforce this: they are concrete instances of the
"Historical/trend anomaly detection" and "Legacy REP field-position rules" categories
`staffing-technical-design.md` already carries as open technical questions, not a new
category of business rule. FDD 20 is a separate, system-wide "Comments & Flags" framework
spanning nearly every domain (Credentialing, PPR, EPP, Professional Learning, RapBacks,
Identity, and Data Quality) — its Comment half is already well-modeled by
`communications-capability.md`'s `Comment` aggregate (free-form text, internal/external
visibility by functional area, `PredefinedComment` templates) and resolves the same way the
FDD 05/06 "Internal Comment Group" question did (a visibility/permission attribute, not a
governance layer). Its Flag half — a status marker that creates an "Action Item" routed to
a specific worklist — has no dedicated SDD concept anywhere yet, though the pieces exist
adjacent to it: `staffing-domain.md`'s per-condition `QualityReviewResult`/justification
model, and staffing's own `worklist.staffing-dataquality.view`/`.process` permissions
(previously cited from a now-removed `solution-areas/worklists/worklists-permissions.md`
doc — under the Worklist/Pending-Items working assumption in `solution-architecture.md`,
these permissions belong in `staffing-permissions.md` as staffing's own scoped permissions
gating its own filtered list endpoint, not a separate cross-domain namespace).

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | FDD 19 Business Specifications / Feature 19.2.1 ("administrative page to define and manage the data quality processing... sortable table... Data Quality ID, DQ Category, Title, Description, Status") | Screen-by-screen admin-page field inventory | Same low-severity wireframe-detail pattern noted in prior reviews (FDD 05, 10, 11, 13, 18) — legitimate underlying configuration data (`staffing.admin.manage-validations`) already exists; the specific table layout is presentation detail. | No SDD action needed. |
| 2 | FDD 19 Business Specifications ("Authorized users will have access to the specifications (backend code) used to generate the DQ data results.") | Exposes literal backend implementation code (not just results/rationale) to end users as a stated business requirement | This reads as an implementation artifact (query/rule source) being treated as a user-facing deliverable, which is unusual for a functional spec — most likely the client means "the rule's logic/criteria in plain language" (which FDD 19 elsewhere calls "Problem, Impact, Resolution" text, already modeled), not literal source code. | Flag back to client for confirmation of intent (see Open Questions below) rather than modeling literal code-exposure as a requirement. |
| 3 | `Staffing Submission Validations.xlsx`'s "REP BRs" sheet content, now corroborated at much greater scale by `REP_DQ Log.md`'s 200+ `DQCCYY##`-coded rules (e.g., `DQRP1001`-`DQRP1151`+), most keyed to legacy REP fixed-width field positions (e.g., "Field 10: School Assignment Data," "Field 25: Employment Status") | A full legacy validation-rule catalog presented at the level of specific field positions in a predecessor system's file format | Already correctly identified as implementation/configuration detail, not domain business rules, in `staffing-technical-design.md`'s "Validation Rule Engine" section (drafted from the FDD 10 review) — this review's new evidence (REP_DQ Log) simply confirms the scale (200+ rules, not "~100+" as FDD 10's spreadsheet alone suggested) and reinforces that a 1:1 port of legacy field-position rules is the wrong altitude for `staffing-domain.md`. | No new SDD action; strengthens the existing Open Technical Question #1 in `staffing-technical-design.md` (see Coverage Gap #3 below). |
| 4 | `Copy of EOY 2024 REP Comparison and Review - Final.md`'s "Compare - 6yr Single"/"Compare - 3yr All" tabs (hard-coded thresholds: "Absolute Percent Change over 75%... over 33% compared to the same collection in the prior year") | Specific numeric thresholds embedded in an internal CEPI analyst spreadsheet, not a stated business rule in either FDD | This is CEPI's own internal working tool, not a functional design document proper — it answers (in part) `staffing-technical-design.md`'s Open Technical Question #3 ("What baseline and threshold apply to the trend/anomaly-detection style Data Quality checks?") with real numbers, but as found-in-the-wild evidence rather than a specified requirement. Treat as informing, not adopting verbatim, since the FDD's own DQ Administrator role is explicitly described as being able to *set* these thresholds. | No SDD change; recorded as corroborating evidence for the existing Open Technical Question rather than a new hard-coded rule. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | FDD vs. SDD — clarifying, not a conflict | Is "Data Quality" its own domain concept, or fully covered by `Collection`/`QualityReviewResult`? | FDD 19 describes a dedicated "Data Quality Administrator" role, a DQ-specific admin page (DQ ID, Category, Title, Description, Status, effective dates, associated data elements, "DQ specifications," report settings, DQ text, justification expectations, versioning/audit), scheduling/pause options, and a district-facing "Data Quality Dashboard" — presented with enough structure that it could be read as implying a standalone `DataQualityCheck` aggregate. | `staffing-domain.md`'s `Collection` aggregate already owns `QualityReviewResult` (errors/warnings/justification-required, three severity tiers) and `staffing-permissions.md` already has `staffing.admin.manage-validations` (assign validation to collection/category/element, set severity) and `staffing.collection.validate`. `staffing-technical-design.md`'s "Validation Rule Engine" section already anticipates exactly FDD 19's admin configuration surface (Category, Title/Description via message text, Status, effective dates, aggregation level) as the *content* of the rule catalog, not a new aggregate. | FDD 19's DQ Administrator role and admin page are a richer configuration UI for the same underlying `Collection`/`QualityReviewResult`/Validation Rule Engine concept already modeled — not a new domain concept. The one piece not yet named in the SDD is a standalone `DataQualityCheck` definition entity (ID, category, title, description, status, effective dates, schedule/interval, pause/sleep options) as the configuration object the Validation Rule Engine executes against — this is a real but narrow gap (see Coverage Gap #1), not evidence that "Data Quality" needs to be pulled back out as a first-class domain the way the corpus's archived "v1-dataquality" domain apparently was. | **No separate domain needed.** Recorded as Coverage Gap #1 (a configuration-entity gap inside the existing Validation Rule Engine, not a new aggregate or domain) — see Priority Question #1 answer at the end of this review for the full reasoning. |
| 2 | FDD vs. SDD — resolves the same way as FDD 05/06 | Does FDD 20's "Comments & Flags" framework need a new "Comment Group" or category-governance layer? | FDD 20 Business Specifications: "MiEdWorkforce System Administrators, by functional area, can mark comments for the respective functional area for internal or external view... Comments will be restricted by user role/authorization to specified functional area... default view... will vary, in each functional area, for each comment box." | `communications-capability.md`'s `Comment` aggregate already models exactly this shape: `visibility: Internal\|External` per comment, comments "associated with a process item (credential application, staffing record, etc.)," and `PredefinedComment` "Template comments available per functional area." This is the identical resolution already reached for the FDD 05/06 "Internal Comment Group" question (`client-questions.md` #2 background; `credentialing-domain.md`'s former Open Question #11, resolved per the FDD 06 review as "atomic permission + category label, not a group abstraction"). | Same shape of resolution as the prior review — no new group/governance concept needed. FDD 20 does not introduce anything the `Comment` aggregate + functional-area-scoped visibility can't already express; it is a second, independent corroboration of the same design decision across a different, wider set of functional areas (this time explicitly naming Credentialing, PPR, EPP, Professional Learning, RapBacks, Identity, and Data Quality as consumers of one shared comment framework, rather than just Credentialing). | **Confirmed, no new work.** `communications-capability.md`'s existing `Comment.visibility` + functional-area scoping already covers FDD 20's Comment half. No SDD change needed; this corroborates rather than revises the FDD 05/06 resolution. |
| 3 | FDD vs. SDD — genuine gap, not a conflict | FDD 20's "Flag" concept (a status marker distinct from a Comment, which creates an "Action Item" for "a specific user, role, or worklist") | FDD 20 Business Specifications: "The MiEdWorkforce System Administrators, in each functional area, will have the ability to flag comments to create 'Action Item' for a specific user, role, or worklist. The Action Item will alert the recipient or worklist." Story 20.0.4: applying "a sensitive flag (such as a Professional Practice Review flag)" to a record is a distinct action from adding the internal remark itself ("Applying an internal flag will also allow the option to add an internal remark..."). | `communications-capability.md`'s `Comment` aggregate has no `Flag` concept — visibility (internal/external) is a property of a Comment, not a separate status marker with its own lifecycle, and there's no cross-domain "Action Item" abstraction in `communications-capability.md`. FDD 19's own "Action Item" (assigned to submitting organizations on DQ violation) is a related but domain-specific instance already loosely covered by `staffing-domain.md`'s per-condition justification model and staffing's own `worklist.staffing-dataquality.*` permissions — but FDD 20 describes this as a *general, cross-domain* mechanism ("in each functional area"), which is broader than staffing alone. | Genuine, narrow coverage gap: a generic Flag→Action Item→routing mechanism, separate from Comment, that multiple domains would use (PPR flags, Credentialing status-change flags requiring mandatory comments, Data Quality Action Items). **This is worth re-checking against the Worklist/Pending-Items working assumption's own revisit triggers** (`solution-architecture.md`) — routing an Action Item to "a specific user, role, **or worklist**" (i.e., naming an arbitrary target, not just filtering a domain's own status field) looks like the "routing finer than a domain's own filters" case the working assumption says should be flagged rather than built around silently. This is arguably a `communications` extension (a `Flag`/`ActionItem` concept with routing) rather than a `worklists` platform capability, since there is no longer a separate `worklists` domain to own it. | Not drafted directly — this is new, cross-domain scope requiring a design decision about ownership (`communications` vs. per-domain), not an unambiguous single-file fix. Flagged as Coverage Gap #2. |
| 4 | FDD vs. itself — minor, not resolved | FDD 20's "mandatory comment on certain flags" pairing | Story 20.0.3: applying "Hold," "Cancel," or "Deny" requires a mandatory **external** comment, defaulted to external visibility. | Story 20.0.4: applying a sensitive/internal flag (e.g., PPR) triggers an *optional* internal remark ("Applying an internal flag will also allow the **option** to add an internal remark") — not mandatory. | Not a contradiction once read carefully (external-visible status changes require justification the applicant can see; internal-only flags merely allow optional context) — but the document doesn't state this distinction explicitly, so a future reader could easily misapply "mandatory comment" uniformly to both flag types. | Low stakes, internal-only. No client question needed; the distinction can be preserved directly wherever this gets modeled (Coverage Gap #2): external-status-change flags require a comment, internal-only flags make a comment optional. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | FDD 19 Business Specifications / Feature 19.2.1, 19.2.5 (DQ check definition: ID, category, title, description, status, effective dates, associated data elements, "specifications" i.e. logic/thresholds, justification expectations required/optional, aggregation level statewide/ISD/district/building/individual, versioning and change-audit notes) | A standalone `DataQualityCheck`/`DataQualityDefinition` configuration entity — the object the Validation Rule Engine executes against — is implied by `staffing-technical-design.md`'s existing "Admin Configuration Surface" paragraph and `staffing.admin.manage-validations` permission, but never modeled as an entity with its own fields, versioning, and lifecycle the way `CredentialDefinition`/`DefinitionVersion` are modeled in `credentialing-domain.md`. | `staffing-technical-design.md` — extend the existing "Validation Rule Engine" section with a `DataQualityCheckDefinition` entity (fields per FDD 19 §19.2.5) and versioning/audit note, parallel to credentialing's `CredentialDefinition`/`DefinitionVersion` pattern | M | Not drafted directly — this is squarely a technical-design elaboration (config schema for an already-acknowledged engine), reasonable for whoever next revises `staffing-technical-design.md`, but it's a judgment call whether it belongs there or promoted into `staffing-domain.md` as a full aggregate; left as a recommendation rather than an unambiguous fix. |
| 2 | FDD 20 Business Specifications and Stories 20.0.3/20.0.4 (Flag → Action Item → worklist/user/role routing, distinct from Comment) | No `Flag` concept, and no generic cross-domain "Action Item" routing mechanism, exists in `communications-capability.md` or elsewhere. `staffing-domain.md`'s per-condition justification-required flow and staffing's own `worklist.staffing-dataquality.*` permissions cover a staffing-specific instance of this pattern but not the general, cross-functional-area mechanism FDD 20 describes. | Likely `communications-capability.md` (a new `Flag` aggregate or extension of `Comment` with a `triggersActionItem` relationship, surfaced via each consuming domain's own filtered list endpoint per the Worklist/Pending-Items working assumption) — genuinely ambiguous whether `communications` or each domain individually should own it, now that there is no separate `worklists` platform capability to fall back on as a third option | H | Not drafted — this is new, multi-domain scope (Credentialing status-change flags, PPR flags, Data Quality Action Items all funneling through one mechanism) that needs an ownership decision before drafting; see Discrepancy #3 above. Does not currently block other work since each domain's existing narrower mechanism (e.g., `staffing-domain.md`'s justification-required conditions, `credentialing-domain.md`'s status change with `is_internal` note) still functions without it. |
| 3 | `REP_DQ Log.md`'s "DQ Log" sheet — 200+ concrete rule instances (e.g., `DQRP1092` "Buildings Without Staff," `DQRP1097` "Open Schools Without Instructional Staff," `DQRP1146` "New Teacher Status Does Not Align with Date of Hire," `DQRP1108` "Re-Reported Terminated Staff") classified `Internal`/`External`/`Summary`/`Proposed to Retire`, each with a defined Problem/Impact/Resolution text triplet, `HighStakes` flag, and scheduling metadata (run per collection cycle, verification-day flag) | Concrete evidence for `staffing-technical-design.md`'s already-open Technical Questions #1 (legacy rule port vs. re-derivation) and #2 (cross-record/collection-level rules requiring EEM lookups — `DQRP1092`/`DQRP1097` are exactly the "Buildings with No Appropriate Positions"/"Buildings Without Staff" EEM cross-validation gap already identified and fixed in the FDD 18/21/22 review's `DistrictAuditReport` sections) and #3 (thresholds) | `staffing-technical-design.md` — Open Technical Questions #1, #2, #3 (no new question needed) | L | Not drafted — purely corroborating evidence at greater scale and specificity than FDD 10's spreadsheet alone provided. No new SDD action; strengthens existing open questions rather than resolving them (thresholds like 33%/75% come from CEPI's internal working tool, not a stated FDD requirement, so they're evidence, not a spec to adopt verbatim). |
| 4 | FDD 19 Feature 19.1.4/19.4 (Data Quality "Action Item" clears via correction-and-repass, justification submission, or district acknowledgment — three distinct clearing mechanisms) plus `Employment Data Collection Worklist` tracking overall district task progress | `staffing-domain.md`'s `QualityReviewResult` models the DQ violation shape correctly but doesn't explicitly name the Action-Item lifecycle (created → cleared via correction/justification) or a district-level "task/checklist" progress view spanning multiple collections | `staffing-domain.md` or `staffing-sequences.md` — could extend "Run Quality Review"/"Certify Collection" sequences with explicit Action Item lifecycle states | L | Not drafted — largely already implied by existing justification-required flow in `Certify Collection` sequence; the gap is naming/explicitness rather than missing behavior. Low priority since the underlying mechanics already work. |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `19 - Data Quality Review & Processing` §Business Specs, Feature 19.2.1-19.2.6 (DQ Administrator role, admin page, check definition attributes, versioning) | staffing | `Collection`/`QualityReviewResult`; `staffing.admin.manage-validations`; `staffing-technical-design.md` Validation Rule Engine | implements (concept); gap — configuration-entity detail not modeled (Coverage Gap #1) |
| `19 - Data Quality Review & Processing` §Feature 19.2.4, 19.2.5 (aggregation levels: statewide/ISD/district/building/individual; analysis referencing current, historical, and other-system data) | staffing | `staffing-technical-design.md` Open Technical Questions #2, #3 | informs — corroborates existing open questions |
| `19 - Data Quality Review & Processing` §Feature 19.1.1, 19.3 (Data Quality Dashboard link on District dashboard, dynamic reports, summary/frequency counts) | staffing, dashboards (unreviewed) | (deferred per FDD 19's own Dependencies to Dashboards (3)/Reporting (2)) | informs — out of scope per FDD's own disposition |
| `19 - Data Quality Review & Processing` §Feature 19.4.1-19.4.2 (Action Item assignment, email notification via DQ Category Email Templates) | staffing, communications | `EmailTemplate` (communications-capability.md); `worklist.staffing-dataquality.view`/`.process` (staffing's own permission, formerly cited from the now-removed `solution-areas/worklists/worklists-permissions.md`) | implements |
| `19 - Data Quality Review & Processing` §Business Specs ("Other contact types include EEM Contacts, and authorized users sourced from CEPI User Lookup Tool") | staffing, organizations (unreviewed) | (no equivalent contact-resolution mechanism) | gap — not yet modeled; low priority, not drafted (informational contact routing, not blocking) |
| `19 - Data Quality Review & Processing` §Feature 19.1.6, 19.1.9 (acknowledge/justify DQ violation, justification stored and accessible for historical review, publishable) | staffing | `Collection.QualityReviewResult`; `COLLECTION_JUSTIFICATIONS` ERD entity | implements |
| `19 - Data Quality Review & Processing` §Feature 19.1.8 (historical DQ checks, status/change details) | staffing | `Collection`/`COLLECTION_CERTIFICATION` audit trail | implements |
| `19 - Data Quality Review & Processing` §Business Specs (Employment Data Collection Worklist tracking task progress) | staffing | `worklist.staffing-dataquality.*` (staffing's own permission, per the Worklist/Pending-Items working assumption in `solution-architecture.md` — not a separate `worklists` domain) | implements — corroborates staffing's own worklist-style permission scoping (District/ISD transitive) |
| `Copy of EOY 2024 REP Comparison and Review - Final` (Compare - 6yr Single / 3yr All tabs, Highlight Count, Anomaly List) | staffing | `staffing-technical-design.md` "Historical/trend anomaly detection" category; Open Technical Question #3 | informs — concrete threshold evidence (33%/75%), not a modeled rule |
| `Workgroup Materials/REP_DQ Log` §DQ Log sheet (200+ `DQCCYY##` rules, Internal/External/Summary/Proposed to Retire classification) | staffing | `staffing-technical-design.md` "Legacy REP field-position rules (MORE-inherited)" category; Open Technical Question #1 | informs — corroborates at greater scale, no new modeling |
| `Workgroup Materials/REP_DQ Log` §DQRP1092/DQRP1097 ("Buildings Without Staff," "Open Schools Without Instructional Staff") | staffing, organizations | `DistrictAuditReport.buildingsNoAppropriatePositions` (staffing-api.yml, added in FDD 18/21/22 review) | implements — independent corroboration of an already-fixed gap |
| `20.0 - Comments and Flags` §Business Specs, Stories 20.0.1/20.0.2 (free-form comment box, internal/external marking by functional area) | communications, credentialing, profpractice, epp, proflearning | `Comment` aggregate, `Comment.visibility` (communications-capability.md) | implements — confirms existing model across a wider functional-area set |
| `20.0 - Comments and Flags` §Story 20.0.3 (mandatory external comment on Hold/Cancel/Deny status flags) | communications, credentialing | `Comment`; `APPLICATION_PROCESSING_NOTES.is_internal` (credentialing-domain.md) | implements — pattern match; comment-required-on-status-change not yet an explicit invariant |
| `20.0 - Comments and Flags` §Story 20.0.4 (internal flag + optional internal remark, PPR example) | communications, profpractice, staffing | (no `Flag` concept) | gap — not yet modeled (Coverage Gap #2) |
| `20.0 - Comments and Flags` §Dependencies (comments used across Professional Practices 9.0, Identity Management 15.0/24.0, Credentialing 16.0, Data Quality 19.0, Employee/Placement Audit 21.7/22.6, EPP 25.0, Professional Learning 26.0, RapBacks 30.0) | communications, profpractice, iam, credentialing, staffing, epp, proflearning, identity (unreviewed for RapBacks) | `Comment` aggregate (communications-capability.md) — cross-domain consumer list | implements — confirms `Comment` is intentionally domain-agnostic infrastructure, as already designed |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Does "Authorized users will have access to the specifications (backend code) used to generate the DQ data results" (FDD 19 Business Specifications) literally mean exposing rule source code to end users, or does it mean exposing the rule's plain-language logic/criteria (already captured via the Problem/Impact/Resolution text model)? | Internal (resolvable by re-reading in context; not blocking) | Open |
| 2 | Who should own the generic Flag → Action Item → routing mechanism FDD 20 describes as spanning every functional area — `communications` (as an extension of `Comment`), or should each domain continue modeling its own narrower instance (as `staffing-domain.md`'s justification-required flow and `credentialing-domain.md`'s status-change notes already do)? (There is no separate `worklists` platform capability to consider as a third option — see `solution-architecture.md`'s Worklist/Pending-Items working assumption.) | Internal | Open |

Note: neither question clears the `client-questions.md` judiciousness bar. #1 is answerable by asking the client informally or by defaulting to the plain-language reading (consistent with the rest of FDD 19's Problem/Impact/Resolution model) without blocking any build decision. #2 is an internal architecture/ownership call — the SDD already has working, if narrower, per-domain defaults that don't block current work while this is unresolved.

## Related Documents

- Source document(s): `functional-design-docs/19 - Data Quality Review & Processing/19 - Data Quality Review & Processing.md`; `functional-design-docs/19 - Data Quality Review & Processing/Copy of EOY 2024 REP Comparison and Review - Final.md`; `functional-design-docs/19 - Data Quality Review & Processing/Workgroup Materials/REP_DQ Log.md`; `functional-design-docs/20 - Data Quality Review & Processing - Comments & Flags/20.0 - Data Quality Review & Processing - Comments and Flags.md`
- SDD documents reviewed against: `solution-areas/staffing/staffing-domain.md`, `solution-areas/staffing/staffing-permissions.md`, `solution-areas/staffing/staffing-sequences.md`, `solution-areas/staffing/staffing-technical-design.md`, `solution-areas/staffing/staffing-api.yml` (skimmed); `solution-areas/communications/communications-capability.md` (`Comment` aggregate); cross-domain excerpt of `solution-areas/credentialing/credentialing-domain.md` (former Open Question #11, now resolved/removed). (The `solution-areas/worklists/worklists-permissions.md` doc cited by this review at the time has since been removed — "worklist" is now treated as a UI/API pattern owned by each domain, not a separate SDD doc; see `solution-architecture.md`'s Worklist/Pending-Items working assumption.)
- Related prior reviews: [10-staffing-admin](10-staffing-admin.md) (established the Validation Rule Engine categorization this review's Coverage Gap #3 corroborates), [18-21-22-staffing-rosters](18-21-22-staffing-rosters.md) (source of the `DistrictAuditReport.buildingsNoAppropriatePositions` fix this review's REP_DQ Log evidence independently corroborates), [06-alerts-emails-communications](06-alerts-emails-communications.md) (source of the FDD 05/06 "Internal Comment Group" resolution this review's Discrepancy #2 confirms applies unchanged to FDD 20's broader scope)

---

## Answers to This Review's Priority Questions

**1. Is "Data Quality" a genuinely distinct concern needing its own concept/aggregate, or
is it fully expressible as validation rules + worklist routing already covered by
staffing's existing model?** **The latter, with one narrow addition.** FDD 19's DQ
Administrator role, admin configuration page, and district-facing dashboard describe a
richer configuration and presentation layer than what's explicit in `staffing-domain.md`
today, but every piece of it maps onto already-modeled concepts: the `Collection`/
`QualityReviewResult` aggregate for the check-execution and results model,
`staffing.admin.manage-validations`/`manage-data-elements` for the admin configuration
permissions, `staffing-technical-design.md`'s Validation Rule Engine for the rule-catalog
concept (including its existing categorization of field-level, cross-field, cross-record,
and historical/trend rule shapes — which FDD 19's own scenarios and both spreadsheets match
category-for-category), and staffing's own `worklist.staffing-dataquality.*` permissions
(District/ISD-scoped, transitive) for the Action-Item routing. The one real gap is that the
Validation Rule Engine's "admin configuration surface" is described only in prose today
(`staffing-technical-design.md`) rather than as a named, versioned entity the way
`credentialing-domain.md`'s `CredentialDefinition`/`DefinitionVersion` is — FDD 19 §19.2.5's
attribute list (ID, category, title, description, status, effective dates, schedule,
pause/justification options) is a ready-made field list for that entity (Coverage Gap #1).
The corpus's archived "v1-dataquality" domain does **not** need to come back as a
first-class domain area: nothing in FDD 19, the EOY REP Comparison spreadsheet, or the
REP_DQ Log describes behavior staffing's existing submission/validation model can't already
express once the configuration-entity gap above is filled.

**2. Does FDD 20's "Comments & Flags" concept parallel or overlap with credentialing's
already-resolved Internal Comment Group question? Same shape, or genuinely different?**
**Same shape for Comments; genuinely new (not previously addressed) for Flags.** FDD 20's
Comment half — free-form text, internal/external visibility set per functional area by a
System Administrator, restricted by role/authorization — is exactly the pattern already
resolved via `communications-capability.md`'s `Comment.visibility` and functional-area
scoping, first established answering the FDD 05/06 "Internal Comment Group" question. FDD
20 doesn't raise anything new on the Comment side; it corroborates that resolution across a
much wider explicit list of consuming functional areas (Credentialing, PPR, EPP,
Professional Learning, RapBacks, Identity, Data Quality) than FDD 06 alone showed. Where
FDD 20 does go further than anything previously reviewed is its **Flag** concept — a status
marker, distinct from a Comment, that creates a routed "Action Item" for a user, role, or
worklist. Nothing in `communications-capability.md` or elsewhere in the SDD models a
generic cross-domain Flag/Action-Item mechanism today; each domain currently has its own
narrower version (staffing's justification-required conditions, credentialing's internal
processing notes). This is recorded as Coverage Gap #2 and needs an ownership decision
(`communications` vs. per-domain — there is no separate `worklists` platform capability to
consider, per `solution-architecture.md`'s Worklist/Pending-Items working assumption)
before it can be drafted.

**3. Does the EOY REP Comparison or REP_DQ Log reveal any concrete validation rules or
discrepancy categories not already captured in staffing-domain.md's validation model?**
**No new categories — concrete, large-scale corroboration of categories already
identified as open technical questions.** The EOY REP Comparison spreadsheet is a
statistical year-over-year anomaly-detection tool (33%/75% change thresholds at
state/ISD/district roll-up levels) — exactly `staffing-technical-design.md`'s existing
"Historical/trend anomaly detection" category and Open Technical Question #3, now with real
(if internally-sourced, not FDD-specified) threshold numbers. The REP_DQ Log's 200+
individual `DQCCYY##` rules are almost entirely instances of categories
`staffing-technical-design.md` already names: field-level/cross-field consistency checks
keyed to legacy REP field positions (Open Technical Question #1's "legacy REP field-position
rules"), and EEM cross-validation checks like `DQRP1092` ("Buildings Without Staff") and
`DQRP1097` ("Open Schools Without Instructional Staff") that independently confirm the exact
gap the FDD 18/21/22 review already found and fixed via `DistrictAuditReport
.buildingsNoAppropriatePositions`. No rule in either spreadsheet describes a business
concept staffing's model doesn't already have a category for — the outstanding work is
entirely in the already-tracked Open Technical Questions (whether/how to port vs.
re-derive the rule catalog, and what engine executes cross-record/collection-level checks),
not in discovering new rule shapes.

## Process/Template Friction Noted

- Two of the three FDD 19 source files were oversized Excel-to-Markdown conversions
  (292KB and 551KB, one exceeding the Read tool's per-call token limit even at a 150-line
  slice) — consistent with the README's expectation to "skim for structure," but worth
  noting for future reviews of similarly large spreadsheet conversions: a full read is not
  feasible in one call, and the file's own internal structure (sheet names, column guides)
  had to substitute for a first-pass table of contents.
- This review confirms the pattern predicted in the 18-21-22 review's own "Process/Template
  Friction Noted" section — that FDD 19/20 (two side-by-side Data Quality folders) would
  make a sensible combined review, which it did; no friction from covering both folders in
  one file.
- **Superseded note:** this section originally recorded that a `solution-areas/worklists/worklists-permissions.md`
  doc had appeared (not present as of the FDD 10/18-21-22 reviews, which both marked
  `worklist (platform capability, unreviewed)`) and that future reviews should check it
  rather than treating `worklists` as wholly unreviewed. That folder has since been removed:
  the working assumption is now that "worklist" is a UI/API pattern over each domain's own
  filtered list endpoint, not a separate platform-capability domain with its own permissions
  doc (see `solution-architecture.md`'s Worklist/Pending-Items working assumption). Future
  reviews should check the relevant domain's own `-permissions.md`/`-domain.md` for
  worklist-routed Action Items instead. The still-missing piece — a generic `Flag`/`ActionItem`
  concept — remains Coverage Gap #2 above, now framed as a `communications`-vs-per-domain
  ownership question rather than a `worklists`-domain one.
