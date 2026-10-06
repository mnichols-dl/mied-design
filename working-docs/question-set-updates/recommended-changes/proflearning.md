# Recommended Changes: Professional Learning

**Impact: Medium.** Professional Learning already assumes the capability; the changes make the assumption concrete. Files under `design/working-docs/solution-areas/proflearning/`.

## proflearning-domain.md

- **Line 39** ("Question Set framework (cross-cutting capability used by multiple domains)" in out of scope): keep as out of scope but change the pointer to the now-defined `questionsets` capability and its doc set.
- **EvaluationTemplate (lines 227-250):** `QuestionReference` and `question_set_id` stay. Update the Design Note at line 247 ("question content, structure, and versioning is managed by the Question Set capability") to name the response model: an evaluation submission requests a Question Set response with consumer context `proflearning/evaluation/{id}` and the template stores only the question set id.
- **Pin the version.** The template references the question set by id. The response pins the version at creation, so an edit to the evaluation questions never alters past evaluations. State this explicitly; today the model stores responses as JSON keyed by question id (ERD lines 349-361, 469-470) with no version.
- **Responses (line 470):** replace "JSON keyed by question ID" with a reference to the Question Set response id, or keep a denormalized copy only if reporting needs it (decide; prefer reference).
- **Integration row (line 559):** replace "API call to Question Set capability" with the specific calls (create response, read response) and the events consumed (`QuestionSetResponseSubmitted`).
- **"Evaluation Required" toggle (lines 125-133, 661-671):** the SCECH award is gated on a submitted evaluation. The gate reads response status (Submitted) rather than a stored answer set.

## proflearning-sequences.md (lines 388-427, Admin Configures Evaluation Questions)

- Replace `GET /question-sets?domain=proflearning` with `GET /question-sets?owningDomain=proflearning` on the new API (parameter name differs).
- Line 416 ("Evaluation templates reference Question Set IDs, not inline question content") stays and is now backed by a real contract.
- Add the participant sequence for the learner answering an evaluation: create response, answer, submit, award SCECH on the event.

## proflearning-api.yml (lines 2117-2137, `questionSetRefs`)

Keep as references. Add `responseId` on evaluation submissions if the API returns it. Confirm the `GET /question-sets` reference points at the new capability's path.

## proflearning-permissions.md

- `proflearning.admin.questions` ("Manage evaluation question sets", CSV line ~207) is replaced by `questionsets.proflearning-question-sets.edit` and `.publish` (and `.create`, `.view`).
- `proflearning.admin.evaluationtemplates` stays: it configures templates, which is Professional Learning's business configuration.

## Other

- The FDD 14 review (`fdd-sdd-review/reviews/14-professional-learning-admin.md` line 86) deferred Question Sets, decision tree, validation severity and date-field conditions to a cross-cutting capability. Update the status from deferred to covered by `questionsets`, with decision tree meaning bounded follow-up branching (not free-form logic). Validation severity (Error versus Warning) is not in the capability draft; flag it to the BRM decision.
- The 9/22 note to tie SCECH to Educator Evaluation and Qualifying is not a Question Sets matter; leave for Professional Learning and Staffing.
