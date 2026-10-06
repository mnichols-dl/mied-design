# Recommended Changes: Reporting

**Impact: Low.** Reporting only hosts catalog entries; it has nothing to build for question set analytics beyond a catalog entry. Files under `design/working-docs/solution-areas/reporting/`.

## Changes

1. **Report catalog entries.** When analytics on responses are wanted (completion rate, halt rate per question, time to complete, version adoption), register Power BI reports in the catalog (`reporting-api.yml` `/admin/reports`, L178). The data comes from Synapse or the data warehouse fed from `questionsets-db`; Reporting does not read the capability's API.
2. **Permission naming.** Per the capability-first convention decided 2026-09-04, report viewing for question set analytics is `reporting.questionsets-reports.view` (or `reporting.credentialing-reports.view` if the report is a credentialing analytic that uses response data). Add to `reporting-permissions.md`. Reports of response content (not just counts) need the owning domain's access rules; do not expose answer text through a general reporting catalog without that gate.
3. **Vocabulary collision.** Reporting uses "dataset" for Power BI datasets (`reporting-capability.md` L203, `reporting-technical-design.md` L52, L131, L206-228). Question Sets uses "option list" to avoid the clash. Add a one-line note to the Reporting ubiquitous language if the collision is likely to confuse authors.
4. **Standard parameter set** (UserUniqueId, OrgType, OrgCode, OrgAncestorCodes, UserRole, L241-258): no change.
5. **POC reports tab.** The POC "Reports" screen (monthly volumes, pending review counts) is Credentialing analytics, not Question Sets, and belongs in the Reporting catalog under Credentialing.
