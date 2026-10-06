# Question Sets - Permissions Catalog

**Domain:** Question Sets (Platform Capability)  
**Version:** 0.1 (draft)  
**Last Updated:** 2026-10-05

## Naming

Permissions follow `{capability}.{functional-area}-{resource}.{verb}`, the capability-first shape already used by Communications (`communications.{fa}-templates.*`) and Reporting (`reporting.{domain}-reports.view`), since the operations are owned by Question Sets but the content belongs to the consuming domain. Per `design/CONVENTIONS.md`, tag each with `sd:appliesToDomain` pointing at `qset:QuestionSetContext` and `sd:concernsDomain` pointing at the owning domain.

Functional areas initially: `credentialing`, `profpractice`, `proflearning`, `epp`. When a new functional area is added, every permission in the applicable pattern must be provisioned, as Communications requires.

New verbs needed in the template verb list: `publish` and `simulate`. Documents (`deprecate`, `apply`) and Communications (`activate`) already extend the list the same way.

## Authoring Pattern (per functional area)

Shown for `{fa}`. Provision for each functional area.

| Category | Permission ID | Description | Applicable Scopes | Notes |
|---|---|---|---|---|
| Question Set Authoring | `questionsets.{fa}-question-sets.view` | View question sets, versions, history and comparisons for the functional area | System-wide | Read only |
| Question Set Authoring | `questionsets.{fa}-question-sets.create` | Create a question set and its first draft | System-wide | Creating new structures can be held by fewer roles than editing, to match the FDD 4.1.2 split between maintaining existing sets and building new ones |
| Question Set Authoring | `questionsets.{fa}-question-sets.edit` | Create drafts from existing versions, edit and discard drafts, cancel scheduled versions | System-wide | |
| Question Set Authoring | `questionsets.{fa}-question-sets.publish` | Publish a draft with an effective date, including acknowledging a conflict | System-wide | Separate from edit so authors can be reviewed before go-live |
| Question Set Authoring | `questionsets.{fa}-question-sets.simulate` | Run a version against a sample subject | System-wide | |
| Question Set Authoring | `questionsets.{fa}-question-sets.delete` | Delete a question set that has never been published or used | System-wide | Not available once any response exists |
| Responses | `questionsets.{fa}-responses.view` | View submitted responses for the functional area | System-wide, Entity, Worklist-specific | Reviewers read through the consuming domain's screens; the scope follows the consuming domain's review scope |

## Cross-Cutting Permissions

| Category | Permission ID | Description | Applicable Scopes | Notes |
|---|---|---|---|---|
| Option Lists | `questionsets.option-lists.view` | View option lists and provider bindings | System-wide | |
| Option Lists | `questionsets.option-lists.manage` | Create and edit static option lists and provider bindings | System-wide | Provider registration itself is a deployment change, not a permission |
| Prefill Sources | `questionsets.prefill-sources.view` | View the prefill source registry | System-wide | Registry changes are code or change-controlled configuration |
| Simulation | `questionsets.simulate-target-subject.use` | Simulate against a real subject's data (read only) | System-wide | Audited. Needs the consuming domain's data access as well |
| Responses | `questionsets.response.complete` | Answer, save and submit one's own response, and request file uploads for it | Self-only | Runtime applicant permission. Additional on-behalf entry by staff uses `questionsets.{fa}-responses.complete-on-behalf` if needed (open item) |

## Not Permissioned Here

- Creating a response request: internal service call by the consuming domain over Managed Identity; the consumer checks its own permission (for example `credentialing.application.submit`) first, per the Documents pattern of a single authorization check in the domain API.
- Reading a response through a consuming domain's screen: gated by the consuming domain's own permission (for example `credentialing.application.view`, `profpractice.disclosure.search`) plus the `{fa}-responses.view` scope check if the capability API is called directly.

## Mapping From Existing Permissions

| Existing | Replaced or related by |
|---|---|
| `profpractice.questions.view` | `questionsets.profpractice-question-sets.view` |
| `profpractice.questions.manage` | `questionsets.profpractice-question-sets.create`, `.edit`, `.publish` |
| `proflearning.admin.questions` | `questionsets.proflearning-question-sets.edit` and `.publish` |
| `proflearning.admin.evaluationtemplates` | Stays in Professional Learning (template configuration), not a question set permission |
| `staffing.admin.manage-validations` | Unchanged. Staffing validation rules are not question sets |
| `credentialing.credential-definition.manage` | Unchanged for requirement rules. Authoring question sets uses `questionsets.credentialing-question-sets.*` |
| `credentialing.application.bypass-validation` | Unchanged |

The FDD 09 discrepancy over who manages PPR questions (Credentialing Admin in the FDD, System Admin in the SDD) becomes a role assignment decision on `questionsets.profpractice-question-sets.*` rather than a modeling difference.

## Scope Definitions

| Scope | Meaning here |
|---|---|
| System-wide | All question sets in the functional area |
| Entity | Responses for a specific consumer context the user is allowed to see (for example an EPP's own applicants) |
| Worklist-specific | Responses attached to items in a worklist the user can see (follows the consuming domain's existing filter) |
| Self-only | The user's own responses |

## CSV Consolidation

When approved, add rows to `solution-level/solution-permissions.csv` with `Source` set to `questionsets-permissions.md`. Expected count: 7 permissions times 4 functional areas, plus 5 cross-cutting, which is 33 rows. Confirm whether to enumerate every functional area or document the pattern once, as Communications does.

---

**Version History:**
- v0.1 (2026-10-05): Initial draft
