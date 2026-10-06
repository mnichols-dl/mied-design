# Recommended Changes: Credentialing

**Impact: High.** Credentialing is the primary consumer and also takes on the requirement-evaluation model the POC explored. Files are under `design/working-docs/solution-areas/credentialing/`. Line references are from the research pass on 2026-10-05 and should be rechecked when editing.

## Summary

1. Stop treating the "Business Rule Engine" as the source of question sets. Replace the two BRM "Get questions" calls with calls to the Question Sets capability.
2. Replace the generic `RequirementRule` (`rule_type` plus `rule_configuration`) with a typed requirement model that records the unmet outcome, the linked question set, reviewer type and review timing, and initial versus renewal applicability.
3. Replace `ApplicantResponses` and the string-only `businessRuleResponses` with references to Question Set responses, and pin both the credential definition version and the question set version on the application.
4. Add a requirement evaluation sequence and a requirement-level simulate capability.
5. Design review-before-payment as an explicit path.
6. Add the new PO-requested requirement types (PKS exam, progression, education history, SCECH renewal-only, exam passing grade).

## 1. credentialing-domain.md

### CredentialDefinition aggregate (lines ~78-115)

- Replace `RequirementRule` with **`Requirement`**, a typed entity. Keep `CREDENTIAL_REQUIREMENT_RULES` as the table but add explicit columns.

| Field | Notes |
|---|---|
| `requirement_type` | Enumerated catalog (see below). The API currently exposes `ruleType` as a free string with no enum anywhere; this closes that gap |
| `parameters` (json) | Type-specific thresholds and references, for example minimum years, minimum hours, test code and minimum passing score, minimum degree and optional major, required credential id |
| `applies_to` | `Initial`, `Renewal`, `Both`. PO feedback suggests separating the sets for management; model as two ordered lists on the definition (InitialRequirements, RenewalRequirements) with a shared requirement allowed in both |
| `unmet_outcome` | `Block`, `FollowUp`, `SpecialistReview`, and (decision pending) `Warn` with optional justification, since the FDD rule model has an Error and Warning severity |
| `question_set_id` | Optional. Reference to the question set, not a version; resolved to the current version when a response is requested |
| `reviewer_type` | Set when `unmet_outcome` is `SpecialistReview`. An enumerated specialty, not an admin-defined group (see worklist note below) |
| `review_timing` | `BeforePayment` or `AfterPayment` (default after). Pending PO answer |
| `rule_order` | Existing |

- **Requirement type catalog** (initial): Exam, Experience, ContinuingEducationHours (SCECH), ProgramCompletion, EducationHistory, PrerequisiteCredential (progression), CriminalHistory, Clearinghouse. PKS is an Exam with a different test code; confirm with the client whether it comes from the same results feed (open question). Each type registers a code-side **evaluator** that resolves data and returns met or unmet with a reason; the parameters are configuration.
- **Exam** gains `minimumScore`. The existing `EndorsementDefinition.RequiredAssessments` already carries `MinimumScore` and AND/OR combination logic; align the exam requirement with that so there is one pattern ("definition plus resolvable data plus fallback", DESIGN-SPIKES item 2).
- **PrerequisiteCredential (progression)** adds a dependency from one definition to another. New invariant: a definition referenced as a prerequisite by an active definition cannot be deprecated or deleted without first changing the dependents. This is the cascade the POs raised.
- Document explicitly the **code versus configuration boundary** from [00-poc-review.md](../00-poc-review.md): a new requirement type is a code change; thresholds, test names, and links to question sets are configuration.

### Requirement evaluation (new business rule)

Add a rule, "Requirement Evaluation and Unmet Outcome", describing:

