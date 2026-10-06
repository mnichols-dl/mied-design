# Question Set POC Review

Sources reviewed: `supporting-artifacts/question-set-poc/` (README, DISCUSSION-NOTES, `src/shared.jsx`, `AdminFlow.jsx`, `ApplicantFlow.jsx`, `ReviewerFlow.jsx`, `stateConfig.js`) and `design/working-docs/solution-level/features/feature-question-sets.md`.

## What the POC actually demonstrates

The POC bundles three different things that look like one:

1. **A form engine.** Questions of four types (yes/no, choice, text, file upload), conditional follow-ups under an answer, an answer that halts the form with a title, message and next steps, numbering like "2.1", option lists, profile prefill, text validation, versioned authoring with drafts, scheduled publish, compare, and a read-only simulate screen.
2. **A requirement evaluator.** A credential has typed requirements (exam, experience, SCECH hours, program, education, prerequisite credential, criminal history, clearinghouse). Each is checked against a mock profile, and each has an "if not met" outcome: block, initiate follow-up, or require specialized review.
3. **A review and routing flow.** Unmet requirements with a reviewer type route the application to a specialist queue; the reviewer sees the applicant's answers beside the underlying record; a "review before payment" timing flag exists.

The PO feedback ("liked the workflow and management concepts") covers all three. Only (1) is a good fit for a shared platform capability. (2) and (3) are credentialing business behavior that happens to use (1).

## Concept-by-concept sort

| POC concept | Where it lives in the POC | Belongs in | Notes |
|---|---|---|---|
| Question types, required flag, descriptions with lite markdown | `Q_TYPES`, `MarkdownText` | Question Sets | Add date and number/email/phone/url subtypes from the feature notes |
| Conditional follow-ups, numbering | `flattenVisibleQuestions`, `numberQuestions` | Question Sets | POC is one level deep; design is multi-level (feature notes, and the ACA tree is 5 levels) |
| Halt with title, message, next steps | `blockOn`, `blockTitle`, `blockMessage`, `blockNextSteps` | Question Sets | Rename "block" to "halt" so it is not confused with the application Blocked state |
| Option lists with a source label | `DATASETS` | Question Sets (definition and provider contract) | Rename to Option List. Providers are Organizations, Reference Data, EPP and so on |
| Prefill from profile | `prefillKey`, `PROFILE_FIELDS` | Question Sets (binding and registry) | Values come from the consumer or a registered provider, not hardcoded profile fields |
| Text validation (max length, regex presets, custom regex) | `validateTextAnswer` | Question Sets | Custom regex needs safety rules (see technical design) |
| Upload question with accepted types | `acceptedTypes` | Question Sets plus Documents | File storage, scanning and category rules belong to Documents |
| Versions, drafts, scheduled publish, compare, conflict warning | versioning engine in `shared.jsx` | Question Sets (for question sets) and Credentialing (for credential definitions) | Same pattern, two owners. Derived status avoids a "wake up" job |
| Simulate (sample persona, read only) | `QSSimulate` | Question Sets (form simulate) and Credentialing (requirement simulate) | Two different simulations, see below |
| Shared question set across credentials, asked once | `needed` memo in `ApplicantFlow` | Question Sets (reuse of a response within one consumer context) | Consumer asks for a response by context plus question set, capability returns the existing one |
| Credential requirement types and thresholds | `evalReq`, `REQUIREMENT_TYPES`, `REQ_DISPLAY` | Credentialing | The "code change vs UI config" boundary is the key design artifact, see below |
| Unmet outcome: block, follow-up, specialist review | `reqModeOf` | Credentialing | A halted question set is an input to this, not the same thing |
| Initial vs renewal requirement applicability | `appliesTo` | Credentialing | PO feedback suggests separate initial and renewal sets |
| Reviewer types and queue | `REVIEWER_TYPES`, `ReviewerFlow` | Credentialing and PPR (existing worklist pattern) | Hits the worklist working assumption, see Credentialing changes |
| Review timing before or after payment | `reviewTiming` | Credentialing, with Payments precondition | Open PO question, needs a designed path |
| Pinning the credential version on the application | `credentialVersions` | Credentialing | Already a recorded gap in the real design |
| Data sources status page | `DataSources` | Credentialing / integrations | Out of scope for Question Sets |
| Holders registry, reports, certificate generation | `HoldersList`, `Reports`, `DocumentDoc` | Credentialing and Reporting | Out of scope for Question Sets |

