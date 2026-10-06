# Recommended Changes: Communications

**Impact: Low.** Communications reacts to events; Question Sets does not send messages. Files under `design/working-docs/solution-areas/communications/`.

## Changes

1. **Do not add a direct Question Sets functional area for templates yet.** Question Sets events carry no answer text and are consumed by domains, which publish their own events for the communications that matter (for example "application returned to applicant", "application routed for review"). Communications' existing pattern is domain events in, templates scoped by functional area out.
2. **Upstream dependencies** (`communications-capability.md` L441-456): no new row needed unless a template is triggered directly by a Question Sets event. If so, add `QuestionSetResponseHalted` and `QuestionSetResponseSubmitted` with the variable-resolution endpoint on Question Sets (`GET /event-types/{eventType}/available-variables` pattern, communications-api.yml L339). Avoid putting answer values into template variables; disclosure content should never be emailed.
3. **Versioning precedent worth reusing.** Communications' "Single Active Template Version" rule (capability L507-521) and `/templates/{id}/versions/{versionId}/activate` and `/communications/preview` are the closest analogs to Question Sets' versioning and simulate. Keep terminology aligned where it does not conflict: Communications "activates" a version and Question Sets "publishes" with an effective date, because Question Sets needs scheduled versions. If the two should align, flag it in the standard verb list (`publish` versus `activate`).
4. **Dashboard alerts.** An applicant's "outstanding question sets" could surface as an alert or dashboard item. This is a Credentialing worklist-pattern concern; Communications `/alerts` can carry it if desired. Not required for the capability.
5. **Optional template content:** halt next steps text is authored in the question set, not in a Communications template. State that clearly so authors do not duplicate it.