1. Applicable requirements are those whose `applies_to` matches the application type (initial, renewal).
2. Each is evaluated against resolved data. Data resolution order is integration match, then uploaded document, then question set response, so the system never asks what it can already answer (DESIGN-SPIKES item 2, the user's POC direction).
3. Unmet plus `Block` stops the application at the requirement with guidance text.
4. Unmet plus `FollowUp` requires a submitted response for the linked question set, then the requirement is treated as resolved unless the response was halted or carries a tag that routes to review.
5. Unmet plus `SpecialistReview` requires review by the reviewer type. A linked question set is optional and collects the applicant's disclosure for the reviewer.
6. A halted question set response is not itself the application Blocked state (reserved for PPR). Default policy: the application stays in Draft with the halt noted on the requirement.
7. Universal rules (PPR clearance) are always checked and are not configurable per credential.

### CredentialApplication aggregate (lines ~285-331)

- Replace `ApplicantResponses` (line 294) and `APPLICATION_BUSINESS_RULE_RESPONSES` (lines 383-388) with `APPLICATION_QUESTION_SET_RESPONSES`: `application_id`, `question_set_id`, `response_id`, `requirement_id` (nullable, since one response can serve several requirements), `question_set_version_id` (copied from the response for reporting).
- Add `credential_definition_version_id` to `CREDENTIAL_APPLICATIONS` (lines 360-373). The existing rule "applications use the definition version active at the time of submission" has no supporting field today. Also fix "View Definition History" which references "associated applications using this version" with nothing to join on.
- Add `APPLICATION_REQUIREMENT_RESULTS`: `application_id`, `requirement_id`, `status` (Met, FollowUp, Blocked, SpecialistReview), `reason`, `evaluated_at`, `data_source`. This is the evaluation snapshot the reviewer sees. It also gives the "why are we asking this" inspectability from DESIGN-SPIKES item 2.
- The ERD stores `response_value` as json while the API accepts `additionalProperties: string`; both go away with this change.

### Application states and manual review

- Add `ManualReviewReason` value `ApplicantDisclosureReview` (or reuse `SpecialistReview` with the reviewer type attached; decide). Today there is no reason for a flagged question response. Also reconcile that `seq` and `domain` use `UnverifiedMentorAssignment` and `InsufficientIDPProgress`, which are not in the enum at line 70.
- Decide where review-before-payment sits (see section 4).
- State-name drift (`Awaiting Manual Review`, `On-Hold` in `seq`) should be cleaned up in the same edit.
- Keep Blocked for PPR only. Document that a halted response is not Blocked.

### Existing items that become consumers of Question Sets

- Mentor assignment on permits (lines 807-885) is an embedded form question with branches (staffing lookup, free-form entry setting `UnverifiedMentorAssignment`, no mentor hard stop). Model it as a question set with answer tags (`unverified-mentor`) and a halt for no mentor, and keep the staffing lookup as a consumer-side validation.
- Felony acknowledgment capture (lines 309, 579; `RequiresFelonyAcknowledgment`) has no sequence. Candidate question set owned jointly with PPR (profpractice Open Question #20).
- Electronic attestation capture (`credentialing-technical-design.md` lines 174-202) stays in Credentialing; it captures a legal attestation, not a form answer. Link it to the response where attestation text accompanies a question set.

### Versioning

- Align credential definition versioning with the derived-state approach (see technical design below).
- The POC allows multiple scheduled changes at once (for example 6/30 and 7/1). The existing invariant "effective ranges cannot overlap" is compatible with consecutive ranges; but Open Question #32 (limit on scheduled future dates) and the rule "Cleanup of Future-Dated Changes" (lines 673-681) need to be revisited so that queueing several is explicitly allowed.
- Add a diff between any two versions and a draft-versus-current compare to the "View Definition History" sequence.

### Worklist and reviewer types

- The POC's reviewer type per requirement conflicts with the working assumption in `solution-architecture.md` lines 300-318 (no admin-defined groupings or finer routing). Treat `reviewer_type` as an enumerated specialty mapped to a role or permission (not an admin-defined group), which stays inside the assumption, and record it as a flagged revisit trigger. The 9/29 note asks whether reviewer type should come from permissions or roles; recommend permission (so reviewer capability is not hardwired to a role name) with a role preview in the admin UI.
- Open Question #34 (Assigned-only scope on approve, deny and place hold) is the same routing question and should be answered together.

## 2. credentialing-technical-design.md

- Rewrite "Definition Version Activation" (lines 83-105): version state is derived from effective dates at read time. Keep the nightly job only for emitting events and invalidating caches at the boundary; correctness must not depend on it.
- Add **Requirement Evaluation Engine** section: evaluator registry, data resolution order, caching of resolved data per application, result snapshot. State plainly that this is not a general rules engine, in line with `configuration.md`.
- Add **Review Before Payment Path** (see section 4).
- Open Technical Questions #1 and #2 (configuration model, Global Parameters ownership): the requirement parameter model answers part of #1 (per-type parameters on the requirement). Leave the Global Parameters question open.
- Open Technical Question #7 (payment status check): recommend event subscription only (`PaymentCompleted`), which also fits the review-before-payment path.

## 3. credentialing-sequences.md

| Sequence | Lines | Change |
|---|---|---|
| Submit New Certificate Application | 14-100 | Replace `GET /eligibility-questions` to BRM (39-42) with: Credentialing evaluates requirements, collects distinct question set ids, calls `POST /responses` on Question Sets, applicant answers through Question Sets, `QuestionSetResponseSubmitted` returns. Keep the BRM call for rule validation at 61 until the BRM decision is made. Add the response references to the stored application. |
| Submit Permit Application | 104-206 | Replace `GET /permit-questions` to BRM (150-153) the same way. Staff answering on behalf of the educator pass `actorId`. Mentor question becomes a question set. |
| Issue Temporary Permit, Exceptional | 210-314 | The "attestations" at line 261 can use a question set response or stay as attestation capture; decide while the status banner on that flow is unresolved. |
| Add Endorsement to Certificate | 317-401 | Line 352 "Answer endorsement application questions" has no fetch step; add it. Several endorsements sharing a question set must reuse one response. Ties to open questions #19 and #20 (multiple endorsements, fee per endorsement). |
| Renew Credential | 404-494 | Line 440 "Answer renewal questions" has no fetch step; add it. Renewal applies only renewal requirements (SCECH, renewal limits). Use `supersede` when a prior response needs refreshing. |
| Application Auto-Approval | 496-583 | Final gate (542-545) should read requirement results and response status: any halted response, unresolved follow-up, or tag that routes to review prevents auto-approval. Add the explicit check; today the policy-exception trigger is implicit. |
| Application Manual Review and Approval | 585-659 | Reviewer sees requirement results and the Question Set response (questions as worded in the pinned version) beside the underlying record. Add the response fetch step. |
| Bulk Renewal Initiation | 792-850 | Bulk renewals evaluate BRM eligibility only. Educators who need renewal question sets must answer individually. Decide whether bulk renewals exclude anyone with an outstanding renewal question set. |
| Configure Credential Definition | 955-1041 | Add requirement editing with unmet outcome, linked question set picker, reviewer type, review timing, initial and renewal lists, and a **Simulate** step (see below). |
| Configure Endorsement Definition | 1045-1129 | Align assessment requirements with the exam requirement model. |
| View Definition History | 1354-1405 | Show requirement changes, compare any two versions, show applications pinned to each version. |
| **New:** Evaluate Requirements for an Application | n/a | The shared sequence the POC shows on the "Apply for credentials" page: data resolution, results per requirement, outstanding question sets. |
| **New:** Simulate Requirements | n/a | Admin picks a draft or current definition and a sample profile or a real target subject (read only). Shows per-requirement result, which question sets would fire, and which reviewer types would be routed. Persists nothing. |

Also add the endpoint gaps already noted: `GET /eligibility-questions`, `/permit-questions`, `/validate-eligibility`, `/renewal-requirements` and `POST /applications/{id}/on-hold` appear in sequences but not in the API.

## 4. Payment sequencing (explicit designed path)

Current model: payment moves Pending Payment to In Review; Credentialing Auto-Approval treats payment as the second gate after PPR; Payments allows "submit now, pay later". Nothing models review before payment, and the PO open question is unanswered.

Recommended designed path, pending PO confirmation:

1. If any unmet requirement has `review_timing = BeforePayment` and `SpecialistReview`, the application moves to a pre-payment review state (new state, for example `Pending Pre-Payment Review`; or `Requires Manual Review` with a `PrePaymentReview` flag; decide).
2. Credentialing does not make the payment link available while that review is open.
3. On review release, Credentialing moves the application to Pending Payment and makes the link available.
4. Payments is unchanged (see [payments.md](payments.md)); the precondition is on the link, which the FDD says is defined by Business Rule Management (FDD 07).
5. Requirements with `AfterPayment` follow the existing flow.

Out-of-state transcripts are the known before-payment case.

## 5. credentialing-api.yml

- `POST /applications` (lines ~124-128): remove `businessRuleResponses` (string map). Accept response references or have Credentialing record them from events.
- `GET /applications/{id}`: add `requirementResults`, `questionSetResponses` (ids, versions, status), `credentialDefinitionVersionId`.
- Credential definition schemas (`requirementRules` near line 2188): replace `ruleType` string and `ruleConfiguration` object with the typed requirement shape and `appliesTo`, `unmetOutcome`, `questionSetId`, `reviewerType`, `reviewTiming`.
- New: `POST /credential-definitions/{id}/simulate` (sample profile or target subject). Needs a `simulate` verb.
- New: `GET /applications/{id}/requirements` (evaluation results for the apply page).
- New: `GET /requirement-types` (catalog and parameter schemas for the admin UI).
- Mark `/eligibility-questions` and `/permit-questions` as removed in favor of the Question Sets capability.
- Add `ManualReviewReason` value(s) and `ApplicationStatus` changes.

## 6. credentialing-permissions.md

- No permission exists today to author question sets or requirement rules (rules are under `credential-definition.manage`). Authoring question sets moves to `questionsets.credentialing-question-sets.*`.
- Add `credentialing.credential-definition.simulate` (new verb) for requirement simulation. Simulating against a real subject reuses `questionsets.simulate-target-subject.use`.
- Reviewer type routing: define which permission grants membership in each specialist review type (for example a PPR review specialist), decide permission versus role (9/29 note), and confirm scope handling alongside `application.approve` Assigned-only.
- Reports: `credentialing.reports.view` already migrated to `reporting.credentialing-reports.view`; no new change.

## 7. UI-level feedback that lands in Credentialing docs

From the PO walkthroughs; these are applicant UI notes, not model changes:

- Apply page: show fully eligible credentials first; show per-requirement breakdown only in a per-credential detail view.
- A payment step as the last step of an application.

## Tracking references

See [../tracking-proposals.md](../tracking-proposals.md): OQ-QS-01 (BRM boundary), OQ-QS-03 (halted response), OQ-QS-10 (review before payment), OQ-QS-11 (PKS dataset source), OQ-QS-12 (warn outcome), OQ-QS-13 (bulk renewals and renewal questions).
