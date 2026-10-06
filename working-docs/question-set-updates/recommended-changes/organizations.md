# Recommended Changes: Organizations

**Impact: Low.** Organizations becomes an option list provider; no model change. Files under `design/working-docs/solution-areas/organizations/`.

## Changes

1. **Downstream consumers table** in `organizations-capability.md`: add Question Sets as a consumer of organization search for option lists (districts, institutions, universities, ISDs).
2. **Use the existing search and cache contract.** `GET /organizations/search?type=...` (`organizations-api.yml` L27) and `/organization-types` (L200) cover the lists. Consumers cache for 60 minutes using the canonical keys (capability L262-275). Question Sets follows the same TTL.
3. **Typeahead.** District and organization lists are large; confirm the search endpoint supports prefix or typeahead with a result limit. The POC caps its dropdown at 20 options as a symptom.
4. **Institution lists for educator preparation.** The POC lists Michigan universities and EPPs from "Transcript Exchange". EPP provider configuration lives in `epp`, not Organizations. Decide which of the two is the source for "universities and EPPs" and register one provider key. Filtering "by credential type" needs data from Credentialing or EPP, which Organizations does not hold, so the filter context comes from the consumer (see technical design, option list providers).
5. **Existing inconsistency to be aware of:** `solution-tech-standards.md` says Organizations uses Cosmos for its document model, while `solution-databases-WORKING.md` says Azure SQL. This does not affect the provider contract; do not cite either file as authoritative on the point.
