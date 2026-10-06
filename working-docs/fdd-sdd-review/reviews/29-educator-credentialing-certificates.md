# Review: Educator Credentialing - Certificates (29)

| Field | Value |
|---|---|
| Source Document(s) | `29 - Educator Credentialing - Certificates/29 - Educator Credentialing - Certificate.docx` (reviewed via its `.md` sibling — this is the review's only scored document); noted, out of scope (unconverted diagrams, tracker rows left `Not Started`): `Certification Flow Diagrams - Certificates.pdf`, `Certification Flow Diagrams - Certificates.vsd`, `IT0000001193615-2025-05-01_03-10-35-PM.pdf`, `School Social Worker - Certificate & Permit.vsdx`, `SSW Renewal.vsdx` |
| Related Domain(s) | credentialing (primary); epp, proflearning, documents, communications, payments (cross-domain references) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-14 |
| **Overall Status (per document)** | `29 - Educator Credentialing - Certificate.docx`: **Aligned — Minor SDD Revisions Needed**. The five diagram/PDF files: **Not Started** (unchanged, out of scope for this pass). |

## Summary

FDD 29 is the detailed citizen/educator-facing "landing page" specification that FDD 11's
Feature 11.5.4 cites as "Ref User Stories 29.2" for the standard Certificate Application
Workflow — this review closes that citation loop. The document is written entirely from
the Citizen User's own landing page (Apply/Renew, Confirm Enrollment, Add College
Credits/DPPD, Add Endorsements, Print Cover Letter, Complete SCECH Evaluations, Print
Certificate, View Enrollment/Guidance/Pending Applications) with **no admin-submission
path described anywhere** — it strongly and directly confirms the educator-self-service-
primary model already locked in for certificates (`credentialing.application.submit`,
"Submit New Certificate Application" sequence), and, being purely citizen-facing, is
silent on admin correction/nullification/manual-issuance capabilities and adds nothing new
toward the open `credentialing.credential.issue` gap. It corroborates the PPR/Needs-
Responses gating pattern already modeled, but also surfaces one real internal SDD gap this
review fixed directly: `credentialing-sequences.md`'s "Renew Credential" sequence had no
PPR-clearance check at all, even though FDD 29 treats "Apply/Renew" as a single combined
action with identical PPR-flag routing for both. It also surfaces, in the opposite
direction, that endorsement-addition applications are consistently *not* PPR-gated per the
FDD — now documented as a deliberate exception rather than a silent sequence-diagram
omission. The certificate-specific content this FDD's title promises (Print Certificate,
Print Cover Letter) adds one genuinely new PDF-rendering rule (blank issuance/expiration
dates for Revoked/Nullified/Suspended certificates) and one entirely unmodeled document-
generation capability (Cover Letter Generation, per pending application) — both now
reflected in `credentialing-technical-design.md`. Several other landing-page actions
(Confirm Enrollment, Add College Credits, Add DPPD, Complete SCECH Evaluations) describe
real functionality with no home in `credentialing-domain.md` today, but most look like
they belong to `epp`/`proflearning` rather than `credentialing` itself — flagged as
coverage gaps/cross-domain notes rather than drafted.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | Wireframes 1-10 throughout (e.g. Enrollment Info table: "Student ID, Provider, Program level, Program(s), Submitted Date, Status, Enrollment date, Exit Date, Exit reason"; Pending Applications table: "Application #, Certificate Type, Submitted on, Status"; SCECH Evaluations table columns) | Exhaustively enumerates exact table columns for every landing-page list/search screen | Same shape as the FDD 05 and 11 reviews' wireframe-column findings — screen/field-inventory detail rather than a business requirement. The underlying data these tables surface is either already modeled (`CredentialApplication`, `ISSUED_CREDENTIALS`) or belongs to a cross-domain source (`epp` enrollment data) not owned by credentialing at all. | No SDD action needed; consistent with the precedent set in the 05 and 11 reviews (not escalated to Boundary Violation). |
| 2 | 29.4.5 (Delete Pending Certificate Application): "The user can select the 'More' menu (three dots) and will be presented with a 'Delete' option"; 29.4.1: "a single certificate via the download icon within the row" | Names the exact UI control/affordance (a "More" kebab menu, a row-level download icon) rather than the underlying capability ("the user can delete a Submitted application" / "the user can download an approved certificate") | Minor, low severity — same pattern as prior reviews' UI-widget findings (e.g. FDD 05's "Rich text editor," FDD 17's mentor-search "dynamically-filtered search field"). The SDD's "Delete Pending Application" and "Print Certificate" sequences already capture the functional behavior correctly. | No SDD action needed; note for future FDD authoring to describe the capability, not the control. |
| 3 | 29.1.8 (Persistent Certificate Dashboard State): "The system stores dashboard preferences tied to the user profile... Preferences persist across sessions and devices" | Describes generic cross-session UI-preference persistence as a certificate-specific feature | This is a platform/dashboard capability, not credentialing-specific — and the FDD's own Dependencies section already defers dashboard behavior to "the role-based dashboard (3)." Same disposition as FDD 11's Feature 11.7 (Dashboard), which was correctly treated as out of scope for the credentialing domain. | No SDD action needed; out of scope for `credentialing-domain.md` per the FDD's own disposition — flag only if/when the Dashboard FDD (3) is reviewed. |
| 4 | 29.1.6/29.4.1: exact PDF field list (Certificate type, Endorsements, Issue date and expiration date) | Names specific PDF content fields | Not actually a boundary problem — this level of detail already has a correct home at the same altitude in `credentialing-domain.md`'s existing "Certificate Document Generation" business rule and its "PDF Contents" list, which predates this FDD and is more complete. | No action; this FDD corroborates rather than adds new prescriptive detail. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | FDD vs. SDD (resolved — strongly confirms) | Who submits a standard certificate application | FDD 29, entire document: every single action (Apply/Renew, Confirm Enrollment, Add College Credits, Add DPPD, Add Endorsement, Print Cover Letter, Complete SCECH Evaluations, Print Certificate, view/delete pending applications) is performed by "the MiEdWorkforce Citizen user" directly, from their own landing page. No admin, district, or "Business Credential Applications role" actor appears anywhere in the document — not even as the narrower exception path FDD 11's Feature 11.5.4 describes for certificates. | `credentialing-sequences.md`'s "Submit New Certificate Application" sequence (`actor Educator`, admin-on-behalf as a secondary path); `credentialing-permissions.md`'s `credentialing.application.submit` (Self-only, "All authenticated educators"). | **This is the strongest confirmation yet of the educator-self-service-primary model** — stronger than FDD 11,16's finding (Discrepancy #1 in that review), because this is the actual detailed document FDD 11.5.4 cites by name ("Ref User Stories 29.2") for the Certificate Application Workflow, and it describes zero admin-submission scenarios of any kind, not even as a minor/exception path. This closes the citation loop from FDD 11 cleanly in favor of the SDD's existing model. | **No SDD action needed — model is correct as-is.** Answering this review's assigned priority question directly: FDD 29 confirms (does not merely refine) the educator-self-service-primary model for certificates. It adds no admin-submission evidence at all, consistent with — and in fact more one-sided than — FDD 11,16's finding. |
| 2 | SDD vs. itself (fixed directly, 2026-08-27) | PPR-clearance check on renewal applications | FDD 29's main narrative treats "Apply/Renew" as one combined landing-page action; every certificate type's business-rule bullet list (repeated ~7 times, once per certificate type) includes, verbatim and without distinguishing new vs. renewal: "Route or assign the application to worklist(s) based on the PPR flag" and "Determine the status of the application based on the Needs Responses flag." | `credentialing-sequences.md`'s "Submit New Certificate Application" and "Submit Permit Application" sequences both call `GET /educators/{id}/ppr-clearance` and branch on the result; the "Renew Credential" sequence, prior to this review, had **no PPR check step at all** — it went straight from BRM eligibility validation to application creation. | Genuine, well-evidenced internal gap: nothing in `credentialing-domain.md`'s Auto-Approval Criteria or Manual Review Triggering rules exempts renewals from the PPR gate, and FDD 29's own framing (Apply/Renew as one flow) treats them identically. This is the same shape as prior direct fixes in this series (a missing branch mirroring an already-established pattern elsewhere in the same document). | **Fixed directly.** `credentialing-sequences.md`'s "Renew Credential" sequence now includes the same `GET /educators/{id}/ppr-clearance` check and `Blocked`/`Hold`/`Clear`-or-`ConditionalClearance` branching as "Submit New Certificate Application," with matching state-change and error-scenario updates. |
| 3 | SDD vs. itself (fixed directly, 2026-08-27) | PPR-clearance check on endorsement-addition applications | FDD 29's Feature 29.3 (Additional Endorsements) narrative and its 11 acceptance-criteria items (29.3.1-29.3.11) consistently **omit** any PPR-flag or Needs-Responses language — every other certificate-type feature description in the same document explicitly lists it, but the endorsement-addition feature never does, across both the main Business Specifications section and the Addendum. | `credentialing-domain.md`'s "Auto-Approval Criteria" business rule lists `ProfessionalPracticeStatus = Clear` as condition 1 of 5 for auto-approval, worded generically enough to read as applying to all `CredentialApplication` types; `credentialing-sequences.md`'s "Add Endorsement to Certificate" sequence has no PPR check step, which — without this FDD's corroboration — could plausibly have read as an unintentional omission rather than a deliberate design choice. | The FDD's consistent, repeated omission across 11 separate acceptance-criteria items is strong enough evidence that this is a deliberate business rule (endorsement additions to an *already-certified* educator don't re-trigger PPR screening) rather than an SDD documentation gap. | **Fixed directly.** Added an explicit "Exception — Endorsement Additions" note to `credentialing-domain.md`'s Auto-Approval Criteria rule, and a "No PPR Check" Key Decision to `credentialing-sequences.md`'s "Add Endorsement to Certificate" sequence, both citing FDD 29 so a future reader doesn't mistake the sequence's silence for an oversight. |
| 4 | FDD vs. SDD | PDF rendering for Revoked/Nullified/Suspended certificates | FDD 29.1.6/29.1.9: certificate PDFs show "Issue date and expiration date (**or blank for revoked/nullified/suspended**)"; the citizen's certificate list "visually distinguishes revoked, suspended, or nullified certificates (as their dates may be blank)." | `credentialing-domain.md`'s "Certificate Document Generation" business rule and `credentialing-technical-design.md`'s "Certificate PDF Generation" section described PDF content only for the happy path (issued, valid) with no rule for what a PDF/list entry looks like for a non-valid credential. | Real, narrow, well-evidenced gap — not a conflict, just previously undocumented. Straightforward to model since the FDD states the rule plainly (blank dates, not omission from the list, not the original historical dates). | **Fixed directly.** Added the blank-dates rendering rule to both `credentialing-domain.md` (under "Certificate Document Generation") and `credentialing-technical-design.md` (under "Certificate PDF Generation"), citing FDD 29. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | Business Specifications, Feature 29.1/29.4.2 (Print Cover Letter): "Cover letters are generated based on the business rule definition with respect to the type of certificate that the citizen applied for," per-pending-application, with a "No Pending Applications" empty state and download/logging behavior | An entire document-generation capability distinct from Certificate PDF Generation (different trigger — a pending application, not `CredentialIssued`; different content — a cover letter, not the certificate itself). No aggregate, business rule, or sequence anywhere covers it. | `credentialing-technical-design.md` (new "Cover Letter Generation" subsection, sibling to "Certificate PDF Generation") or `credentialing-domain.md` (new business rule) | M | Noted directly in `credentialing-technical-design.md` (see "Related, Not Yet Modeled" callout) rather than drafted in full — the FDD doesn't specify what the per-certificate-type cover letter business rules actually contain beyond "as defined," so a full spec would be guessing. |
| 2 | Business Specifications, Feature 29.1/29.4.3-29.4.4 (Confirm Enrollment / View Enrollment Information): citizen self-submits EPP Name, Program Level, Program Type, Student ID "to establish my enrollment record in the system for verification by my EPP," triggering an email confirmation; separately views enrollment status/history in a table | `credentialing-domain.md`'s Dependencies table only models `epp` as an upstream source Credentialing *reads* enrollment verification from ("REST API call during eligibility check") — it has no representation of the educator *writing* a new enrollment-confirmation record through the credentialing UI, nor of viewing enrollment history. This looks like an `epp`-domain write path exposed through the credentialing landing page, not a credentialing-owned aggregate. | `epp` domain (not reviewed as primary in this pass) — likely a "Confirm Enrollment" aggregate/sequence there, with credentialing's landing page as the UI entry point only | M | Flag for whoever reviews the EPP-domain FDDs (13, 25, referenced in this FDD's own Dependencies); this document alone doesn't have enough EPP-side detail (e.g., what "Program Level"/"Program Type" values exist, how EPP verifies the confirmation) to draft the aggregate from. |
| 3 | Business Specifications, Feature 29.1 (Add College Credits) and Wireframe 5: Course number, Course Title, Semester Credit hours, College/University name, Date Completed, with "business rule validation" | No `CollegeCredit`/coursework-tracking entity anywhere in `credentialing-domain.md`. This data feeds the "Professional Development hours by source" view (College credits / District PD & SCECHs / pre-2020 DPPD / Totals), which reads like a `proflearning`-domain concern (SCECH/hours tracking), consistent with the domain's own Scope exclusion ("Professional learning hours tracking > owned by proflearning"). | `proflearning` domain (not reviewed as primary in this pass) | M | Flag for the Professional Learning domain's FDD review; credentialing's role appears to be UI entry-point only (landing-page action), same pattern as the Confirm Enrollment gap above. |
| 4 | Business Specifications, Feature 29.1 (Add DPPD) and Wireframe 6: Professional Development Category, Activity Title, School District, Hours Engaged, Date; additional DPPD form submission during renewal; "DPPD Earned prior to July 01, 2020" as a distinct legacy bucket | Same disposition as Coverage Gap #3 — professional-development/hours tracking, not modeled in `credentialing-domain.md` and likely `proflearning`-owned. The "prior to July 01, 2020" legacy cutoff is a specific historical-data-migration detail worth preserving when that domain is reviewed. | `proflearning` domain | L | Flag alongside Coverage Gap #3 for the Professional Learning domain review. |
| 5 | Business Specifications, Feature 29.1 (Complete SCECH Evaluations): citizen views attended programs awaiting evaluation and submits evaluations | Same disposition — SCECH/professional-learning tracking, no home in `credentialing-domain.md`, likely `proflearning`-owned (the domain's Dependencies table already lists `proflearning` as the source of "SCECH hours completed" for renewals, but not as a place educators submit program evaluations). | `proflearning` domain | L | Flag alongside Coverage Gaps #3-4. |
| 6 | 29.1.2/29.2.6: MTTC score validity described as "not older than 5 years," and introduces "**Professional Knowledge and Skills (PKS) test**" as an assessment type alongside MTTC, dynamically determined per endorsement | `credentialing-domain.md`'s `AssessmentDefinition`/`AssessmentProvider` examples only name "Pearson-MTTC" and "ETS-Praxis" — "PKS" doesn't appear anywhere. The generic `AssessmentDefinition` model can absorb it as another provider/identifier without a structural change; this is a terminology/example-completeness gap, not a modeling gap. | `credentialing-domain.md` — add "PKS" as an example `AssessmentProvider`/identifier | L | Low priority; corroborates rather than conflicts with the 5-year validity example already in the "Assessment Result Validity" business rule. Worth a one-line addition next time that section is touched. |
| 7 | 29.1.3/29.4.5: application deletion is blocked "if routing has begun (**OPPS**, **EPP**, or **ARP** involvement)" | `credentialing-sequences.md`'s "Delete Pending Application" sequence only checks `status = Submitted` vs. not — functionally consistent, but the three named routing destinations (OPPS, EPP, ARP) don't appear anywhere else in the credentialing FDD corpus reviewed so far, which instead uses "OEE worklist" and "PPR worklist" (FDD 17) for manual-review routing. Unclear whether OPPS/ARP are synonyms for OEE/PPR under different names, or genuinely distinct legacy-MOECS routing destinations. | `credentialing-domain.md` — no structural gap (the underlying "only Submitted can delete" rule is already correct), but worth a terminology reconciliation note | L | Internal open question (see below) rather than a modeling gap — the behavior is already correctly captured either way. |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `29 - Educator Credentialing - Certificate` §Business Specs, Wireframes 2-3, Feature 29.2.1-29.2.10 (Apply/Renew, all 6 certificate types) | credentialing | `CredentialApplication`; Sequence: Submit New Certificate Application; `credentialing.application.submit` | implements — strongly confirms self-service-primary model (see Discrepancy #1) |
| `29 - Educator Credentialing - Certificate` §Feature 29.2.7 (Needs Responses flag gating) | credentialing, profpractice | `ProfessionalPracticeStatus`; "Needs Responses" self-disclosure gap (tracked in 09/17 reviews) | implements (corroborates existing gap tracking, no new content) |
| `29 - Educator Credentialing - Certificate` §Feature 29.2.8 (PPR flag worklist routing) | credentialing, profpractice | "Professional Practice Clearance" business rule; Sequences: Submit New Certificate Application, Application Auto-Approval | implements |
| `29 - Educator Credentialing - Certificate` §Business Specs (Renew via combined Apply/Renew action) | credentialing | Sequence: Renew Credential | gap — fixed directly (see Discrepancy #2) |
| `29 - Educator Credentialing - Certificate` §Feature 29.2.6, 29.2.9 (MTTC/PKS real-time validation, in-state/out-of-state routing) | credentialing | `AssessmentResult`/`AssessmentDefinition`; "Subject Area Assessment Validation" business rule | implements — PKS example gap noted (Coverage Gap #6) |
| `29 - Educator Credentialing - Certificate` §Feature 29.3.1-29.3.11 (Additional Endorsements) | credentialing | `IssuedEndorsement`; Sequence: Add Endorsement to Certificate | implements — PPR-exception fixed directly (see Discrepancy #3) |
| `29 - Educator Credentialing - Certificate` §Feature 29.4.1 (Print/download certificates) | credentialing | "Certificate Document Generation" business rule; Sequence: Print Certificate; `credentialing.credential.print` | implements — blank-dates rule fixed directly (see Discrepancy #4) |
| `29 - Educator Credentialing - Certificate` §Feature 29.1.9 (visual distinction for Revoked/Suspended/Nullified) | credentialing | `IssuedCredential.CredentialStatus` | implements — corroborates existing states, adds rendering rule (Discrepancy #4) |
| `29 - Educator Credentialing - Certificate` §Feature 29.1/29.4.2 (Print Cover Letter) | credentialing, documents | `credentialing-technical-design.md` — "Cover Letter Generation" (noted, not modeled) | gap — not yet modeled (see Coverage Gap #1) |
| `29 - Educator Credentialing - Certificate` §Feature 29.1/29.4.3-29.4.4 (Confirm Enrollment, View Enrollment Information) | credentialing, epp | (no equivalent write path modeled) | gap — not yet modeled; likely epp-owned (see Coverage Gap #2) |
| `29 - Educator Credentialing - Certificate` §Wireframe 5 (Add College Credits) | credentialing, proflearning | (no equivalent aggregate) | gap — not yet modeled; likely proflearning-owned (see Coverage Gap #3) |
| `29 - Educator Credentialing - Certificate` §Wireframe 6 (Add DPPD, PD hours by source) | credentialing, proflearning | (no equivalent aggregate) | gap — not yet modeled; likely proflearning-owned (see Coverage Gap #4) |
| `29 - Educator Credentialing - Certificate` §Business Specs (Complete SCECH Evaluations) | credentialing, proflearning | (no equivalent aggregate) | gap — not yet modeled; likely proflearning-owned (see Coverage Gap #5) |
| `29 - Educator Credentialing - Certificate` §Feature 29.1.3, 29.4.5 (delete pending application, OPPS/EPP/ARP routing block) | credentialing | Sequence: Delete Pending Application | implements — terminology question noted (Coverage Gap #7) |
| `29 - Educator Credentialing - Certificate` §Feature 29.1.8 (dashboard widget persistence) | credentialing, dashboards (unreviewed) | n/a | informs — out of scope per FDD's own disposition |
| `29 - Educator Credentialing - Certificate` §Business Specs (fee collection via CEPAS) | credentialing, payments | "Payment Requirement" business rule | implements |
| `29 - Educator Credentialing - Certificate` §Business Specs (document upload for applications) | credentialing, documents | `SubmittedDocuments` | implements |
| `29 - Educator Credentialing - Certificate` §Business Specs (View Certification Guidance Documents) | credentialing | `credentialing.guidance.view` | implements |
| `29 - Educator Credentialing - Certificate` (entire document) — absence of any admin correction/nullification/manual-issuance content | credentialing | `credentialing.credential.issue`; Sequence: Issue Temporary Permit — Exceptional Cases (Admin) | informs — no new evidence either way (see priority answer below); the FDD's scope (citizen landing page only) means its silence carries less weight than FDD 05's (System Admin FDD) |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Are "OPPS" and "ARP" (FDD 29's application-deletion-block routing destinations) the same offices/worklists that FDD 17 calls "OEE" and "PPR worklist" under different names, or genuinely distinct legacy-MOECS routing paths? | Internal | Open |
| 2 | What are the actual business rules governing per-certificate-type cover letter content (Feature 29.1/29.4.2)? The FDD states only that they're "generated based on the business rule definition," with no further detail provided. | Internal (resolvable once Business Rule Management FDD (4) or a follow-up cover-letter-specific document is reviewed) | Open |

Note: neither question clears the `client-questions.md` judiciousness bar. #1 is a
low-stakes terminology question that doesn't change any modeled behavior (the underlying
"only Submitted status is deletable" rule is unaffected either way) — resolvable, if it
ever matters, by checking the unconverted diagram files in this same FDD folder rather
than asking the client. #2 isn't blocking anything today since Cover Letter Generation
hasn't been drafted as a full capability yet (see Coverage Gap #1) — it becomes worth
asking only once that capability is actually being built.

## Related Documents

- Source document: `functional-design-docs/29 - Educator Credentialing - Certificates/29 - Educator Credentialing - Certificate.md`
- Noted, out of scope, not converted (unchanged in tracker): `Certification Flow Diagrams - Certificates.pdf`, `Certification Flow Diagrams - Certificates.vsd`, `IT0000001193615-2025-05-01_03-10-35-PM.pdf`, `School Social Worker - Certificate & Permit.vsdx`, `SSW Renewal.vsdx`
- SDD documents reviewed against: `solution-areas/credentialing/credentialing-domain.md` (updated), `solution-areas/credentialing/credentialing-permissions.md`, `solution-areas/credentialing/credentialing-sequences.md` (updated), `solution-areas/credentialing/credentialing-technical-design.md` (updated), `solution-areas/credentialing/credentialing-api.yml` (endpoint coverage skim)
- Related prior reviews: [11-educator-credentialing](11-educator-credentialing.md) (the FDD 11.5.4 → FDD 29 citation this review closes the loop on), [05-system-admin-credentialing](05-system-admin-credentialing.md) (the stronger, dedicated source for the admin-issuance/`credentialing.credential.issue` question), [17-educator-temporary-credentials](17-educator-temporary-credentials.md), [09-professional-practices](09-professional-practices.md), [06-alerts-emails-communications](06-alerts-emails-communications.md)

---

## Answers to This Review's Priority Questions

**1. Does FDD 29 confirm, refine, or contradict the educator-self-service-primary model
for certificates?** **Confirms it, more strongly than any prior source.** FDD 29 is
written entirely from the Citizen User's own landing page; every action across all four
Features (29.1-29.4) is performed directly by the educator, with zero mentions of an
admin, district, or "Business Credential Applications role" actor anywhere in the
document — not even as the narrower on-behalf exception FDD 11's Feature 11.5.4 itself
describes. Since this is the actual document FDD 11.5.4 cites as "Ref User Stories 29.2,"
this closes that citation loop cleanly: no change needed to `credentialing-sequences.md`'s
"Submit New Certificate Application" or `credentialing-permissions.md`'s
`credentialing.application.submit` (Self-only).

**2. Does it add anything new about the Certificate PDF Generation process?** Yes, one
concrete addition: PDFs (and the citizen's certificate list) show **blank issuance/
expiration dates for Revoked, Nullified, and Suspended certificates**, rather than hiding
those certificates from the list or showing their original historical dates (Feature
29.1.6, 29.1.9). This is now reflected in both `credentialing-domain.md`'s "Certificate
Document Generation" business rule and `credentialing-technical-design.md`'s "Certificate
PDF Generation" section. Separately, the FDD's "Print Cover Letter" feature describes a
related but structurally distinct document-generation capability (per pending application,
not per issued credential) that has no home in the SDD at all — flagged as Coverage Gap #1
and noted (not drafted) in `credentialing-technical-design.md`.

**3. Does it describe anything resembling admin override/nullification/manual-issuance for
certificates that should inform the open `credentialing.credential.issue` gap?** **No.**
FDD 29 is scoped entirely to the citizen landing page and contains no admin actions of any
kind — no suspend/revoke/nullify, no manual issuance, no correction workflow. This is
different in kind from FDD 05's silence (the dedicated System Administrator FDD, which
*does* cover the Credential Admin's full toolset and still describes no such capability —
the strong corroborating evidence already noted in `client-questions.md` #8): FDD 29's
silence is expected given its scope and carries no additional evidentiary weight either
way. The `credentialing.credential.issue` gap remains open exactly as documented in the 05
review; no change made here.

## Process/Template Friction Noted

None beyond what prior reviews in this series already flagged (wireframe-column altitude
findings, the value of checking whether a "boundary violation" IDD/diagram might still
carry a load-bearing fact). This review's main takeaway for the process itself: FDD 11's
explicit "Ref User Stories 29.2" citation was a reliable signal that a corroborating,
scope-narrower document existed and was worth chasing down — worth keeping an eye out for
similar internal cross-references in FDDs not yet reviewed (e.g., "16" was already
identified as a stub deferring to "11,16"; this suggests the corpus has more such
citation chains than have been explicitly tracked so far).