## What is code and what is configuration (the boundary the feature notes asked for)

The feature notes wanted to "point to what is a code change and what is a UI configurable thing". From the POC:

| Requirement type | Code change (evaluator logic) | UI configuration |
|---|---|---|
| Exam (MTTC, PKS) | The relationship lookup: does this person have a passing result for the named test | Which test, and (new from the 9/29 notes) minimum passing grade |
| Experience | Where years come from (placement records) | Minimum years |
| SCECH hours | Where hours come from (professional development registry) | Minimum hours; renewal only |
| Program completion | Lookup of completion record for an approved program | Which program type |
| Education history | Degree comparison logic and rank order | Minimum degree level, optional major |
| Prerequisite or progression | Held-credential lookup, plus the cascade check when a credential is removed | Which credential must be held |
| Criminal history, clearinghouse | Presence-of-record check against the source | Nothing beyond routing |
| New requirement type | Always a code change plus an evaluator registration | Once registered, appears in the Type dropdown |

Question sets themselves are fully configuration: no code change to add a question, a branch, or a halt message. A new question type or a new prefill source is a code change.

## POC gaps to fix before the POC is used as a design reference

1. **Responses are not pinned to a question set version.** Applications pin `credentialVersions` but store responses keyed only by question set id. A later edit to a question set changes how an old response reads. The design pins the version on the response.
2. **Two halt-like mechanisms.** A requirement can block outright (`blockIfMissing`), and an answer can halt (`blockOn`). The design names these differently and documents which one is which.
3. **Hardcoded special case.** `ApplicantFlow` checks `cur.id === "qs-criminal"`. Behavior that depends on a particular question set should come from data (a consumer tag on an answer), not an id check.
4. **Branches are one level deep by design.** The feature notes call for multi-level. The design allows multi-level with a depth cap.
5. **Question ids are mixed.** Seed questions use ids like `q1`; new ones use random ids. Diff and response mapping depend on stable keys, so the design requires a stable `question_key` that survives across versions.
6. **Custom regex failure is silent.** An invalid custom pattern is treated as no restriction. The design rejects invalid patterns at publish time.
7. **Hardcoded profile fields.** Prefill keys are a fixed list. The design uses a registry of prefill sources.
8. **No concurrency lock.** Optimistic only. The design keeps optimistic but requires an explicit acknowledgement to publish over a stale base.
9. **Compare is only against the live version.** Arbitrary version-to-version compare is listed as not covered. The design includes it.
10. **Everything is browser memory.** Expected for a POC, but it means the POC says nothing about response storage, document handling, event publication, or permissions. Those are covered in the capability draft.

## PO feedback and 9/22 and 9/29 notes mapped to an owner

| Feedback item | Owner |
|---|---|
| Simplify apply page, show eligible credentials first, details behind a click | Credentialing UI |
| PKS exam, possibly a type of the MTTC dataset | Credentialing (exam requirement subtype) plus Reference Data |
| Progression or precedence requirement, cascade on delete | Credentialing |
| Education history requirement | Credentialing |
| SCECH hours, renewal only | Credentialing (requirement applicability) |
| Separate initial and renewal requirement sets | Credentialing |
| Versioning and pending changes with effective dates, history, two pending changes (6/30 vs 7/1) | Credentialing for requirements; Question Sets for question sets |
| Simulate or test mode for question sets and requirements | Both (see capability draft and Credentialing changes) |
| Question-level validation (max length, regex) | Question Sets |
| Payment sequencing, review before payment | Credentialing with a Payments precondition |
| Exam and passing grade | Credentialing (exam requirement config) |
| Reviewer type by permission or by role, preview roles | Credentialing and IAM |
| Candidate tracking, STAR, districts report | Not Question Sets; note for Staffing and Reporting |
| Tie SCECH to Educator Evaluation and Qualifying | Professional Learning and Staffing, not Question Sets |
