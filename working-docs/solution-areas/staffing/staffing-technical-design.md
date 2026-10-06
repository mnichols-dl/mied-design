# Staffing - Technical Design

## Purpose

This document provides technical design details for Staffing domain concepts that are
implementation-specific and don't belong in the domain model documentation. It covers
content flagged during the FDD 10 ("Data Collection & Compliance - Staffing Admin")
review as needing a home outside `staffing-domain.md`: the validation-rule engine
underlying `Collection`'s quality-review process, and the collection-to-warehouse batch
pipeline. Staffing previously had no technical-design document; this is its first version.

---

## Validation Rule Engine

### Overview

`staffing-domain.md`'s `Collection` aggregate models quality review at the business level
correctly (a `QualityReviewResult` with `errors_payload`/`warnings_payload`/
`justification_required_payload`, three severity tiers matching FDD 10's Error/Warning/Data
Quality classification). What it does not — and should not — capture is the actual rule
catalog: FDD 10's companion spreadsheet, `Staffing Submission Validations.xlsx`, documents
on the order of 100+ individual validation checks, many inherited verbatim from the
legacy MORE system's fixed-width record format (its "REP BRs" sheet references specific
field positions — e.g. `[Field 7] ... Social Security Number`, `[Field 12] Funded Position
Status Code` — from a byte-position record layout that predates MiEdWorkforce). This is
implementation/configuration detail for a rules engine, not a business rule catalog, and
belongs here rather than as a `staffing-domain.md` business rule per row.

### Rule Categories

The spreadsheet's rules fall into a few distinct shapes that likely need different
implementation treatment:

| Category | Example | Likely Implementation |
|---|---|---|
| Field-level syntax/format | "SSN must meet SSA definitions (no 000/666/900-999 area, no 00 group, no 0000 serial)" | Declarative field-level validator, evaluated per record on submission |
| Cross-field consistency | "If Employment End Date provided, Separation Reason required" | Already modeled as a `staffing-domain.md` invariant ("Employment End Date and Separation Reason Co-requirement") — no engine needed beyond aggregate-level validation |
| Cross-record / collection-level | "All open districts must report active staff," "If building has CTE in EEM, CTE assignments should be reported" | Requires EEM program-flag lookups at quality-review time, evaluated once per collection, not per record — see Coverage Gap noted in the FDD 10 review (`fdd-sdd-review/reviews/10-staffing-admin.md`) |
| Historical/trend anomaly detection | "Changes in Assignment counts by assignment groups if more than X% (headcount/FTE)," "Staff assigned to a facility a significant distance from your district" | Needs a defined comparison baseline (prior collection period) and threshold; not simply a per-record check — flagged as an Open Technical Question below |
| Legacy REP field-position rules (MORE-inherited) | The full "REP BRs" sheet (`F02_01`-`F26_6`) | These reference a legacy fixed-width record layout. Whether MiEdWorkforce's own data model needs an equivalent 1:1 rule-for-rule port, or whether these get re-derived from the CEDS-aligned data element model FDD 10 §10.8 describes, is unresolved — see Open Technical Questions |

### Admin Configuration Surface

FDD 10 Feature 10.3/10.13 ("Manage business rules and validations") describes an admin
screen for assigning a validation to a Collection/Category/Data Element, setting severity,
editing user-facing message text, and requiring justification. This is captured today by
`staffing-permissions.md`'s `staffing.admin.manage-validations` permission — the engine
that actually executes those assigned validations (how a "rule" is represented internally,
how the CEDS-aligned data-element model informs which rules apply to which submissions) is
the technical gap this section exists to eventually fill in, not the admin-facing
configuration surface itself.

---

## Collection-to-Warehouse Batch Processing

### Overview

FDD 10 Feature 10.15 ("Data Closeout and Processing") describes an admin-configurable
option to push finalized (certified) collection data to the CEPI CEDS Data Warehouse
either immediately or on a scheduled future date. `staffing-domain.md`'s Downstream
Dependencies table already names the mechanism ("reporting (CEPI Data Warehouse): Synapse
Pipeline after certification") but doesn't describe the pipeline itself.

### Pipeline (as implied by `CollectionCertified`)

1. `Collection` aggregate transitions to `Certified` (see "Certify Collection" sequence in
   `staffing-sequences.md`)
2. `CollectionCertified` event published
3. Synapse Pipeline consumes the event (or a scheduled trigger, per FDD 10.15's "schedule
   some or all data closeout processing for a future date" option) and extracts finalized
   `EmployeeRoster`/`PositionRoster`/`Assignment` records for the certified collection
4. Data lands in the CEPI CEDS Data Warehouse for standardized state/federal reporting

### Open Question

FDD 10 itself flags this as incomplete ("Full implementation details for the data
processing closeout and connections to Data Warehouse to be determined") — see Open
Technical Questions below.

---

## Open Technical Questions

| # | Question | Impact | Owner | Target Date |
|---|----------|--------|-------|-------------|
| 1 | Should the ~100+ rows in `Staffing Submission Validations.xlsx` (particularly the legacy "REP BRs" sheet, keyed to a MORE fixed-width field-position layout) be ported 1:1 into the new validation engine, or re-derived from the CEDS-aligned data-element model FDD 10 §10.8 describes? A 1:1 port perpetuates legacy field-position framing that has no meaning in MiEdWorkforce's own data model. | H | Product / Engineering | TBD |
| 2 | How are cross-record/collection-level validations (e.g., "all open districts must report active staff," "if building has CTE in EEM, CTE assignments should be reported") evaluated — as part of the same per-record quality-review pass, or a separate collection-level pass? These require EEM program-flag lookups not currently modeled anywhere in `staffing-domain.md`. | M | Engineering | TBD |
| 3 | What baseline and threshold apply to the trend/anomaly-detection style Data Quality checks (e.g., "changes in Assignment counts by more than X%," "staff assigned to a facility a significant distance from the district")? FDD 10's source spreadsheet doesn't define X% or "significant distance." | M | Product | TBD |
| 4 | What are the mechanics of the CEPI CEDS Data Warehouse Synapse Pipeline — full extract vs. incremental, retry/failure handling, and what "schedule some or all data closeout processing for a future date" means operationally (partial collection push)? FDD 10 explicitly defers these details. | M | Engineering (staffing + reporting) | TBD |

---

## Related Documents

- [Domain Model](./staffing-domain.md)
- [Workflows & Sequences](./staffing-sequences.md)
- [Permissions Catalog](./staffing-permissions.md)
- [API Contracts](./staffing-api.yml)
- FDD source: `functional-design-docs/10 - Data Collection & Compliance - Staffing Admin/`

---

**Version History:**
- v1.0 (2026-08-14): Initial version - validation rule engine categorization and collection-to-warehouse batch pipeline, drafted from the FDD 10 review (`fdd-sdd-review/reviews/10-staffing-admin.md`)
