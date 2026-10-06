# PO-FEEDBACK.md — items needing a Product Owner decision

Tracks category 2 ("clarify with POs") and category 3 ("propose removal/change, POs confirm")
items surfaced while reviewing FDD outputs against the current MiEdWorkforce solution design. See
`fdd-review/CONVENTIONS.md` for how these were originally found; this doc is where they go once
they need a human decision rather than a graph edit.

**Important caveat on how to read this doc:** the MiEdWorkforce solution design (SDD) is our own
draft — it has NOT been reviewed or approved by the client or POs. Where an item below says the
SDD "already models" or "already decided" something, that means *our design work currently
assumes that*, based on the FDDs we've read and conversations with technical folks on the client
side — it is not a client-confirmed fact, and it is not evidence that the FDD's ask is wrong.
Treat every "the SDD already handles this" note as "here's what we drafted, does it hold up,"
never as "this is settled, no need to check." Some of the source FDD/comment files were also
pulled from a different repo than this one, so a few internal cross-references (a doc named as
the source of a decision, a file a note says a concurrence "was supposed to land in") may not
actually exist here — that's expected given the provenance, not necessarily evidence the
decision was never made; it just means we can't verify it from what's in this repo alone.

Each item: **Status** (open / answered / accepted / rejected), **Disposition** (question /
proposed-removal), **Raised**, source FDD(s), the ask, and what our current SDD draft actually
shows (so the PO isn't deciding blind). Update Status/PO answer in place once resolved — don't
create a new entry.

---

## FDD 01 — User Management

### 1. Impersonation / "View As" — proposed removal
- **Disposition:** proposed removal (confirm FDD 1.8 is dropped)
- **Status:** open
- **Raised:** 2026-09-04

FDD 1.8 asks for a read-only "View As"/"Impersonate" feature (render the target user's
Dashboard/Profile/in-progress applications, all Submit/Save/Delete disabled), with a
role-restricted access matrix on who can impersonate whom.

Our draft SDD's `iam-technical-design.ttl` (`iam:Question_UserImpersonationOmission`) records
that we evaluated a design matching this and chose to leave it out, reasoning that diagnostic
value could instead come from admin views of records/history/audit logs plus screenshare
tooling — with reintroduction criteria noted if that ever proves insufficient. FDD 1.8's ask is
materially the same feature. That's not proof the FDD is wrong — it's our own unapproved
design decision, based on FDD material we'd read at the time, that happens to conflict with what
this FDD is separately asking for.

**Recommendation:** bring this to POs as a proposed removal rather than silently keeping our
exclusion — lay out both the FDD's ask and our reasoning for leaving it out, and let them decide
whether the admin-view/audit-log/screenshare alternative is actually sufficient for whatever
workflow prompted the original "View As" request. If POs want the feature, that's new
information overriding our draft, not a contradiction to explain away.

**If confirmed dropped:** clean up in the graph (category 1, no further PO input needed) —
`iam:Perm_UserImpersonate` is still cataloged as a Permission in `iam-permissions.ttl` despite
having no API operation; either delete it or mark it explicitly `status: "rejected"` so it stops
reading as a half-built feature, and convert the FDD 01 review's
`Question_ImpersonationFeatureContradictsOmission` Annotation to `findingType: "rejected"`.

### 2. User Groups vs. Role + Scope — proposed removal
- **Disposition:** proposed removal (confirm Group layer is dropped)
- **Status:** open
- **Raised:** 2026-09-04

FDD Feature 1.3 asks for a full admin CRUD feature for "User Groups" — a first-class,
separately-assignable, admin-managed construct distinct from Role.

Our draft SDD's `iam:RoleDefinitionAggregate` carries a note recording our own 2026-08-14 design
decision not to add a separate User Group layer — reasoning that an earlier FDD draft's Group
concept had exactly one enforced constraint (a user's groups must all share one MiLogin type),
and that constraint "never carried behavior that Role + Scope couldn't already express." Nothing
in FDD 1.3/1.4/1.5's acceptance criteria describes ad-hoc grouping or bulk-action targeting
independent of Role — Groups there are described purely as a categorization layer for access
control, the same purpose Role already serves in our draft. This looks like terminology overlap
rather than a missing capability, matching your instinct ("I believe this is all synonymous with
roles, as long as they can see users with a given role, is that enough?") — but that's our
reading, not a confirmed fact, and it's exactly the kind of judgment call that needs a PO's eyes
before it's final.

**Recommendation:** bring this to POs as a proposed removal — present the Role+Scope-only design
and ask directly whether "browse/filter users by Role" satisfies what FDD 1.3 was actually
after, or whether there's a real distinct use case for Groups we're not seeing.

**Note on cross-references:** our design note says this was "pending final client concurrence,"
citing a `fdd-sdd-review/client-questions.md` file that doesn't exist in this repo. Since these
FDD/comment source materials were pulled in from a different repo, this is most likely a pointer
to a document that lives elsewhere (or hasn't been carried over yet) rather than evidence the
concurrence never happened — worth asking whoever has access to the original source repo before
assuming this question was never actually put to the client.

**If confirmed dropped:** category 1 cleanup — reject Feature 1.3's Group CRUD stories in the
FDD 01 review file; no ontology change needed since Group was never added as a class in our
draft.

### 3. Transitive permission inheritance (ISD > District > Building) — question (low priority)
- **Disposition:** question (informational confirmation)
- **Status:** open
- **Raised:** 2026-09-04

Our draft SDD models this as a deliberate, explicit design choice, not just an implementation
detail — `iam:Rule_TransitiveAuthorizationResolution`, backed by a named ubiquitous-language term
and a technical process with a worked example. In our draft, individual scope does not
participate in the hierarchy, and permissions don't inherit upward or across organization types.
That's a real, considered position we took — but it was our call, made without explicit client
sign-off on whether ISD-level admins having automatic visibility into constituent
Districts/Buildings (without a separate approval step per org) is actually the behavior the
client wants.

**Recommendation:** low-effort confirmation with POs, not a design overhaul — describe the
transitive model plainly and ask whether it matches their expectations, since it changes who can
see what without an explicit grant at every level.

---

## FDD 02 — Reporting

### 4. "Group" scoping in report permissions — question
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

FDD 2.1.1's AC requires authorization enforcement "at the entity, group, and role level," but our
draft `permission-scopes.ttl` has no `Group` scope — only SystemWide/ISD/District/Entity/
SelfOnly/AssignedOnly. Same terminology question as FDD 01's Group item above.

**Recommendation:** fold into the FDD 01 Group/Role question above rather than treating as a
separate PO ask — once that's resolved, this AC's wording either maps onto existing Entity/
District/ISD scopes or surfaces a real gap we don't have yet.

### 5. Legacy batch-job → domain-model mapping (see also FDD 05 batch jobs) — question
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

Not originally in your list, but surfaced while investigating FDD 05's batch jobs below —
noting here since it may also touch Reporting's own "Calculate Dashboard data" legacy job. See
FDD 05 item 8.

---

## FDD 03 — Dashboards

### 6. Credential-Admin widget catalog — circular ownership between FDD 3.6 and FDD 11.7
- **Disposition:** question (document ownership, not a design gap)
- **Status:** open
- **Raised:** 2026-09-04

No ontology-level "functional area" concept is driving this — there's no `sd:FunctionalArea`
class anywhere in our draft ontology, and each dashboard widget in our draft already just pulls
a summary/count from its own owning domain independently. Your instinct — that this is a UI
pattern rather than a shared domain concept — matches what our draft currently does, though
that's a design choice we made, not something the FDDs themselves required us to do this way.

The actual issue is in the FDD material itself, not our SDD: FDD 3.6 (Dashboards) defers the
Credential-Admin widget catalog's content to FDD 11.7 (Educator Credentialing), while FDD 11.7
independently defers back to FDD 3 — neither source document actually specifies the widget
catalog, each just names the other as the source of truth.

**Recommendation:** ask AO/POs which document is meant to be authoritative for the
Credential-Admin dashboard's widget list. Our lean (Dashboards owns catalog *structure*,
Credentialing supplies the *data* each widget shows) matches the pattern our draft already uses
for every other per-role dashboard feature, but that's a proposal to confirm, not a decision
already made.

---

## FDD 04 — Business Rule Management

### 7. Rule-change / historical-data cutover risk — question, with a recommended guardrail
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

Real and only half-addressed within our own draft, let alone with the client. Our draft's
Credential/Endorsement *definition* versioning rule (`Rule_CredentialDefinitionVersioning`:
in-progress applications use the definition version active at submission, historical versions
stay queryable) is a considered design — but it's a different concern from the Business Rule
Engine's *validation* rules, which is what FDD 04 is actually about. The FDD 04 comment thread
itself (a reply from Doherty relaying "Yashi," i.e. client-side input, not our own design note)
says a rule change applies to new transactions going forward and won't touch pending
applications, but explicitly leaves the "Day 1" baseline-rule strategy at system launch
undefined, and references an undocumented "Standardized System Live Date" concept.

**Recommendation:** this is a genuine open item from the client's own comment thread, not
something our draft has an answer for. Propose a guardrail rather than an open-ended question —
*"Rule changes never retroactively re-evaluate already-decided applications — only the Day 1
baseline and go-forward changes need a defined cutover strategy"* — to head off the scope-creep
risk you flagged, and separately ask what "Standardized System Live Date" refers to (it may be
defined in a doc that didn't make it into this repo).

---

## FDD 05 — System Admin (Credentialing)

### 8. Batch job legacy-system mapping — question
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

The 2 batch jobs in our draft (`Definition Version Activation`, `Credential/Permit Expiration
Processing`) are specific and tied to named business rules with real query logic and downstream
event consumers — so "generic" probably isn't about how they're written. More likely source of
that reaction: FDD 05's own wireframe evidence shows an existing "Run & View Batch Jobs" admin
console with 8+ differently-named legacy jobs (Process CEPAS posting, Delete TPI Endorsement,
Calculate Dashboard data, etc.). Our draft never attempted to map those onto the 2 jobs we
modeled — the FDD review file only notes "closest conceptual overlaps," which is our own
hedge, not a confirmed mapping.

**Recommendation:** ask POs/AO to confirm whether the legacy console's 8+ jobs are (a) fully
superseded by the 2 jobs in our draft, (b) still-needed jobs we haven't modeled at all yet, or
(c) redundant/deprecated. Don't model more batch jobs speculatively until this is answered —
right now under-modeling (missing real legacy jobs we don't know about) is as much a risk as any
confusion about what's already there.

### 9. Recommend scheduling a re-review pass on FDD 05
- **Disposition:** process recommendation, not a question
- **Status:** open
- **Raised:** 2026-09-04

The FDD 05 review file is self-flagged as a partial pass on our end: Feature 5.5 is an unfinished
stub in the source FDD itself ("worklist/role assignment... not actually modeled end-to-end
anywhere" in our draft), Feature 5.6 is a pure deferral pointer, a chunk of narrative content
(Alternative Test Scores, Public Search admin, Alerts, Email templates, Question Sets) was never
formalized into requirements at all on our side, only 3 of 15 "redesign" wireframes were
individually viewed, and three cross-cutting questions remain open in our own review notes. Your
instinct to schedule another pass is well-founded — recommend doing it after the Business Rule
Management / Question Set design spike (see `DESIGN-SPIKES.md` item 2) lands, since several of
FDD 05's open threads depend on it.

---

## FDD 06 — Alerts, Emails, Communications

### 10. Email template format: Markdown vs. HTML — question
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

This is already a logged, open contradiction in our draft, not something we invented just now.
Our draft's `comm:TP_TemplateStorageAndEditing` deliberately chose Markdown with a narrow,
server-controlled extension allowlist (an `:::alert{...}` block, a `:::button{...}` block) over
raw HTML or a WYSIWYG editor — the design note says a WYSIWYG-with-sanitization approach was
considered and rejected for XSS complexity and noisy version diffs, and explicitly disallows raw
HTML blocks, custom CSS/JS, and iframes. FDD 6.3.2 asks for the opposite: a rich text/HTML editor
with image insertion (for MDE/CEPI letterheads) and font-selection UI enforcing ADA compliance.

**Recommendation:** this is a real tradeoff, not a case where one side is obviously right — our
Markdown choice buys XSS safety and clean diffs but can't do letterhead images or arbitrary
formatting; the FDD's ask buys richer formatting but reopens the security/versioning concerns our
draft was trying to avoid. Bring both options to POs plainly rather than defaulting to either:
if letterhead/logo images are a hard requirement, a scoped middle ground (e.g. Markdown plus a
vetted, non-arbitrary "insert approved letterhead image" block, still disallowing free-form HTML)
may satisfy both without reopening the XSS surface — but that's a proposal to validate with them,
not a decision to make unilaterally.

### 11. Drop the Outlook/Graph API email channel? — proposed removal
- **Disposition:** proposed removal (confirm the SendGrid-only channel is sufficient)
- **Status:** open
- **Raised:** 2026-09-04

A separate, wholly distinct Outlook channel (Microsoft Graph API `sendMail` from dedicated
mailboxes, for two-way correspondence) is described in a companion interface-design document but
has no counterpart anywhere in our draft's communications capability/API/sequences — our draft
models exactly one outbound channel, SendGrid, one-way only. This is also in apparent tension
with the primary FDD's own Business Specifications text, which says the system will let users
view sent email via an embedded viewer "WITHOUT requiring a direct or two-way integration with a
Microsoft Outlook client" — i.e., the source material may be internally inconsistent on this
point, not just inconsistent with our draft.

Our draft does have a real audit trail that could plausibly stand in for at least part of what
Outlook would provide: `comm:EmailInstanceAggregate` keeps a full historical record of sent
emails with delivery-event tracking (Processed/Delivered/Bounced/Deferred/Dropped/Failed) and
resend tracking. That's a one-way send log, though — it doesn't give two-way correspondence the
way a real Outlook mailbox would, so it substitutes for *audit history*, not for *two-way
messaging* if that turns out to be a genuine requirement.

**Recommendation:** ask POs directly whether the Outlook/Graph interface document reflects a
real, still-wanted requirement, or whether it's a superseded draft that the primary FDD's own
"no two-way Outlook integration needed" line already overrides. Frame it as: "our design uses
SendGrid only, with a full send/delivery audit trail — is that sufficient, or is there a real
two-way-correspondence need the audit trail doesn't cover?"

### 12. Two-way comms / shared inbox — confirmed out of scope, no action needed
- **Disposition:** informational (question resolved, no PO input needed)
- **Status:** resolved
- **Raised:** 2026-09-04

Checked directly: "shared inbox" doesn't appear anywhere in the FDD 06 source material or our
domain model, and "two-way" only shows up in the Outlook/Graph tension already covered in item 11
above — it's never framed as a shared-inbox/support-ticketing concept. The FDD's own Business
Specifications text explicitly disclaims needing two-way Outlook integration. Your instinct was
right: this looks like it was never actually asked for, not something we dropped or missed.

### 13. Alerts: definition/authoring lifecycle is underspecified in the FDD itself — question
- **Disposition:** question
- **Status:** open
- **Raised:** 2026-09-04

The ambiguity here originates in the FDD text, not just our modeling of it. FDD 06's Business
Specifications describe admin-configurable alert *definitions/templates* — create, modify, remove
with versioning control, grouped by domain (Staffing/Credentialing/EPP/Professional Learning),
with title/message/description/coded-inserts/start-end dates/on-demand-or-scheduled — but this
was never promoted into a numbered Addendum story with real acceptance criteria the way Email
Templates were (Feature 6.3). Our draft's `comm:AlertAggregate` only models the *instance* side
(one alert shown to one user, with Draft/Pending/Active/Resolved/Expired states) — list/create/
resolve operations exist, but nothing for searching, updating, deleting, or versioning alert
*definitions*.

**Recommendation:** rather than guessing at the missing definition-side API, ask POs/AO directly
what "dynamically created and managed" alerts should look like in practice — is this meant to
mirror Email Templates' admin-authoring workflow closely enough to reuse that pattern, or is it a
simpler, more constrained set of alert types than free-form template authoring implies? This
likely needs the FDD's own Business Specifications elaborated into real numbered stories before
it's modelable at all, which is really a request back to whoever owns the FDD material rather
than a pure PO question.

### 14. Permission-naming pattern: Communications vs. Reporting — resolved, Reporting migrated
- **Disposition:** informational (your own question — resolved with a real design decision and
  a completed migration)
- **Status:** resolved
- **Raised:** 2026-09-04

You asked whether Communications' template permissions (`communications.staffing-templates.manage`
-shape) are modeled consistently with Reporting's `{domain}.reports.view` pattern. Initial answer
(explaining it as two different actor models — domain users consuming vs. a central team
authoring) was wrong: you pointed out the actor model is actually the same in both cases (a
domain's own admin manages that domain's reports and templates alike), and the real explanation
was simpler — Communications got a full solutioning pass (90 permissions, 5 verbs × 8 domains,
consistently named) while Reporting only got a partial one (7 permissions, view-only, 4 of 9
domains, one of which — profpractice — was already internally self-contradicted).

**Decision made:** bring Reporting's naming up to Communications' capability-first shape. The
stronger argument for this (beyond matching actor models) turned out to be collocation:
Reporting's own `ReportDefinitionAggregate` operations all live in `reporting-api.ttl`, so a
permission gating them belongs in `reporting-permissions.ttl` too — otherwise the file that
defines an operation and the file that defines its required permission are different files for
no reason.

**Completed:**
- Migrated `credentialing.reports.view`, `documents.reports.view`, and `epp.reports.view` into
  `reporting-permissions.ttl` as `reporting.credentialing-reports.view`,
  `reporting.documents-reports.view`, `reporting.epp-reports.view` — each tagged with the new
  `sd:appliesToDomain` property pointing at its domain's `sd:BoundedContext`.
- Added `sd:appliesToDomain` to the ontology (`sd:Permission` → `sd:BoundedContext`) so
  "every permission relevant to domain X" across capabilities is a real query, not a guess from
  the permissionId string — this was the structural fix for the one real discoverability cost of
  capability-first naming.
- Added an explicit rule to `CONVENTIONS.md` (new "`sd:permissionId` prefix order" section)
  documenting both valid shapes and when each applies, so this doesn't have to be rediscovered
  for the next cross-cutting capability.
- Deliberately left `iam.reports.view` and `profpractice.report.view` untouched — confirmed these
  gate each domain's own *native* report-generation endpoints (IAM's New-Users/Denied-Requests/
  Inactivity reports; ProfPractice's PPR report data), unrelated to the cross-cutting Reporting/
  Power BI capability entirely. Renaming them would have incorrectly implied they were part of
  this migration.
- Whole-tree parse validated after the change (28,332 quads, no parse failures).

**Deliberately NOT done as part of this pass:** giving Reporting the same create/edit/activate/
delete verb richness Communications has. Today, ALL report catalog management (create/edit/
activate/deactivate) is gated by a single global `reporting.catalog.manage` (System Admin only) —
there's no way yet for a Staffing Administrator to manage Staffing's own report catalog entries
the way they can manage Staffing's own email templates. Bringing that to full parity needs new
authorization checks on the admin API operations themselves, not just new permission instances —
that's real design work, tracked as an expansion to `DESIGN-SPIKES.md` item 3 rather than done
here.

**Correction, 2026-09-04 (same day, later pass):** you asked whether we'd missed anything, and we
had — twice. The first migration only searched for the plural pattern (`\.reports\.view`), which
missed `payments.report.view` and `proflearning.report.view` (singular "report") — both real,
unwired, self-documented-as-belonging-to-Reporting gaps, now migrated the same way as the first
three. It also missed `staffing.admin.manage-reports`, a differently-worded Phase-1 leftover that
turned out to be the more interesting find: it's a **manage** permission, not a view permission,
and FDD 10's own review already links its underlying story (Feature 10.6, "Configure Staffing
report availability and access") to the existing *global* `reporting.catalog.manage` as a "good
match" — but that's System-Admin-only and covers every domain's reports, not a Staffing-scoped
delegation. FDD 14 (Professional Learning) independently describes the identical acceptance
criteria pattern ("define report availability... define report access by authorized user
roles..."), which corroborates this is a real, recurring ask, not a one-off. Renamed rather than
deleted — kept as `reporting.staffing-reports.manage`, an explicit placeholder anchor for the
still-missing per-domain delegation capability, folded into `DESIGN-SPIKES.md` item 3.

**Lesson for future sweeps like this:** when hunting for every instance of a naming pattern
across ~10 domain files, grep for the *concept* (e.g. `report` without an `s`, or a synonym like
`admin.manage-X`), not just the one exact string you already know about — a plural-only search
silently missed a third of the real matches here.

**Second correction, same day:** you also flagged that a single `sd:appliesToDomain` pointing at
the "about" domain (e.g. `staff:StaffingContext` on a Reporting-owned permission) made it look
like Staffing owned that permission rather than Reporting. Split into two properties — kept
`sd:appliesToDomain` for who owns/defines the permission (now `rpt:ReportingContext` on all six
migrated permissions) and added `sd:concernsDomain` for whose resource it actually gates (the
domain-specific value, e.g. `staff:StaffingContext`). See `CONVENTIONS.md`'s updated
"`sd:permissionId` prefix order" section and `reporting-permissions.ttl`'s header for the full
model.
