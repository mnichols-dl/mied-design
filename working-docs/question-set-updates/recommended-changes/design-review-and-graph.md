# Recommended Changes: Design Review Docs, Conventions, Graph, FDD Reviews

**Impact: Medium.** Mostly status updates and one conventions change. Files under `design/`.

## design/CONVENTIONS.md

- Line 113: `questionsets` / `qset:` is listed under stub-only bounded contexts ("no real source docs of their own"). Once the capability docs are accepted, move it to the first table (core and platform domains with real modeling) as `PlatformCapability`, display name "Question Sets".
- Add naming examples for the new class names so later passes stay consistent: aggregates `qset:QuestionSetAggregate`, `qset:OptionListAggregate`, `qset:PrefillSourceAggregate`, `qset:QuestionSetResponseAggregate`; states `qset:VersionState_Draft` style names following `<AggregateAbbrev>State_<StateName>` (for example `qset:ResponseState_Halted`); events `qset:QuestionSetVersionPublishedEvent`, `qset:QuestionSetResponseSubmittedEvent`, `qset:QuestionSetResponseHaltedEvent`, `qset:QuestionSetResponseSupersededEvent`; permissions `qset:Perm_...`, sequences `qset:Seq_...`.
- Permission ID order: add a worked example for a capability minting permissions on behalf of consuming domains (`questionsets.credentialing-question-sets.edit`), tagged `sd:appliesToDomain qset:QuestionSetContext` and `sd:concernsDomain cred:CredentialingContext`.

## design/graph (MiEdWorkforce.ttl)

Edit through the Solution Design Editor, not by hand. Recommended graph edits after the docs are accepted:

- Promote `qset:QuestionSetContext` from stub to a real `sd:BoundedContext` of kind PlatformCapability; remove `sd:status "stub"`. The existing stub has one GET operation and no API spec.
- Add the aggregates, events, permissions, API operations and sequences from the draft doc set. Add `sd:BusinessRule` nodes (`Rule_SingleCurrentVersion`, `Rule_ResponsesPinAVersion`, `Rule_HaltIsFormLevelOnly`, and so on) from the capability doc; the conventions note that business rules and annotations are underpopulated.
- Add `sd:Dependency` blank nodes from `qset` to `docs`, `orgs`, `refd`, `iam` and from `cred`, `profp`, `plrn` to `qset`.
- Remove or repoint the PPR question aggregate and operations (`PPRQuestionConfiguration`, `/ppr-questions`, `/disclosure-questions`) when PPR changes are accepted, and re-point proflearning's `questionsets` dependency to a real context.
- `bre:BusinessRuleEngineContext` stays a stub; do not use `qset` to stand in for it.
- Run the validation snippet in CONVENTIONS.md after the edit (parse, no untyped object-property targets).

## design/review/DESIGN-SPIKES.md

- **Item 2 (Question Sets):** mark as proposed resolved by this staging folder, subject to PO review. The spike asked for a combined POC with item 1 and for the integration, then document, then question order modeled as an explicit rule on the requirement. The Credentialing recommendations model that order as a requirement evaluation rule and keep the BRM decision separate. Note the departure from "combined with item 1" and why (Decision 2 in `01-classification-and-boundaries.md`).
- **Item 1 (Business Rule Engine):** leave open. Add a pointer that Question Sets now covers FDD 4.1 and that requirement evaluation is kept inside Credentialing until this item resolves.

## design/review/PO-FEEDBACK.md

- **Item 7 (rule change cutover):** the guardrail "rule changes never retroactively re-evaluate already-decided applications" is supported by pinning both the credential definition version and the question set version on the application and response. Add that to the item.
- **Item 9 (FDD 05 re-review):** recommend scheduling the re-review after PO review of this staging folder; the FDD 05 gap "no authoring surface for Question Sets" (Coverage Gap #9) is addressed by the capability.
- Add the POC walkthrough feedback (2026-09-23) as an item if the PO-FEEDBACK file is the intended home for it (the POC README currently cross-references hub tracking files).

## design/review/GAPS.md

- FDD 29 section (lines 8-35) and FDD 04 section (lines 93-102): update status from missing to partially addressed (4.1 by Question Sets; 4.2 still open).
- FDD 19/20 data quality section (lines 148-155): leave; the "same capability under two names" question stays attached to the BRM decision.

## design/working-docs/fdd-sdd-review

| File | Change |
|---|---|
| `reviews/05-system-admin-credentialing.md` L47 and L67 (Coverage Gap #9, priority H) | Resolve with "explicit shared platform capability" (one of the two options the review proposed). Note the FDD itself says decision-tree logic is defined in BRM (4), and the capability covers bounded follow-up branching only |
| `reviews/09-professional-practices.md` L31 (Discrepancy #2, Needs Decision) | Resolve through scoped permissions on the Question Sets capability |
| `reviews/13-25-epp.md` | Note FDD 13 Question Sets maps to the `epp` functional area when needed |
| `reviews/14-professional-learning-admin.md` L86 | Update from deferred to covered |
| `reviews/17-educator-temporary-credentials.md` Discrepancy #4 and Coverage Gap #1 | The two-stage permit flow (PPR flag at search, then "Needs Responses") is partly supported by consumer-requested responses; the missing "Awaiting PPR disclosure" application status is still open |
| `reviews/26-27-28-professional-learning.md` L6 | Update the `questionsets` deferral |
| `reviews/29-educator-credentialing-certificates.md` | Update question set references |
| `tag-index.md` L107, L214 | Retag |
| `client-questions.md` | Add candidate client questions: pre-payment review scope, PKS source, warn outcome, who authors structures, retention of disclosure responses (see tracking proposals) |
| `tracker.md` | No FDD 4 row exists. Add an FDD 4 row (4.1 covered by Question Sets, 4.2 open) |

## design/parity-check and generated-from-graph

No change now. After the graph is edited, `generated-from-graph/` regenerates and the parity check should be re-run for the affected domains (credentialing, profpractice, proflearning, and the new questionsets).
