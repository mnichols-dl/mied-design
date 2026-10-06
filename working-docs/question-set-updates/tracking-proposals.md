# Tracking Proposals (staging)

Staged 2026-10-05. Per the hub convention, open questions, decisions and backlog work belong in `hub/tracking/open-questions.md`, `hub/DECISIONS.md` and `hub/tracking/tasks/backlog.md`, not inside working docs. The drafts in this folder reference the IDs below (OQ-QS-nn) instead of carrying the content. When accepted, copy the entries into the hub files in their formats and replace the ID references with links.

Entry formats match the hub files: open questions as `| Date | Question | Who | Status |`, decisions as `## YYYY-MM-DD - Title` with bold labels, backlog as `- [ ]` items.

## Proposed open questions (hub/tracking/open-questions.md)

| ID | Date | Question | Who | Status |
|---|---|---|---|---|
| OQ-QS-01 | 2026-10-05 | Is Question Sets a platform capability (recommended) rather than a domain, and is requirement evaluation (typed requirements, unmet outcome, reviewer routing) kept inside Credentialing for now rather than folded into a shared Business Rule Management capability (FDD 4.2)? Staffing validations and EPP recommendation rules overlap in spirit. | Internal (design); MDE/CEPI POs for FDD 4.2 scope | Open |
| OQ-QS-02 | 2026-10-05 | Who authors question set structures? FDD 4.1.2 says admins maintain existing question sets and new structures are developer work, but the POC lets admins build new sets and branches freely and the POs liked it. Should creating a new question set and publishing be separate permissions held by different roles? | MDE/CEPI POs | Open |
| OQ-QS-03 | 2026-10-05 | When an applicant gives an answer that halts a question set, should the halted response be saved (so the applicant can return and the halt is on the record) or discarded? What does a halt mean for the application: stay in draft, or something visible to staff? | MDE/CEPI POs | Open |
| OQ-QS-04 | 2026-10-05 | Prefill: should the consuming domain supply prefill values when it requests a response (recommended, keeps the capability decoupled) or should Question Sets call a provider per source? Which prefill sources are in scope initially (name, email, unique id, experience years, SCECH hours, institution, degree, prior license state, exams)? | Internal (design) | Open |
| OQ-QS-05 | 2026-10-05 | Should applicants call the Question Sets runtime API directly (simpler, Self-only scope) or should every call be proxied through the consuming domain's API (single authorization point, as in the Documents pattern)? | Internal (design) | Open |
| OQ-QS-06 | 2026-10-05 | Event topic convention: a per-domain topic (`<domain>-events`, per solution-architecture.md) or the shared `miedworkforce-domain-events` used by Documents, Payments and Communications? Existing docs conflict. | Internal (design) | Open |
| OQ-QS-07 | 2026-10-05 | Documents categories for question set uploads: one category per owning domain (recommended, allows different retention) or one shared category? What happens to uploads on responses never submitted? | Internal (design); records retention owner if different | Open |
| OQ-QS-08 | 2026-10-05 | Retention, legal hold and field-level encryption for responses, especially criminal history and disciplinary disclosures. How long are submitted and abandoned responses kept? | MDE/CEPI POs and DTMB | Open |
| OQ-QS-09 | 2026-10-05 | Does PPR have live question configuration that must be migrated into Question Sets, or can PPR question sets be seeded fresh? | Internal (design); MDE/CEPI POs | Open |
| OQ-QS-10 | 2026-10-05 | See existing 2026-09-23 entry on review before payment. Proposed design to confirm: pre-payment review is withheld payment link availability, controlled per requirement, with out-of-state transcripts as the known case. Is the rule per requirement or per review type? | MDE/CEPI POs | Open (existing entry) |
| OQ-QS-11 | 2026-10-05 | Is the PKS exam sourced from the same results feed and test list as MTTC (a type of MTTC exam) or a separate source? Does the exam requirement also need a minimum passing score (9/29 note: exam and passing grade)? | MDE/CEPI POs | Open |
| OQ-QS-12 | 2026-10-05 | Should a requirement outcome include a Warning (user can bypass, optionally with a required justification) in addition to Block, Follow-up and Specialist Review? FDD 4.2 describes Error and Warning severities; the POC has no Warning. | MDE/CEPI POs | Open |
| OQ-QS-13 | 2026-10-05 | Bulk renewals evaluate eligibility only. For educators who also need to answer renewal question sets, do bulk renewals exclude them, or do they enter a "needs response" state? | MDE/CEPI POs | Open |
| OQ-QS-14 | 2026-10-05 | When a requirement is deleted or a credential definition is deprecated, and other definitions reference it as a prerequisite (progression), should the system block the change, warn, or cascade? | MDE/CEPI POs | Open |
| OQ-QS-15 | 2026-10-05 | Reviewer type: should a specialist review type be derived from permission or from role, and should the admin UI allow previewing what a role can see (9/29 note from Nadia and Courtney)? | MDE/CEPI POs; IAM design | Open |

