# FDD review status

Running tracker for Phase 2 (FDD ingestion — see `CONVENTIONS.md`). **All 31 FDD
folders under `solutions/MiEdWorkforce/inputs/fdd/` have now been reviewed**
(19/20 combined into one ingestion as companion documents, matching the 11/16
precedent — 20 folders, 19 files). `sd:UserStory` counts only exist for passes done
after that class was added to the ontology — the five earliest passes (05, 11-16, 17,
29, 31) still say plain `sd:Requirement` for what are, in substance, the same numbered
user stories; retype opportunistically when next touching those files, per
`CONVENTIONS.md`.

Whole-tree validation (84 `.ttl` files, ontology + shapes + the whole solution design): parses
cleanly, **0 unaccounted-for Requirement-family instances, 0 untyped object-property
targets, 736 total Requirement/UserStory/NonFunctionalRequirement instances.**

## All FDDs

| FDD | Domain(s) | Stories | Satisfied | Gap | Rejected/OOS | Notable |
|---|---|---|---|---|---|---|
| 01 – User Management | iam | 27 (26 UserStory) | 15 | 11 | 1 | **Two high-impact contradictions**: assumes a "User Group" layer IAM already decided to drop; re-describes an impersonation/"View As" feature already evaluated and excluded (4th document on record wanting it — user says POs may be open to dropping it if admin screens land). Also: Role CRUD gap, Worker-onboarding gap. |
| 02 – Reporting | reporting | 12 (6 US + 6 Req) | 7 | 3 | 2 | Two FDD-self-inconsistent gaps found only by viewing wireframes ("Delete Report", "Favorite Reports") with no backing story. Confirmed FDD 08/32 as dependencies (08 since resolved, see below). |
| 03 – Dashboards | dashboards (stub, confirmed real gap) | 18 (7 US + 11 Req) | 1 partial | 12 | 5 | Confirmed entirely unbuilt — but the most cross-cutting FDD reviewed (29 dependency lines). Found a circular deferral bug: FDD 3.6 points to FDD 11.7 for content, which itself points back to FDD 3. Recommends keeping Dashboards as its own bounded context. |
| 04 – Business Rule Management | *(new stub)* `bre:BusinessRuleEngineContext` | 20 (all US) | 0 | 20 | 0 | Confirmed genuinely missing — minted a new stub context. "Business Rule Engine"/"BRM" is a real, named-but-unresolved participant in credentialing's and epp's own sequence diagrams. |
| 05 – System Admin (Credentialing) | credentialing | 28 (pre-`sd:UserStory`) | 15 | — | — | Predates this session. |
| 06 – Alerts, Emails, Communications | communications | 20 (18 US + 2 Req) | 18 | 2 | 0 | Real architecture contradiction: FDD wants a rich HTML editor, tech design explicitly rejected that for Markdown-only/XSS reasons. Found a genuinely unmodeled second email channel (Outlook/Graph API) only by reading the docx directly. |
| 07 – Credit Card Payments and Refunds | payments | 11 (10 US) | 11 | 0 | 0 | Unusually complete — essentially the source spec Payments was built from. New gap: no validation blocks paying a fee on a stale/post-school-year application. Live unresolved contradiction on payment success/error messaging. |
| 08 – Interface and Integrations Setup | organizations, profpractices, proflearning, credentialing, staffing, reporting | 49 (all US) | 19 | ~23 | 8 | **Resolves the long-standing "FDD 8" dependency** other passes (02, 04, 10, 12, 15, 27) were waiting on — not a missing bounded context, decomposes onto existing domains. CEPI Data Warehouse piece confirmed genuinely out of scope (owned by a separate CEDS SLDS team per explicit reviewer rejection). New gap: staffing has zero modeling for 4 named external systems (Data Hub, CTEIS, NexSys, MSDS-TSDL). |
| 09 – Professional Practices | profpractices (+ reporting) | 24 (all US) | 21 | 0* | 2 | Confirmatory pass (domain was written with this FDD open). One architecture-vs-story discrepancy (on-demand compliance calc vs. expected stored/reset flag). *Narrower gaps folded into `sd:notes` per "partial satisfaction is normal." |
| 10 – Staffing Admin | staffing | 27 (19 US + 8 Req) | 19 (5 design-intent only) | 4 | 4 | Confirms systemic Staffing bulk-upload gap (4th time). Bigger finding: almost the entire Staffing admin-config layer (6 of 7 `Perm_Admin*`) has no backing API — the FDD's own reviewers independently called most of these "still under internal discussion." |
| 11/16 – Educator Credentialing | credentialing | 26 (pre-`sd:UserStory`) | 20 | — | — | Predates this session. |
| 12 – System Administration - Admin Users | credentialing/staffing (12.1), publicportal-stub (12.2), dashboards-stub (12.3), homeless (12.4) | 15 (all US) | 2 | 13 | 0 | **Not an IAM FDD** despite its title — correctly stayed silent on FDD 01's Group/impersonation findings. New whole-area gap: Data Dictionary/system-metadata reference manual has no home anywhere. |
| 13 – EPP - Admin Users | epp | 15 (11 US + 4 Req) | 10 | 1 | 4 | A "Worklist Management" wireframe shown as an EPP admin screen has a Credentialing breadcrumb — independently confirms a pre-existing open question that worklist admin may not belong to EPP at all. |
| 14 – Professional Learning - Admin Users | proflearning | 19 (16 US + 3 Req) | 11 | 6+ | 3 | Extends FDD 28's audit-log finding: no general admin-action audit/versioning mechanism exists anywhere in Professional Learning's admin surface (not just for programs). |
| 15 – User Management - Identity Mgmt Integration | iam | 47 (43 US + 4 Req) | 34 (2 partial) | 9 | 4 | Strongest coverage yet — this was clearly the primary source spec for iam's identity-resolution model. Confirms FDD 13.7's withdrawal was correct. Flagged likely duplicate-effort overlap with FDD 24 (confirmed real and deliberate by FDD 24's own pass). |
| 17 – Temporary Credentials | credentialing | 30 (pre-`sd:UserStory`) | 27 | — | — | Predates this session. |
| 18 – Roster of Positions | staffing | 49 (all US) | 29 | 15 | 5 | One of the best-covered FDDs in the project — staffing's v1 model maps almost 1:1. First to surface the systemic Staffing bulk-upload gap. |
| 19/20 – Data Quality Review & Processing (+ Comments & Flags) | dataquality (stub, cross-cutting) | 25 (24 US + 1 Req) | 9 | 15 | 1 | Confirmed as a real, still-unconverted admin-config capability — but not a total void: pieces already exist scattered across Staffing, Communications, and duplicated comment-handling in ProfPractices/EPP. Wireframe evidence suggests the DQ admin console (19.2) and Business Rule Management (FDD 4) might be the same underlying capability. |
| 21 – Roster of Employees | staffing | 23 (22 US + 1 Req) | 18 | 4 | 1 | 5th confirmation of the systemic Staffing bulk-upload gap. Found a live unresolved contradiction: a reviewer says adding an employee with no Unique ID is "not a valid use case," but the converted domain model does exactly that. |
| 22 – Employee Assignment Details | staffing | 20 (18 US + 2 Req) | 17 | 1 | 2 | **Corrected a prior GAPS.md claim** — FDD 29 had said this area had no domain conversion; it does (staffing's `AssignmentAggregate`, near word-for-word match). |
| 24 – Request for ID Collection | staffing + iam | 12 (11 US + 1 Req) | 11 | 1 | 0 | Confirmed the FDD-15 overlap flagged by that pass is real and deliberate — two independently-authored source docs converging on the same identity-request flow, documented in both files rather than merged. |
| 25 – EPP - User | epp | 11 (all US) | 11 | 0 | 0 | Resolved FDD 29's open question cleanly: `Seq_BulkUploadCandidateTrackingData` really does cover this FDD's content. New finding: an "Educator Profile" composite page is referenced by name in two FDDs (17, 25) but never modeled as its own thing. |
| 26 – Professional Learning - Sponsor | proflearning | 21 (20 US + 1 Req) | 13 | 7 | 1 | **Original source** of the SponsorApplication gap (Open Question #9) — traced its actual origin through the comment history: a real, undocumented third approval actor ("OEE") never made it into the visible story text, plus a login-model contradiction. Now triple-confirmed (26 + 28 + permissions catalog). |
| 27 – Professional Learning - Educator | proflearning | 14 (13 US + 1 Req) | 7 | 6 | 1 | Confirms two of FDD 28's findings from the citizen-educator side. Dashboards now flagged missing by a 3rd independent FDD (02, 29, 27). |
| 28 – Professional Learning – Processing | proflearning (+ communications, documents) | 27 (26 US) | 14 | 12 | 1 | **High-impact, multi-FDD-confirmed gap**: no `SponsorApplication` entity anywhere. Also: no Program/audit log anywhere; an orphaned "Coupons and Payments" UI tab with zero backing. |
| 29 – Educator Credentialing – Certificates | credentialing | 41 (pre-`sd:UserStory`) | 32 | — | — | Predates this session. Named 5 dependency areas — see `GAPS.md`. |
| 30 – Interface Integration - Rapbacks | profpractices | 24 (21 US + 3 Req) | 7 | 17 | 0 | profpractices owns the integration plumbing (`ExternalBackgroundCheckAggregate`, real sequences) but the FDD describes a much larger, mostly-unmodeled admin console run by a "Rapback Admin" role that doesn't exist anywhere. |
| 31 – Public Search | credentialing (+ others) | 39 (pre-`sd:UserStory`) | 23 | — | — | Predates this session. |
| 32 – Technology, Standards & Auditing | cross-cutting (new `t32:` instance namespace, no owning domain) | 42 (31 US + 2 Req + 9 NFR) | 18 | 22 | 2 | **Top finding**: general audit-logging is assumed as a working capability by 6+ stories here (and independently by FDD 14), but doesn't actually exist anywhere except Documents' own narrow audit log — a clear systemic gap. Resolved 32.13 (Reporting) as satisfied. |

**Two ingestion waves this session: 25 FDDs (folders), 635 new Requirement/UserStory/NFR
instances, all validated together with the 5 pre-existing passes (101 total across both
waves + 5 pre-existing = 736 whole-tree). Zero parse failures, zero untyped references,
zero unaccounted-for Requirements across the entire solution design graph.**

## Cross-cutting patterns worth a second look

- **Systemic Staffing bulk-upload gap** — independently confirmed by 5 separate FDDs
  (10, 18, 21, 22, 24's Employee-Roster half). Worth one consolidated fix/decision
  rather than five scattered per-aggregate notes.
- **No general admin-action audit/versioning mechanism** — flagged by FDD 32 (broad)
  and FDD 14 (Professional Learning specifically); only Documents has anything
  resembling real audit logging anywhere in the project.
- **Dashboards (FDD 03) confirmed unbuilt, referenced by everyone** — 3 independent
  FDDs (02, 27, 29) plus FDD 03's own 29-dependency count name it.
- **Business Rule Management (FDD 04) confirmed unbuilt** — corroborated by FDD 10,
  14, 15, 28, 29, and possibly overlapping with FDD 19/20's own admin-console ask.
- **"Educator Profile" composite page** referenced by name in FDD 17 and FDD 25, never
  modeled as a first-class thing.
- **New instance-namespace convention question**: FDD 32 (and to a lesser extent 04,
  08, 12) span multiple domains or none at all — FDD 32 minted a scoped `t32:` prefix
  rather than forcing one domain's ownership. Worth ratifying in CONVENTIONS.md if more
  cross-cutting FDDs come up in a future pass.
