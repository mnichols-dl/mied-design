# DESIGN-SPIKES.md — items needing deeper design/POC work before they can become stories

Category 4 from the FDD-review triage: findings where the gap is real *in our own unreviewed
draft SDD* — the fix isn't a straightforward domain edit, it needs a concrete design proposal
(and often a POC) worked out with AO/POs before it can turn into stories. This SDD has not been
reviewed or approved by the client; where an item below describes what our draft "already does,"
that's our own unconfirmed design work, not a client-validated baseline. Engineering-facing, not
PO-facing — cross-reference into `PO-FEEDBACK.md` where a design choice also needs PO sign-off.

Each item: **Status** (open / in-progress / proposed / done), source FDD(s), why it needs design
work rather than a direct edit, and a starting-point sketch of the shape a proposal might take.

---

## 1. Business Rule Engine (BRE) — elevate from stub to a real bounded context
- **Status:** open
- **Source:** FDD 04 (Business Rule Management), corroborated by FDD 10, 14, 15, 28, 29
- **Related PO item:** none yet — this is a pure design/engineering gap, not awaiting a PO
  decision

`bre:BusinessRuleEngineContext` currently exists only as a bare stub with no Aggregate/
ApiOperation/Sequence. But the BRE isn't a passive, background rules repository — every sequence
that touches it (`credentialing-sequences.ttl`, `epp-sequences.ttl`) calls it **synchronously,
as a blocking step inside user-facing workflows**: "Get questions for certificate type,"
"Validate all eligibility criteria," repeated across permits, reciprocity, endorsements,
renewal, batch renewal, and assessment-combination logic. Both files' own header notes flag BRM
as an "unmodeled internal collaborator... nothing for them to resolve to."

This means the BRE needs to be designed as a first-class participant with real API operations
matching the calls already implied by the existing sequences (eligibility validation, question
retrieval, rule evaluation with a result/reason payload), not just an admin-authoring UI for
toggling rules. The FDD 04 stub *does* separately describe a real admin-authoring surface (rule
CRUD, versioning, effective dates, severity, preview) — that's a second, distinct piece of scope
alongside the runtime evaluation API.

**Suggested shape for the design pass:**
- Split the work into two aggregates: a runtime `RuleEvaluationAggregate` (or similar) that
  credentialing/EPP sequences call synchronously, and a `RuleDefinitionAggregate` for the
  admin-authoring side (versioned rules, effective dates, single-active-version enforcement).
- Design the runtime API contract first (request: entity + context data; response: pass/fail +
  reasons + triggered actions) since it's referenced from the most places and unblocks retyping
  those "unmodeled collaborator" sequence steps into real Sequence references.
- Fold in the historical-versioning cutover question from `PO-FEEDBACK.md` item 7 as an input to
  the definition side's design, not a separate follow-up.

## 2. Question Sets — resolve the credentialing/EPP eligibility-question workflow
- **Status:** open
- **Source:** FDD 04 (admin authoring), FDD 05 (your existing POC on requirement-resolution),
  FDD 29 (dependency reference)
- **Related PO item:** bring the POC back to AO/POs per your note below

