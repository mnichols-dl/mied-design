# Recommended Changes: EPP

**Impact: Low to medium.** EPP has one unmodeled Question Set requirement and several read-only dependencies on credential application responses. Files under `design/working-docs/solution-areas/epp/`.

## Changes

1. **FDD 13 EPP Question Sets are unmodeled.** The BRD summary (line ~547) says the EPP Admin defines Question Sets and validations for EPP processes. Add `epp` as a functional area for `questionsets.epp-question-sets.*` and note in `epp-domain.md` that EPP processes needing configurable questions use the capability. Do not invent EPP questions now; the review found no free-form questionnaire in the recommendation flow.
2. **Recommendation and endorsement forms stay structured.** The "Recommend Candidate for Credential" flow (`epp-sequences.md` L1406-1516) is a structured action form: endorsement checkboxes constrained to the EPP's approved list, optional internal remarks, and a warning on MTTC with override. Do not convert it to a question set. It does expose two capability-adjacent needs:
   - The **EPP approved endorsements** list is a natural option list provider (`epp.approved-endorsements`) if any Question Set ever needs it.
   - The **warning with override** is the same Error versus Warning idea as FDD 4.2. It stays a code-side validator in EPP until the BRM decision is made.
3. **Read-only response data on approval applications.** `ApprovalApplicationReview` holds `ApplicationAcknowledgements` and `ProfessionalPracticeAnswers` (`epp-domain.md` L189-190, L497; view sequence L1641-1688). If these answers come from Credentialing's Question Set responses, EPP should read them through Credentialing (or with a `questionsets.epp-responses.view` scoped to its own applicants) rather than copying text. Update the "View Approval Application Detail for EPP Review" sequence accordingly.
4. **Option lists that depend on EPP.** "Programs filtered by the applicant's enrollment" (feature notes) needs an EPP provider contract: given a subject, return the programs they are enrolled in. Document the provider in EPP's Dependencies (downstream).
5. **Open items already in EPP that this touches:**
   - Open Question #11 and the worklist assignment (per-user assignment, FDD 13.3.1 and FDD 5.5) is the same routing question as Credentialing's Assigned-only scope. Answer together; Question Sets does not solve it.
   - Open Question #5 (Alternative Pass business rules) and FDD 25 transition ordering stay with the BRM decision.
   - Remarks required versus optional (REV 13-25 Discrepancy #3) is unrelated to question sets.
6. **Permissions.** EPP has no question permission. Add `epp` functional area to the Question Sets pattern only when an EPP question set is actually needed.
