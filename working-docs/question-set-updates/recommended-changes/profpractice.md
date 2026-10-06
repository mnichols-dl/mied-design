# Recommended Changes: Professional Practices

**Impact: High.** PPR already contains its own question configuration and branching engine, and says it would hand it to a shared capability if one existed. Files are under `design/working-docs/solution-areas/profpractice/`.

## Summary

1. Retire `PPRQuestionConfiguration` as an owned aggregate. PPR consumes Question Sets for the annual PPR form and the self-disclosure form.
2. Keep everything that is PPR business logic: disclosures, account markers, clearance assessment, worklist, review statuses.
3. Use answer tags so a "Yes" can create a Disclosure without PPR owning question structure.
4. Remove or redirect `/ppr-questions` and `/disclosure-questions`.
5. Resolve the FDD 09 permission discrepancy about who manages PPR questions through scoped question set permissions.

## profpractice-domain.md

### Scope (line 25)

Move "Disclosure question configuration and branching logic" from "owns" to "does NOT own, owned by `questionsets`". PPR still owns which question sets apply to which cycle (annual PPR, self-disclosure) and what answers mean.

### PPRQuestionConfiguration (lines 460-495)

- Replace the aggregate with a short **consumption note**: PPR references question set ids for FormType values (Annual PPR, Self-Disclosure), requests responses from Question Sets, and reads answers through the API.
- Delete the note "may be owned by a forms/configuration capability" and the dependency note "Decision pending architecture review"; the decision is now recorded.
- Map the existing fields: ResponseType (YesNo, Date, FreeText, FileUpload, Dropdown) maps to Question Sets types (YesNo, Date, Text, FileUpload, SingleChoice). ValidationRules and BranchingLogic map to question validation and follow-up branches. EffectiveDate and IsActive map to version effective dates and derived state. Draft, Active and Inactive map to Draft, Published and Superseded.
- Open Question #7 (ownership of question configuration) becomes resolved by this change.

### ProfessionalPracticeResponse (lines ~125-165)

- `Questions` and `Answers` become a reference to a Question Set response (`response_id`, `question_set_id`, `version_id`) plus PPR's own fields (ResponseType PPR versus Self-Disclosure, dates). Stop storing question text inline.
- ERD: remove or deprecate `PPR_QUESTIONS`, `QUESTION_BRANCHING_RULES`, `PPR_QUESTION_CONFIGURATIONS` (lines ~764-790 and 527-528). `follow_up_answers` (line 623) and `question_key` (line 776) become references into the response.

### Disclosure creation

- Today a "Yes" answer creates a `Disclosure` aggregate (see "Submit PPR Response", `profpractice-sequences.md` lines 78-165). Define answer tags on the relevant question options (for example `creates-disclosure`) so PPR reacts to `QuestionSetResponseSubmitted` by fetching the response and creating one Disclosure per tagged answer, using that answer's follow-up detail answers. See sequence 7 in the Question Sets sequences.
- "Yes without details is blocked" and "unanswered mandatory questions are blocked" move to Question Sets validation (required follow-ups under the Yes answer).

### Things that do NOT change

- Account markers (`MandatoryHoldRequirement`, `EnhancedMonitoringStatus`, `ReReviewRequirement`) and their audit trail.
- `PPRClearanceAssessment` and its priority order, and `GET /educators/{id}/ppr-clearance`.
- The universal rule that PPR holds and blocks apply to every credential regardless of any question set. Document this as a boundary note: PPR blocking never depends on a question set outcome.
- "Requires PPR Response" is still computed from `LastPPRResponseDate`; PPR triggers the annual cycle by requesting a new response.

### Felony acknowledgment (lines ~242, 280; Open Question #20)

`RequiresFelonyAcknowledgment` needs a district acknowledgment capture that no sequence models today. Candidate: a small question set requested by Credentialing or PPR for the district contact. Keep the open question, add the option.

## profpractice-sequences.md

| Sequence | Lines | Change |
|---|---|---|
| Submit Self-Disclosure | 14-77 | Replace `GET /disclosure-questions` and local branching with Question Sets create-response, answer, submit; Disclosure creation driven by tags. |
| Submit Professional Practice Review Response | 78-165 | Same change for `GET /ppr-questions`; "Yes" detail forms and uploads are follow-up questions and File Upload questions in the set. Uploads go through Question Sets and Documents. |
| Configure PPR Worklist | n/a | No change. Worklists stay in PPR per the pattern. |
| Evaluate PPR Clearance for Credential Application | 549-616 | No change. |
| Manage Account Markers | 245-342 | No change. |

## profpractice-api.yml

- Lines ~444 and ~515: `/disclosure-questions` and `/ppr-questions`. Replace with a short note that questions come from Question Sets (or keep a thin read-through that returns the current version for a form type, to avoid breaking the UI during migration, then deprecate).
- `solution-api-catalog.md` lines 176 and 191 regenerate from the yml.
- Disclosure endpoints may return a `questionSetResponseId` link.

## profpractice-permissions.md

| Existing | Change |
|---|---|
| `profpractice.questions.view` | Replaced by `questionsets.profpractice-question-sets.view` |
| `profpractice.questions.manage` ("System Admin only") | Replaced by `questionsets.profpractice-question-sets.create`, `.edit`, `.publish`. The FDD says Credentialing Admin manages these; role assignment on the new permissions resolves the discrepancy (REV 09 Discrepancy #2) |
| `profpractice.disclosure.submit` | Stays. It is also the controlling permission in the Documents Upload Permission Model for PPR uploads through question sets |

## Data migration note

Existing configured PPR questions and their versions need to be imported as the first version(s) of PPR question sets, preserving question keys, with historical responses staying linked to the question text they were answered against. Whether a migration is needed depends on whether PPR has live data yet; if not, seed the question sets directly (OQ-QS-09).

## Tracking references

OQ-QS-02 (who authors), OQ-QS-09 (migration), felony acknowledgment (existing PPR Open Question #20). See [../tracking-proposals.md](../tracking-proposals.md).
