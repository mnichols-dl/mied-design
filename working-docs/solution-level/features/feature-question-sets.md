Rough notes:
- A Question Set is a reusable capability for defining and managing a form tied to a business process (e.g., credential application) in MiEdWorkforce.
- The POC demonstrates a builder workflow + end-user preview workflow so admins can author a form and immediately see how it behaves for an applicant (optionally using a sample user context for demoing prefill/validation).
- There should be a build and a preview mode for administrators. This should use, if anything, a sample user for demonstrating functionality.
- The UI should generally enable:
  - Adding and deleting questions
  - Editing question text
  - Marking questions required vs optional
  - Reordering questions (within the same "level" or top-level questions; follow-ups under a parent option)
- The question set supports simple branching:
  - A multiple-choice response can trigger follow-up questions
  - Follow-ups can themselves trigger additional follow-ups (multi-level)
  - The branch naturally converges once no further follow-ups are triggered, returning to the next question in the main sequence (conceptually like a depth-first traversal of visible questions)
- The authoring experience should support configuring response behavior for choices, such as:
  - Normal continue
  - Trigger follow-up question(s)
  - Halt/terminate the form with an admin-authored message and optional "next steps"
- The form experience should support basic input types + validation, e.g.:
  - Short text fields (including email/phone/number/date/url subtypes)
  - Multiple choice
  - Validation feedback shown inline during preview/application
- The form should support reusable option lists ("datasets/lookup lists") for multiple choice questions, with an optional custom "other" value when needed.
- The question set should be versioned so admins can save iterations over time and the applicant submission can reference the version used (auditability).

# Some additional UI thoughts

- May need additional types of input validations
  - Date
    - Before Today
    - After Today
    - Any Date (default)
- Question text may include hyperlink (should always open in new tab)
- Option set choices
  - List of Michigan Universities
  - List of Michigan Universities (filtered by credential type) (this would require a question set instance to be tied to a particular credential definition)
  - List of Programs (filtered by user enrollment)
- For any of the above, there may be (choose to enter custom value)
- Pre-fill suggestion choices
  - Value from user profile (may include demog. data, PL hours, etc.)
  - Value from option set choice (not sure how to implement this -- this would need to definitely use the same identifier as the system-provided option set list up there) (also would only be visible if the question type was "single choice")

# Proposed design

1. Credential Definition Management:
   - Each credential (certificate or endorsement) has configurable requirements (e.g., MTTC (required test) completion, years of experience, program completion, maybe some others).
   - For each requirement, define whether failure blocks application submission or triggers a question set follow-up.
   - Allow selection of which question set is linked to a particular requirement if applicable.
   - Some requirements may be cross-cutting and not specific to a definition. (There's a thing called PPR that can raise issues like unaddressed arrest records that might block submission of ANY credential)
   - For some of these, like MTTC, it is relationship based. The code behind evaluating the requirement may need to do some data lookups. For others, it may be a configurable threshold. Maybe setting a minimum SCHECH hours that the user needs. I want enough here to sort of "simulate" these things and at least be able to point to what is a code change (e.g. checking the relationship) and what is a UI configurable thing (like establishing the relationship, or storing a threshold).
2. User Application Experience:
   - Show users a list of credentials they can apply for, sorted by those fully satisfied vs. those needing follow-up. Visually these should be shown in a clear manner, maybe tooltips showing what is missing.
   Users may apply for one certificate and two endorsements. Some of these may have the same question set referenced as a required follow up, so the end user will not answer the same question set over and over.
   - We should be able to show where the user is in the process. E.g. what question sets will be shown.
   - Launch the user into a question set flow for any unsatisfied requirements that allow supplementary input.
   - Question sets provide simple user input (e.g., multiple-choice or text) and may either allow continuation or block submission with a message.
3. Differentiating Flows:
   - Show a credential manager flow where an admin configures requirements, sets blocking vs. follow-up behavior for each of those, and assigns question sets (including creating question sets, probably in a separate area.).
   - Show an applicant flow where a user sees the requirements, passes or fails each, and encounters follow-up question sets if applicable.
4. Universal Rules:
   - Ensure any universal (non-configurable) requirements, like PPR flags, are always checked and block submission if unmet.
5. Technology & Prototype Scope:
   - Use a simplified Blazor WebAssembly-based prototype.
   - Represent a minimal credential definition screen with requirement configuration and follow-up question set assignment.
   - Demonstrate a basic question set flow (add question, choose response, block or continue).
   - Focus on simulating both the admin and end-user perspectives to validate the concept.
   - Ideally we should store state for any of this stuff in the browser somehow, maybe local storage or something, so it is easy to stage up a demo.