## Proposed uncertainty-log entries (hub/tracking/uncertainty-log.md)

| Date | Concern | Why it matters | Next step |
|---|---|---|---|
| 2026-10-05 | Requirement evaluation stays in Credentialing by recommendation, but Staffing validations, EPP recommendation rules and FDD 4.2 all describe rule engines. If they converge we may rebuild the requirement model later. | Risk of rework if a shared Business Rule capability is chosen after Credentialing's typed requirement model is built | Keep the requirement model's evaluator registry narrow; revisit after DESIGN-SPIKES item 1 |
| 2026-10-05 | Reviewer type per requirement hits the "no admin-defined routing" working assumption in solution-architecture.md. | The worklist pattern was chosen to avoid a separate routing capability | Keep reviewer type as an enumerated specialty mapped to permission; confirm with POs |
| 2026-10-05 | Answer tags give consumers a way to react to answers without hardcoding question ids, but tags are opaque and could become a hidden rules language. | Tags are the one place logic could leak into configuration | Constrain to consumer-defined keys with documented meanings in the consumer's doc |

## Proposed decision (hub/DECISIONS.md)

```
## 2026-10-05 - Question Sets as a platform capability; requirement evaluation stays in Credentialing

**Status:** Proposed - pending PO review of the staged doc set (design/working-docs/question-set-updates).

**Decision:** Question Sets (form definitions, versions, branching, option lists, prefill, validation, uploads, responses, simulation) is a platform capability with identifier `questionsets` and prefix `qset:`. Requirement evaluation, unmet outcomes (block, follow-up, specialist review), reviewer routing and payment sequencing remain in the consuming domain, currently Credentialing. Professional Practices and Professional Learning consume the capability instead of owning question configuration.

**Why:** A question set is a means to ask questions for a business process, not a business process itself. The docs already assume this (Professional Learning treats it as cross-cutting; PPR defers ownership of question configuration to a shared forms capability). Keeping outcome logic in the consumer keeps business meaning and permissions where they already live and avoids building a general rules engine before the Business Rule Management scope (FDD 4.2) is decided.

**Alternative considered:** A Question Sets domain that also owns requirement evaluation and routing; or one combined Question Set plus Rules capability. Not pursued: it would compete with Credentialing for ownership of what an application requires.

**Open before this is fully settled:** Business Rule Management scope (DESIGN-SPIKES item 1); who authors structures (OQ-QS-02); PPR question configuration migration (OQ-QS-09).
```

## Proposed backlog items (hub/tracking/tasks/backlog.md)

```
## Question Set capability and Credentialing alignment (2026-10-05)

Follow-ups from the staged doc set in design/working-docs/question-set-updates.
Pick up after PO review of the classification decision.

- [ ] Review the staged capability doc set and confirm the platform capability
      classification and the requirement evaluation boundary (decisions 1 and 2).
- [ ] Merge the accepted capability docs into design/working-docs/solution-areas/
      questionsets/ and regenerate the capability and API catalogs.
- [ ] Apply credentialing changes: typed requirement model, response references,
      pinned definition and question set versions, requirement results,
      evaluate-requirements and simulate sequences.
- [ ] Apply profpractice changes: retire PPRQuestionConfiguration, answer tags
      for disclosure creation, permission mapping.
- [ ] Apply proflearning, documents, payments, iam, epp, organizations,
      communications, reporting, staffing changes from recommended-changes/.
- [ ] Update the graph through the Solution Design Editor and re-run the parity check.
- [ ] Update POC to pin question set version on responses, replace the
      qs-criminal id check with answer tags, support multi-level branching,
      and add a halt label separate from application Blocked.
- [ ] Add POC features still on the PO list: requirement-level simulate,
      PKS exam, progression requirement, education history, initial and renewal sets.
- [ ] Re-run the FDD 05 re-review after the capability lands (PO-FEEDBACK item 9).
```
