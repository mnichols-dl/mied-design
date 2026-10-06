# Review: Public Search (31)

| Field | Value |
|---|---|
| Source Document(s) | `31 - Public Search/31.1, 31.2 - Public Search - Educator Credential.docx`<br>`31 - Public Search/31.3 - Public Search - Legacy Reports.docx`<br>`31 - Public Search/31.4 - Public Search - Pro Prep.docx`<br>`31 - Public Search/31.5 - Professional Learning Search.docx` |
| Related Domain(s) | credentialing, epp, proflearning (each owns its own public search surface); solution-level/solution-architecture.md (capability-boundary question) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-31 |
| **Overall Status (per document)** | 31.1,31.2 (Educator Credential): **Gaps Identified** (fixed directly — public search/detail endpoints and `credentialNumber` field added to credentialing). 31.3 (Legacy Reports): **Not Applicable / Superseded** (feature explicitly not carried into MiEdWorkforce, per the FDD's own text). 31.4 (Pro Prep): **Gaps Identified** (partially fixed directly — public provider search/detail added; Program-centric catalog and City/Website fields remain open). 31.5 (Professional Learning Search): **Aligned — Minor SDD Revisions Needed** (already the most mature of the four — one real public-facing endpoint existed; minor field/access-flag gaps fixed directly, Sponsor address/ID gap flagged as an open question). |

## Summary

This folder is four independent public-facing search experiences — Educator Credential Search,
Educator Preparation Provider ("Pro Prep") Search, Professional Learning Resources Search, and a
withdrawn "Legacy Reports" stub — each with its own landing-page hyperlink, its own search bar,
its own result schema, and its own detail page. None of the four FDDs describe a single combined
search box or cross-domain result type; a member of the public who wants credential data and
program data visits two entirely separate pages. That answers this review's central boundary
question directly (see "Shared Capability vs. Per-Domain Endpoints" below): **no shared "search"
platform capability is warranted — each domain should (and mostly does or now does) own its own
public, unauthenticated, read-only endpoint**, the same resolution already applied to the
Worklist/Pending-Items pattern in `solution-architecture.md`. Confirmed no `search`/`public-search`
capability exists anywhere in the Domain and Capability Catalog today. Coverage was very uneven
going in: **Professional Learning already had a working `x-access: external-public` endpoint**
(`GET /catalog/search`), while **Credentialing and EPP had no public endpoint at all** despite
`credentialing-domain.md` explicitly naming a `publicportal` downstream consumer for "Public
lookup of valid credentials" that had never been built. Both gaps are now closed with new
`external-public` endpoints modeled on each domain's existing internal schemas. Along the way this
review also found a genuine missing attribute — `IssuedCredential` had no human-facing
`credentialNumber` field anywhere in the credentialing SDD despite it being a CEDS-aligned element
central to both the FDD's search and its results display — fixed directly. Two data-model
questions (EPP provider City/Website; Professional Learning Sponsor mailing address and its
human-readable Sponsor ID) don't have an unambiguous SDD answer and are left as internal Open
Questions rather than guessed at.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | 31.1/31.2 Wireframes 1-4; 31.4 Wireframes 1-5; 31.5 Wireframes 1-2 | Exhaustively enumerate exact result-table columns, icon placement (hover tooltips, expand/collapse '+'/'-' vs. hyperlink/'x' conventions), and button labels ("Back to List Results," "Learn about Credential Types") | Same shape as prior reviews' wireframe-column findings (e.g. 29-educator-credentialing-certificates) — UI/screen-inventory detail, not a business requirement. The underlying data these tables surface is either already modeled or is exactly the coverage gap this review identifies. | No SDD action needed; consistent with precedent — note the *data* the columns need (addressed in Coverage Gaps), not the widget choice. |
| 2 | 31.1/31.2 §Business Specifications: literal validation-error strings ("No results found.", "Enter search criteria in at least one field.", "Enter search criteria in at least one field along with credential type.") repeated near-verbatim across 31.1/31.2, 31.4, and 31.5 | Specifies exact user-facing copy | Borderline, not blocking — this is presentation-layer text, not a business rule, but it's short and low-risk to carry through as-is (unlike the wireframe-column detail above, there's no aggregate/schema decision buried in it). | No SDD action needed; leave for UI implementation, not a modeling concern. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | SDD vs. itself (fixed directly) | Public accessibility vs. declared `x-access` on Professional Learning's program/sponsor detail endpoints | `proflearning-api.yml`'s `GET /programs/{programId}` and `GET /sponsors/{sponsorId}` descriptions both stated "Also accessible unauthenticated via APIM for [public lookup]," but their `x-access` field was `internal-user` only — the single-value tag contradicted the document's own prose. | The established dual-access convention already exists elsewhere in the SDD: `credentialing-api.yml`'s `GET /educators/{educatorId}/credentials` uses `x-access: internal-user, internal-service` with an explicit "Non-standard — two access patterns" note. | Straightforward internal inconsistency — the prose was right, the access-control tag was stale/incomplete. | **Fixed directly.** Both endpoints now use `x-access: internal-user, external-public` with a clarifying note mirroring credentialing's dual-access convention, and their descriptions spell out what each access pattern returns (any status for internal-user; Active-only, 404 otherwise, for external-public). |
| 2 | FDD vs. SDD | Existence of a public Educator Credential Search endpoint | FDD 31.1/31.2, entire document: a fully-specified no-login search over issued credentials (name/credential-number search, Credential Type filter, sorted/grouped Active-vs-Inactive results, blank dates + adverse-action messaging for Nullified/Revoked/Suspended/PPR-Withdrawn). | `credentialing-api.yml` had zero `external-public` endpoints before this review — `GET /credentials` and `GET /credentials/{id}` are both `internal-user` only, scope-gated by `credentialing.credential.view`. `credentialing-domain.md`'s own Downstream Consumers table (line 566) already named `publicportal` — "Public lookup of valid credentials," "REST API call" — as a consumer, but nothing backed it. | Real, unambiguous gap — the FDD, the domain doc's own consumer table, and the actual API surface all pointed the same direction (a public endpoint is supposed to exist) but it had never been built. | **Fixed directly.** Added `GET /public/educator-credentials/search` and `GET /public/educator-credentials/{educatorUniqueId}` (`x-access: external-public`, no permission required) to `credentialing-api.yml`, plus a new "Public Educator Credential Search" business rule in `credentialing-domain.md` citing FDD 31.1/31.2. |
| 3 | FDD vs. SDD | Existence of a public Pro Prep provider/program search endpoint | FDD 31.4, entire document: a fully-specified no-login Providers/Programs search with filters (Provider Name, Program Type, Pathway, EPP Type, City) and detail pages. | `epp-api.yml` had zero `external-public` endpoints before this review. `epp-domain.md` names "Pro Prep" as a defined term ("Public-facing catalog of EPP programs and offerings") and has a "Pro Prep Visibility Based on Close Dates" business rule, but no endpoint enforced it publicly — only `GET /epp-providers/{eppCode}/proprepconfig` (internal-user, admin-configures visibility) and `GET /epp-providers/{eppCode}/approved-programs` (internal-service, consumed by Credentialing) existed. | Real, unambiguous gap for the Provider-search half (the underlying `EppProviderSummary`/`certificateCategories`/`pathway` data already exists and just needed a public-filtered read path). The Program-centric half is not equally straightforward — see Coverage Gap #1. | **Partially fixed directly.** Added `GET /pro-prep/providers/search` and `GET /pro-prep/providers/{eppCode}` (`x-access: external-public`) to `epp-api.yml`, filtered to Active providers and visibility-window-passing certificate categories/endorsements. The cross-provider Program search/detail flow (search by Program Name → "Available Providers" page) is **not** modeled — see Coverage Gap #1. |
| 4 | FDD vs. itself | Whether "Learn about Credential Types" is a client-provided static URL or a system-configured guidance link | 31.1/31.2 Business Specifications and Addendum 31.2.9 both describe the hyperlink identically ("MDE will provide the link" / "The specific link (URL) for this hyperlink will be provided by MDE") — no actual contradiction, just confirming consistency across the two sections of the same document. | N/A — restated here only because it was checked; the two sections agree. | Not a real discrepancy; already correctly modeled via `credentialing.guidance.view`/guidance documents in `credentialing-permissions.md`. | No action needed. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | 31.4 §Business Specifications, Wireframes 4-5 ("Programs" search button, "Program Name" search results, "Available Providers" page reached by clicking a Program Name) | A cross-provider "Program" concept searchable in its own right (by Program Name, independent of any one provider), with Program Code, Grade Band, Availability, and Classification as first-class fields. `epp-domain.md`/`epp-api.yml` model programs only as `certificateCategories`/`endorsements` nested *under* a provider (no standalone Program entity, no Code/Availability/Classification field anywhere in the domain). | New read/query capability in `epp-domain.md` (possibly a projection/materialized view across providers' certificate categories, not a new write-side aggregate) plus `epp-api.yml` endpoints (`/pro-prep/programs/search`, `/pro-prep/programs/{code}/providers`) | H | Not drafted — this needs a decision (see Open Question #1) on whether "Program" becomes a first-class query-side concept before endpoints can be designed; noted as `epp-domain.md` Open Question #12. |
| 2 | 31.4 §Wireframes 1-5 ("City," "Website" as both search filters and result/detail columns) | `city` and `website` fields for EPP providers. Neither exists in `epp-api.yml`'s `EppProviderSummary`/`EppProviderDetail` (only an unstructured `organizationAddress` string) nor in `organizations-capability.md`. | `epp-domain.md`/`epp-api.yml` (new fields) or `organizations-capability.md` (if sourced from EEM) — ownership undecided | M | Not drafted — see Open Question #1 (`epp-domain.md` Open Question #12); the new `/pro-prep/providers/*` endpoints added this review explicitly omit these two fields pending the decision. |
| 3 | 31.5 §Advanced Search ("Address city," "Address zip code," "County" as Sponsor-level filters; Sponsor detail page's "Address: Mailing") and Sponsor ID ("one alpha code followed by a 6-digit number") | Sponsor mailing address (city/zip/county) and a human-readable Sponsor ID distinct from the internal `sponsor_id` UUID. `proflearning-domain.md`'s `SPONSORS` ERD has neither; `county` otherwise exists only per-Session, not per-Sponsor. | `proflearning-domain.md` (`SPONSORS` ERD), `proflearning-api.yml` (`SponsorDetail`, `/catalog/search` filters) | M | Not drafted — see Open Question #2 (`proflearning-domain.md` Open Question #13); whether this is EEM-sourced or PL-specific data needs a decision first. |
| 4 | 31.1/31.2 §CEDS Data Elements table ("Credential Number"); Wireframes 2-4 (search field, results columns, sort/collapse keyed by Credential Number) | A human-facing `credentialNumber` field (e.g. "PV0000000772064," "CC-K46940078990") on issued credentials — this is a CEDS-aligned element the FDD treats as central, but no such field existed anywhere in `credentialing-domain.md`/`credentialing-api.yml` (only the internal `credential_id` UUID). | `credentialing-domain.md` (`ISSUED_CREDENTIALS` ERD), `credentialing-api.yml` (`CredentialSummary`, new public schemas) | H | **Fixed directly** — field added to the ERD and to `CredentialSummary`/`CredentialDetail`/the new public schemas. The actual generation/legacy-migration rule for the value itself is undefined by any FDD reviewed so far — tracked as `credentialing-domain.md` Open Question #35 (low priority; doesn't block the field's existence, only its population logic). |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `31.1, 31.2 - Public Search - Educator Credential` §Business Specifications, Wireframes 1-4, Addendum 31.1.1-31.2.9 | credentialing | `IssuedCredential`; new "Public Educator Credential Search" business rule; `GET /public/educator-credentials/search`, `GET /public/educator-credentials/{educatorUniqueId}` | implements — gap fixed directly (Discrepancy #2) |
| `31.1, 31.2` §CEDS Data Elements (Credential Number) | credentialing | `ISSUED_CREDENTIALS.credential_number` | implements — gap fixed directly (Coverage Gap #4) |
| `31.1, 31.2` §Business Specifications (blank dates for Nullified/Revoked/Suspended/PPR-Withdrawn, adverse-action message) | credentialing | "Certificate Document Generation" business rule (existing, from FDD 29 review); `PublicCredentialHistoryItem` | implements — reuses existing rule, extends to permits |
| `31.3 - Public Search - Legacy Reports` (entire document) | credentialing (n/a), reporting (n/a) | n/a — feature explicitly withdrawn per the FDD's own text | not applicable / superseded |
| `31.4 - Public Search - Pro Prep` §Business Specifications, Wireframes 1-3, Addendum 31.4.1-31.4.4, 31.4.7-31.4.10 (Providers search/detail) | epp | `EducatorPreparationProvider`; new `GET /pro-prep/providers/search`, `GET /pro-prep/providers/{eppCode}` | implements — gap fixed directly (Discrepancy #3) |
| `31.4` §Business Specifications, Wireframes 4-5, Addendum 31.4.5-31.4.6, 31.4.8 (Programs search, Available Providers page) | epp | (no cross-provider Program entity exists) | gap — not yet modeled (Coverage Gap #1; `epp-domain.md` Open Question #12) |
| `31.4` §Wireframes 1-5 (City, Website fields) | epp, organizations (unreviewed for this field) | (no field exists) | gap — not yet modeled (Coverage Gap #2; `epp-domain.md` Open Question #12) |
| `31.5 - Professional Learning Search` §Business Specifications, Wireframe 1, Addendum 31.5.1-31.5.6, 31.5.10-31.5.13 (basic search, results grid, Program/Sponsor detail) | proflearning | `ProfessionalLearningProgram`, `ProfessionalLearningSponsor`; `GET /catalog/search` (existing), `GET /programs/{programId}`, `GET /sponsors/{sponsorId}` | implements — access-flag fix applied directly (Discrepancy #1); result-field additions applied directly (category, beginDate, endDate on `CatalogSearchResult`) |
| `31.5` §Advanced Search (Sponsor ID, Address city, Address zip code, County) | proflearning | (no field exists on `SPONSORS`) | gap — not yet modeled (Coverage Gap #3; `proflearning-domain.md` Open Question #13) |
| `31.5` §Advanced Search (Sponsor filter) | proflearning | `/catalog/search` `sponsorId` query param | implements — added directly this review |

---

## Shared Capability vs. Per-Domain Endpoints

**Question:** Is a unified, anonymous, cross-domain "Public Search" a genuinely shared platform
capability (one search index/service spanning credentialing + EPP + proflearning), or should each
domain expose its own public, filtered, read-only endpoint — mirroring the resolution already
reached for the Worklist/Pending-Items pattern?

**Answer: per-domain endpoints, no new shared capability.** All four FDDs describe **entirely
separate landing pages, separate hyperlinks below the MiEdWorkforce banner, separate search bars,
and separate result schemas** — "Educator Credential Search," "Educator Preparation Providers
Search," and "Professional Learning Resources" are three distinct pages a member of the public
navigates to independently; there is no single search box, no combined result type, and no shared
UX described anywhere across 31.1-31.5. This is the opposite shape from `organizations`, which
exists specifically because *every* domain needs the same read-only EEM/org data and would
otherwise re-integrate with EEM directly — here, each domain's public data (issued credentials,
EPP providers/programs, SCECH sponsors/programs) is disjoint, sourced from that domain alone, and
already has (or, per this review, now has) its own natural home in that domain's own API. Building
a shared "search" capability would mean either (a) a thin façade that just proxies to three
separate domain endpoints — adding an indirection layer with no combined query it actually serves
— or (b) a real cross-domain search index, which no FDD asks for and which would violate each
domain's ownership of its own data/visibility rules (e.g. Pro Prep's close-date visibility window,
credentialing's blank-date-for-adverse-status rule) by centralizing logic that's genuinely
domain-specific. Confirmed via `solution-architecture.md`'s Domain and Capability Catalog that no
`search`/`public-search` capability exists today — this review does not add one, consistent with
the Worklist precedent's reasoning (`solution-architecture.md`'s "Worklist / Pending-Items
Pattern" note): each domain exposes its own filterable, unauthenticated list endpoint, gated by
`x-access: external-public` and no permission, rather than a first-class shared search platform
capability.

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Should "Program" become a first-class, cross-provider queryable concept in the EPP domain (to support FDD 31.4's Program-name search → Available-Providers flow), and where do the missing City/Website provider fields belong (EEM/Organizations vs. EPP-specific)? | Internal | Open — see `epp-domain.md` Open Question #12, Coverage Gaps #1-#2 |
| 2 | Does Professional Learning Sponsor mailing address (city/zip/county) come from EEM, or is it PL-specific data entered separately from the organization's official EEM address? Is the FDD's human-readable Sponsor ID a new PL-generated identifier or a legacy-MOECS code to migrate? | Internal | Open — see `proflearning-domain.md` Open Question #13, Coverage Gap #3 |
| 3 | What is the actual format/generation rule for the newly-added `credentialNumber` field (the `PV`/`CC-K` style prefixes seen in the FDD's examples) — system-generated at issuance, or migrated from legacy MOECS numbering? | Internal | Open — see `credentialing-domain.md` Open Question #35 |

None of the three clears the `client-questions.md` judiciousness bar: all three are field-sourcing/
ownership decisions the team can default on (add the field, document it as TBD-format, or defer
the cross-provider Program work) without blocking anything currently being built — the public
search endpoints added this review function correctly today with the data that does exist, and
none of these three would change what was just built if answered differently later.

## Related Documents

- Source documents: `functional-design-docs/31 - Public Search/31.1, 31.2 - Public Search - Educator Credential.md`, `31.3 - Public Search - Legacy Reports.md`, `31.4 - Public Search - Pro Prep.md`, `31.5 - Professional Learning Search.md`
- SDD documents reviewed against and modified: `solution-areas/credentialing/credentialing-domain.md` (Public Educator Credential Search rule added, `credentialNumber` added to ERD, Open Question #35 added), `solution-areas/credentialing/credentialing-api.yml` (`credentialNumber` field added; two new `external-public` endpoints and three new schemas added), `solution-areas/epp/epp-domain.md` (Open Question #12 added), `solution-areas/epp/epp-api.yml` (two new `external-public` endpoints and two new schemas added), `solution-areas/proflearning/proflearning-domain.md` (Open Question #13 added), `solution-areas/proflearning/proflearning-api.yml` (dual-access fix on two endpoints, `sponsorId` filter and `category`/`beginDate`/`endDate` fields added to the existing public catalog endpoint), `solution-level/solution-architecture.md` (read only — Domain and Capability Catalog and Worklist/Pending-Items note consulted, not modified)
- SDD documents reviewed, not modified: `solution-areas/credentialing/credentialing-permissions.md`, `solution-areas/epp/epp-permissions.md`, `solution-areas/proflearning/proflearning-permissions.md`, `solution-areas/organizations/organizations-capability.md`, `solution-areas/reporting/reporting-capability.md` (checked — Legacy Reports/31.3 does not map to this capability; it's Power BI embedded analytics, not the withdrawn MOECS legacy-reports feature)
- Related prior reviews: [29-educator-credentialing-certificates](29-educator-credentialing-certificates.md) (source of the existing blank-dates-for-adverse-status rendering rule this review extends to permits/public search)
