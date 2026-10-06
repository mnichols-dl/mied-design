# Recommended Changes: Staffing

**Impact: Low. Boundary note only.** Files under `design/working-docs/solution-areas/staffing/`.

## Changes

1. **Do not fold Staffing validations into Question Sets.** `staffing-technical-design.md` L14-60 defines a Validation Rule Engine: a 100+ rule catalog, five rule categories, an admin screen assigning a validation to a Collection, Category or Data Element with severity, message and required justification, and `QualityReviewResult` payloads for errors, warnings and justification-required. This is data validation of collections, which is a different problem from asking a person questions. Add a short boundary note to the technical design: staffing validations are not question sets; if a collection validation needs a free-text justification from a user, that remains a justification field on the validation (`CredentialErrorJustification`, `COLLECTION_JUSTIFICATIONS`), not a question set.
2. **Shared concern to track, not solve here.** Staffing's Error and Warning severity with required justification overlaps FDD 4.2 (Business Rule Management) and the "warn" outcome the POC lacks. This is the BRM and Data Quality decision (DESIGN-SPIKES item 1 and GAPS FDD 19/20: "might be the same underlying capability wearing two names"). Reference that decision from the staffing doc; do not duplicate it.
3. **Mentor validation and employment lookups.** Credentialing's permit mentor question needs staffing endpoints (credentialing Open Technical Question #8). That is a consumer-side validation after a question set answer, not a Question Sets dependency. Keep it in the Credentialing and Staffing contract.
4. **Prefill sources.** "Years of teaching experience" and assignment history come from placement and assignment data owned by Staffing. Register them as prefill sources whose values Credentialing supplies. Confirm that Staffing exposes the needed read (for example employment history for a unique id) to Credentialing.
5. **Permissions.** No change. `staffing.admin.manage-validations` stays.
6. **From the 9/22 notes:** candidate tracking (STAR), districts report, and "tie to Educator Evaluation and Qualifying" are Staffing and Reporting topics that came up around the POC but are not Question Sets work. Log them with Staffing's backlog if they are not already tracked.