`qset:QuestionSetContext` is a bare stub (one GET-only operation, "no questionsets-api.yml
exists anywhere in this project to confirm against"). FDD 04's Feature 4.1 (9 user stories)
describes a full admin-authoring surface — search, edit/copy/version, effective dates,
single-active-version enforcement, preview — none of which the stub implements.

You've already sketched a POC direction (per your note on FDD 05): credential requirements
should resolve through a standard sequence — data integration match, then uploaded document,
then fall back to a manual question-set response — with the goal of never asking a question the
system can already answer from existing data. That's exactly the kind of proposal this spike
should produce concretely enough to demo.

**Suggested shape for the design pass:**
- Treat this as one combined POC with item 1 (BRE) rather than two separate spikes — Question
  Sets are how BRE's synchronous "Get questions for certificate type" call is actually answered,
  so their APIs need to be designed together.
- Model the requirement-resolution priority order (integration → document → question) as an
  explicit business rule/state machine on the requirement itself, not just a UI concept — this
  is what will let "why are we asking this" be inspectable later.
- Demo target: bring this back to AO/POs as you planned, using the combination-logic pattern
  already established for `RequiredAssessments`/`AssessmentDefinitionAggregate` in credentialing
  as a reference for how "definition + resolvable data + fallback" is already modeled elsewhere
  in this ontology.

## 3. Report catalog: draft/preview + availability window
- **Status:** open
- **Source:** FDD 02 (Reporting)
- **Related PO item:** none required to start — this can likely proceed as a direct design
  proposal, with POs confirming the resulting behavior rather than debating the mechanism

Smaller in scope than items 1–2, but still needs an explicit design decision rather than a
one-line ontology edit, since it touches the report lifecycle state machine. Currently
`ReportDefinitionAggregate` only has a two-state Active/Inactive lifecycle — no draft/preview
concept, no begin/end availability dates, despite FDD 02's own wireframe showing a "Preview"
action before "Add Report," and `Req_02_BizSpec_1`/`_2` explicitly asking for report versioning
and an availability window.

Your framing in your notes points toward the shape of a proposal: "all report usage is done
THROUGH MiEd... one point of entry for the general public, admins can see more" — our draft's
broker model already has Reporting sit between the client and Power BI rather than letting the
client touch Power BI directly, which is consistent with a report being technically "published"
in Power BI but gated by MiEd's own catalog status. That's our own draft's architecture, not a
client-confirmed constraint, but it does mean this addition wouldn't be fighting the existing
pattern.

**Suggested shape for the design pass:**
- Add a Draft/Published (or similar) status to `ReportDefinitionAggregate` alongside the
  existing Active/Inactive, plus `AvailableFrom`/`AvailableTo` dates.
- Add a `reporting.catalog.preview` permission (or equivalent) gating access to
  unpublished/pre-availability-window reports, following the same `reporting.{domain}-reports.*`
  pattern already established (see item 4 below — that pattern now lives in
  `reporting-permissions.ttl` itself, not each domain's file).
- Resolve `rpt:Question_DatasetIdDiscoveryStrategy` (`datasetId` missing from
  `ReportDefinition`/`ParameterContractVO`, needed for Power BI RLS's `identities.datasets`
  field) as part of the same pass, since it's a small, adjacent, already-flagged gap in the
  same aggregate.
- **Added 2026-09-04, expanding this spike's scope:** bring Reporting's per-domain permissions up
  to the same create/edit/activate/delete verb richness Communications has for templates. Today
  every report catalog management operation (`Op_AdminCreateReport` and siblings in
  `reporting-api.ttl`) is gated by one global `reporting.catalog.manage` (System Admin only) —
  there's no way for a Staffing Administrator to manage Staffing's own report catalog entries
  independently, unlike Staffing's own email templates via Communications. This needs new
  domain-scoped manage permissions (`reporting.{domain}-reports.manage`, following the naming
  migration in item 4) AND new authorization checks on the admin API operations to actually honor
  them (currently they only check the one global permission) — do this alongside the draft/
  preview lifecycle work above, since "who can create/activate a report version" and "what
  versions/statuses exist" are the same design surface.
- **Corroborating evidence found 2026-09-04:** this isn't speculative — it's independently asked
  for by at least two FDDs. FDD 10 (Staffing) Feature 10.6, "Configure Staffing report
  availability and access," describes exactly this: define report availability (including
  retirement), define report access by authorized user roles, scoped to Staffing specifically.
  FDD 10's own review currently satisfies this via the existing global `rpt:Perm_CatalogManage`
  as a "good match" — worth revisiting once this spike lands, since that permission doesn't
  actually scope to one domain today. FDD 14 (Professional Learning) Feature 14.x describes the
  near-identical pattern verbatim ("define report availability and allow future/immediate
  retiring... define report access by authorized user roles..."). A placeholder permission,
  `rpt:Perm_StaffingReportsManage` (`reporting.staffing-reports.manage`), was migrated into
  `reporting-permissions.ttl` from a Phase-1 leftover (`staffing.admin.manage-reports`) as the
  anchor for this — it's not wired to anything yet, but it's now in the right place to be wired
  once the domain-scoped authorization design lands. Worth checking `proflearning-permissions.ttl`
  for whether an equivalent placeholder should be minted there too when this spike is scoped.
- **Related but distinct gap, also found 2026-09-04:** FDD 05 (Credentialing System Admin)
  describes something structurally different from PermissionKey-based report visibility —
  assigning *specific named reports* to a specific user or role, and later viewing that
  assignment list ("Assign worklist and/or report access to a user or role," "View a user's/
  role's assigned report access"). The FDD review's own note is explicit that
  `rpt:Perm_CredentialingReportsView` (the PermissionKey-based visibility permission from item 4)
  "models scope-based report visibility, not an assignable per-user/per-role list of specific
  named reports" — nothing in the current design models a many-to-many ReportAssignment concept
  at all. This is a separate, smaller design question from the manage-permission gap above (it's
  a new relationship/value-object question, not an authorization-model question) — worth scoping
  independently rather than assuming it falls out of the other two pieces of this spike for free.

## 4. Reporting permission-naming migration — completed 2026-09-04, noted for context
- **Status:** done
- **Source:** your own review of the Communications/Reporting permission-naming inconsistency

Not a new spike — recording here since item 3 above references it. Reporting's per-domain
report permissions were migrated from domain-first (e.g. `credentialing.reports.view`, owned by
each domain's own permissions file) to capability-first (e.g.
`reporting.credentialing-reports.view`, owned by `reporting-permissions.ttl`), matching
Communications' established shape. Final migrated set (across two passes — the first missed the
singular-"report" and differently-worded instances, caught on a follow-up sweep): credentialing,
documents, epp, payments, proflearning (all view-only), and staffing (a manage-verb placeholder,
see item 3's corroborating-evidence note above). Deliberately NOT migrated: `iam.reports.view`
and `profpractice.report.view`, which gate each domain's own native report-generation endpoints,
unrelated to this capability.

Ontology support for this went through two iterations: first a single `sd:appliesToDomain`
property, then split (same day, after review) into `sd:appliesToDomain` (who owns/defines the
permission — `rpt:ReportingContext` for all six of these) and `sd:concernsDomain` (whose resource
it actually gates — the domain-specific value). The single-property version made it look like
Staffing/Payments/etc. owned these permissions rather than Reporting. Full rationale and
migration notes live in `reporting-permissions.ttl`'s header, `CONVENTIONS.md`'s new
"`sd:permissionId` prefix order" section, and `PO-FEEDBACK.md`'s FDD 06 item 14.
