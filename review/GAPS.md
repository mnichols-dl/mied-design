# GAPS.md — FDD subject areas with no corresponding solution-design artifact yet

Per `fdd-review/CONVENTIONS.md`'s "Things NOT yet in the solution design at
all" section: when a whole FDD folder's subject area doesn't correspond to any of the 11
bounded contexts Phase 1 converted, it gets logged here instead of forcing `sd:Requirement`
nodes with no possible `sd:satisfiedBy`/rejection target. Grouped by FDD number.

## FDD 29 — Educator Credentialing - Certificates

FDD 29's own Dependencies section names ten other Business Specifications it relies on.
Of these, no corresponding solution design modeling exists (as of this FDD 29 review pass) for:

- **Business Specification #3 — role-based dashboard admin.** Only a stub
  (`dash:DashboardsContext`, `sd:status "stub"`) exists, minted by the earlier FDD 11/16
  pilot pass. FDD 29's own 29.1.8 (persistent certificate-dashboard-state preferences)
  is out-of-scope for Credentialing on this same basis.
- **Business Specification #4 — Business Rule Management.** Referenced throughout FDD 29
  as the source of per-certificate-type question sets, eligibility rules, and routing
  logic ("as defined in business rule management"), but no bounded context or admin-facing
  configuration surface for it exists anywhere in the 11 converted domains. Credentialing's
  own `CredentialDefinitionAggregate`/`EndorsementDefinitionAggregate`/
  `AssessmentDefinitionAggregate` model some of this configurability internally, but a
  distinct, cross-domain "Business Rule Management" authoring surface (as FDD 29 describes
  it) has no home.
- **"Educator Certificate – Application Processing (17)"** — referenced by FDD 29 as a
  distinct FDD ("the certificate application will be processed as defined by..."), not
  itself reviewed as part of this FDD 29 pass.
- **Data Collection & Compliance — Employee Assignment Details (22)** — referenced as
  supplying data consumed by credential application workflows; no corresponding domain
  conversion exists (22 is one of several Data Collection & Compliance FDD folders with no
  mapped bounded context at all).
- **Candidate Tracking and Recommendation (25)** — a related-but-not-identical concept
  (`epp:Seq_BulkUploadCandidateTrackingData`) exists in the real `epp` domain conversion,
  but FDD 25 itself wasn't reviewed as part of this pass, so it's unconfirmed whether that
  sequence actually covers what FDD 29's Dependencies section is pointing at.

