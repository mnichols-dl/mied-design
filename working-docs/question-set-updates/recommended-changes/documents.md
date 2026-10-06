# Recommended Changes: Documents

**Impact: Medium.** No change to the Documents model; additions to categories and the upload permission table. Files under `design/working-docs/solution-areas/documents/`.

## Changes

1. **New document category or categories** for question set response attachments. Documents' category model carries slug, allowable formats, file size limit and retention. Recommended: one category per owning domain so retention can differ (for example `qs-response-credentialing`, `qs-response-profpractice`, `qs-response-proflearning`), rather than a single generic category. Decide (OQ-QS-07). Question Sets stores the category slug on a File Upload question; accepted types and size come from the category, and the question may only narrow them.
2. **New attachment type** `question-set-response`, with `attachment_id` set to the response id (or `{responseId}/{questionKey}` if per-question attachment tracking is wanted). Update the Document Attachment definition and the attachment indexing note (`documents-capability.md` ~L216 and ~L644).
3. **Upload Permission Model** (`documents-permissions.md` L47-69): add rows. The controlling permission is the consuming domain's, not a Documents permission.

| Context | Controlling permission | Domain |
|---|---|---|
| Question set response upload for a credential application | `credentialing.application.submit` (applicant), plus the staff on-behalf permission where applicable | credentialing |
| Question set response upload for a PPR or self-disclosure response | `profpractice.disclosure.submit` | profpractice |
| Question set response upload for a program evaluation | The proflearning evaluation submit permission (confirm name) | proflearning |

4. **Sequences** (`documents-sequences.md`, Integration Pattern: Domain API Calling Documents, L16-62): add Question Sets as an example caller. It is the Domain API in the pattern: it checks the response is in progress, the question is visible and the caller is the subject, then calls `POST /api/documents/upload/request`.
5. **Dependencies** (`documents-capability.md` Downstream table, ~L424-436): add Question Sets next to credentialing, proflearning, staffing, epp.
6. **Existing notes to keep in mind:** the "does NOT own" list says business rules about which documents are required belong to the domain. The question set owns "this question requires a file". Documents does not own upload UI components.

## Open points (tracked)

- Draft and abandoned uploads tied to a never-submitted response: what retention applies and who cleans up (OQ-QS-07, OQ-QS-08). The 7-day staging cleanup in Documents is for Synapse staging, not for this.
- Replacement and versioning of a file on a draft response: allowed until submit; after submit the response is immutable, so replacement requires superseding the response.
- Legal hold on disclosure attachments: inherit from the existing Documents legal hold mechanism; confirm that holds on the document also freeze the response.