Domains that DO already exist and were cross-checked as matching FDD 29's own Dependencies
claims: `iam` (User Authorization #1), `payments` (CEPAS #7), `communications` (Alerts #6),
`epp` (Manage EPP #13), `proflearning` (Professional Learning – Processing #28).

## FDD 02 — Reporting

FDD 02's own Dependencies/Assumptions sections name two areas with no corresponding
solution design modeling, independently confirming what FDD 29 had already flagged:

- **Interfaces and Integrations Setup (8)** — Reporting's own stated data-source
  dependency.
- **Technology Standards and Audit requirements (32.13)** — source of the
  Power-BI-as-reporting-framework decision.

## FDD 28 — Professional Learning – Processing

Independently confirms two Open Questions already on record in
`proflearning-domain.ttl` from the *other* side (this FDD reviews sponsor
applications; the existing questions were raised from the submission side via FDD 26)
— raises confidence these are real gaps, not stale documentation:

- **No `SponsorApplication` entity/aggregate/API anywhere** — blocks the entire
  admin-side approve/reject/edit/delete/email/comment workflow FDD 28 describes
  (`plrn:Question_SponsorApprovalWorkflowContradiction`, Open Question #9).
- **No "Mass Email to Sponsors" sequence** — FDD 28's own Wireframe 14 / "Email
  Sponsors" tab describes exactly the functionality `plrn:Question_MissingSponsorSequences`
  (Open Question #10) already flagged as referenced-but-unmodeled.

New findings from this pass, not previously on record anywhere:

- **No Program/ProgramApplication-level audit log** — described 4 separate times
  across this one FDD, no domain-model counterpart at all.
- **"Coupons and Payments" admin tab** — visible in a wireframe's persistent tab bar,
  zero narrative or wireframe backing anywhere in the document, no coupon concept in
  either `proflearning` or `payments`. Possibly orphaned UI, possibly undocumented
  scope — unresolved.

## Cross-cutting note — FDDs reviewed with no new whole-area gap

FDD 01 (User Management → `iam`), FDD 07 (Credit Card Payments and Refunds →
`payments`), and FDD 09 (Professional Practices → `profpractices`, one feature via
`reporting`) were all reviewed and map entirely onto already-converted bounded
contexts — no new "area doesn't exist at all" finding from these three. Their
within-domain gaps and contradictions are captured as `sd:Annotation` instances in
their own `fdd-review/*.ttl` files, not logged here (see `fdd-review/STATUS.md` for a
per-FDD summary of what those are).

## FDD 03 — Dashboards

Confirmed entirely unbuilt (`dash:DashboardsContext` remains a `sd:status "stub"`),
now the most cross-cutting whole-area gap in the project — referenced as a dependency
by 3 independent other FDDs (02, 27, 29) in addition to FDD 03's own 29 dependency
lines. Found a circular deferral bug in the source material itself: FDD 3.6 points to
FDD 11.7 for dashboard content, which itself points back to FDD 3. Recommendation:
keep Dashboards as its own bounded context rather than folding it into Reporting.

## FDD 04 — Business Rule Management

Confirmed genuinely missing — no bounded context or admin-facing rule-authoring
surface existed anywhere in the 11 Phase-1 domains, matching what FDD 29 had already
flagged as Business Specification #4. Minted a new stub context,
`bre:BusinessRuleEngineContext`, since "Business Rule Engine"/"BRM" is a real,
named-but-unresolved participant in both Credentialing's and EPP's own sequence
diagrams (not merely an FDD-side aspiration). Corroborated by FDD 10, 14, 15, 28, 29,
and possibly overlaps with FDD 19/20's own data-quality admin-console ask — worth
resolving as one decision rather than building two overlapping rule engines.

## FDD 08 — Interface and Integrations Setup

Resolves the long-standing "FDD 8" external dependency that FDD 02, 04, 10, 12, 15,
and 27 were each independently waiting on — it decomposes onto existing domains
(organizations, profpractices, proflearning, credentialing, staffing, reporting)
rather than needing a new bounded context. Two things it surfaces instead:

- The CEPI Data Warehouse piece is confirmed genuinely out of scope for MiEdWorkforce —
  explicitly owned by a separate CEDS/SLDS team per a reviewer's own rejection comment.
- **New gap**: Staffing has zero modeling for 4 named external systems it's expected
  to integrate with (MI Data Hub, CTEIS, NexSys, MSDS-TSDL).

## Systemic Staffing bulk-upload gap

Independently confirmed by 5 separate FDDs — 10 (Staffing Admin), 18 (Roster of
Positions), 21 (Roster of Employees), 22 (Employee Assignment Details), and 24
(Request for ID Collection's Employee-Roster half). Each describes bulk/batch upload
of staffing data with no corresponding `sd:BatchJob` or API surface in the `staffing`
domain. Worth one consolidated fix/decision rather than five scattered per-aggregate
notes.

## Missing general admin-action audit/versioning mechanism

Flagged by FDD 32 (Technology, Standards & Auditing — broadly, across 6+ stories) and
independently by FDD 14 (Professional Learning admin) and FDD 28 (Program/
ProgramApplication level, 4 separate mentions). Only `documents` has anything
resembling real audit logging anywhere in the project; nothing tracks who-changed-what
for admin actions elsewhere. A single cross-cutting audit capability (rather than
per-domain bolt-ons) looks like the right shape given how many domains independently
assume one exists.

## FDD 12 — System Administration - Admin Users

Not an IAM FDD despite its title (correctly stayed silent on FDD 01's Group/
impersonation findings) — instead surfaces its own new whole-area gap: a
Data Dictionary / system-metadata reference manual has no home anywhere in the
converted domains.

## "Educator Profile" composite page

Referenced by name in both FDD 17 (Temporary Credentials) and FDD 25 (EPP - User) as
a single composite profile page pulling together credentialing and EPP data, but never
modeled as a first-class thing anywhere in the graph.

## FDD 19/20 — Data Quality Review & Processing (+ Comments & Flags)

Confirmed as a real, still-unconverted admin-config capability (`dataquality` stub) —
but not a total void: pieces already exist scattered across Staffing, Communications,
and duplicated comment-handling logic in ProfPractices/EPP. Wireframe evidence
suggests the DQ admin console (19.2) and Business Rule Management (FDD 4) might
actually be the same underlying capability wearing two names — worth checking before
building both.

## FDD 22 corrects a prior GAPS.md claim

FDD 29's Dependencies section (see above) had said Employee Assignment Details (22)
had no corresponding domain conversion. FDD 22's own review pass found this false —
`staffing`'s `AssignmentAggregate` is a near word-for-word match. Superseded here for
the record; FDD 29's original bullet is left above unedited as the historical
starting point.

## FDD 26 / 28 — SponsorApplication gap (triple-confirmed)

`plrn:Question_SponsorApprovalWorkflowContradiction` (Open Question #9) and
`plrn:Question_MissingSponsorSequences` (Open Question #10) are now confirmed from
three independent angles: FDD 26 (Sponsor, the original source — traced the gap's
actual origin to a real, undocumented third approval actor, "OEE," that never made it
into the visible story text, plus a login-model contradiction), FDD 28 (Processing,
the admin side — no `SponsorApplication` entity/aggregate/API anywhere, no
Program/ProgramApplication audit log, and an orphaned "Coupons and Payments" admin
tab with zero narrative or data-model backing), and FDD 27 (Educator, confirms two of
FDD 28's findings from the citizen-educator side, and is a 3rd independent FDD
— after 02 and 29 — to flag Dashboards as missing).

## New instance-namespace convention question

FDD 32 (Technology, Standards & Auditing) spans multiple domains — really, none in
particular — and minted a scoped `t32:` instance namespace rather than forcing one
domain's ownership onto it. FDD 04, 08, and 12 have similar (if less extreme)
cross-domain shape. Worth ratifying this pattern in `fdd-review/CONVENTIONS.md` if
more cross-cutting FDDs come up in a future pass; not done yet since only one case
has occurred so far.
