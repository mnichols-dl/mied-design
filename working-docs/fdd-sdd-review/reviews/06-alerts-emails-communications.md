# Review: Alerts, Emails, Communications (06)

| Field | Value |
|---|---|
| Source Document(s) | `06 - Alerts, Emails, Communications/6 - Alerts, Emails and Communication.docx` (main FDD, reviewed via `.md`)<br>`06 - Alerts, Emails, Communications/Comment Logs.docx` (reviewed via `.md`)<br>`06 - Alerts, Emails, Communications/Emails and Remarks.docx` (reviewed via `.md`)<br>`06 - Alerts, Emails, Communications/Interface Design document - Microsoft 365 (Outlook).docx` (reviewed via `.md`)<br>`06 - Alerts, Emails, Communications/Interface Design document - SendGrid.docx` (reviewed via `.md`)<br>`06 - Alerts, Emails, Communications/MOECS_CommentsScreenShot.docx` (reviewed via `.md`)<br>Noted, not reviewed: `06 - Alerts, Emails, Communications/101022 Email notifications research (1).pptx` (unconverted `.pptx`, out of scope per process README) |
| Related Domain(s) | communications (primary); credentialing, staffing, profpractice, iam (cross-domain — comment visibility, template ownership) |
| Reviewer | Claude |
| Date Reviewed | 2026-08-14 |
| **Overall Status (per document)** | `6 - Alerts, Emails and Communication.docx`: **Discrepancy — Needs Decision**. `Comment Logs.docx`: **Gaps Identified**. `Emails and Remarks.docx`: **Aligned — Minor SDD Revisions Needed**. `Interface Design document - Microsoft 365 (Outlook).docx`: **Boundary Violation** (also a genuine Coverage Gap — see below). `Interface Design document - SendGrid.docx`: **Boundary Violation** (also resolved an existing SDD Open Question). `MOECS_CommentsScreenShot.docx`: **Aligned**. |

## Summary

This is the first review of the `communications` domain, which — unlike credentialing/staffing at
their first reviews — already has a complete SDD (capability doc, permissions, sequences,
technical-design, and API spec). The main FDD narrative aligns well with the SDD's template/alert
lifecycle model (versioning, functional-area scoping, event binding, audit trail) and its
manual-send/customization-without-mutating-the-template behavior is a near-exact match to
`communications-sequences.md`'s "Manual Email Send with Template Customization" sequence. Two
substantive findings came out of this review. First, the main FDD contains a genuine **internal
contradiction**: it explicitly disclaims "a direct or two-way integration with a Microsoft Outlook
client" for viewing sent emails, while a dedicated Outlook/Graph API Interface Design Document in
the same folder describes exactly that — a two-way, non-PII correspondence mailbox integration —
which `communications-capability.md` doesn't model at all (SendGrid is its only outbound channel).
Second, and most consequential for the cross-domain design question this review was asked to
inform: `Comment Logs.md`'s "Remark Category" table (Professional Practices / Application / Account
/ Employment) states plainly that "those with PPR categorization are only viewable to those with
PPR access" — this is real, specific evidence bearing directly on the credentialing "Internal
Comment Group" open question from the FDD 05 review, and it resolves in favor of the repo owner's
atomic-permission instinct rather than a generic configurable visibility-group abstraction (see the
Cross-Domain Question section below; `credentialing-domain.md` Open Question #11 has been updated
accordingly). Separately, this review found and fixed a factual error: `communications-technical-
design.md`'s SendGrid rate-limit figures (100 req/sec) contradicted the SendGrid Interface Design
Document's stated account limit (10,000 req/sec) and left `communications-capability.md`'s own Open
Question #1 unresolved despite the answer sitting in the same FDD folder — both have been corrected.

---

## 1. Altitude / Boundary Check

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | `Interface Design document - Microsoft 365 (Outlook).docx`, entire document (Microsoft Graph API contract: `sendMail` endpoint, full JSON message schema, C# SDK sample code, application-registration/RBAC auth details) | A REST/SDK integration contract for a specific vendor API (Microsoft Graph), including endpoint URL, full message schema, and language-specific sample code | Same category as the MiLogin, MTTC, and CEPAS IDDs already flagged as Boundary Violations in prior reviews (01, 11, 07) — wire-format/API-contract detail, not functional content. | Relocate to a technical/interface-design location, consistent with prior precedent. Unlike the CEPAS/MiLogin cases, however, the *underlying capability* this IDD describes (a two-way, non-PII Outlook correspondence mailbox, distinct from SendGrid's one-way templated notifications) has **no home anywhere in `communications-capability.md`** — see Coverage Gap #1 below. Relocating the IDD doesn't resolve that; the business capability itself still needs to be modeled. |
| 2 | `Interface Design document - SendGrid.docx`, entire document (SendGrid v3 REST API contract, JSON payload schema, C# library "Kitchen Sink" sample covering every SDK method, bearer-token auth) | A REST API / SDK integration contract for the SendGrid vendor | Same category as #1. | Relocate to a technical/interface-design location. Unlike other reviewed IDDs, this one is **immediately useful** — it states the account's rate limit ("10k calls per second max"), which directly resolves `communications-capability.md`'s existing Open Question #1 and corrected a wrong figure already in `communications-technical-design.md` (see Discrepancy #3). Recommend technical-design authors always cross-check vendor IDDs for exactly this kind of fact before leaving an Open Question unanswered. |
| 3 | Main FDD, Assumptions ("The system will use SendGrid API...") and Feature 6.1.3 Acceptance Criteria ("must interface with the SendGrid API... successfully authenticates with the SendGrid service... creates batches... if the number of emails exceeds the thresholds defined by the SendGrid service integration") | Names a specific vendor/product as a functional acceptance criterion rather than stating the business requirement ("reliable bulk email delivery with batching for high-volume sends") | Borderline, low severity — same pattern as the FDD 07 review's "AES over TLS" finding (Altitude #4 in that review): naming an implementation choice in an otherwise business-level acceptance-criteria list. Because `communications` already has a complete technical-design doc, this content has a natural home and isn't a symptom of a documentation gap. | No SDD action needed — already correctly elaborated at the right altitude in `communications-capability.md`'s Integration Patterns ("SendGrid Email Delivery") and `communications-technical-design.md`. Note for future FDD authoring: describe the requirement and let the technical design own the vendor choice. |
| 4 | Main FDD, Feature 6.3.2 Acceptance Criteria ("MDE letterhead", "CEPI letterhead" as specific image inserts) and Feature 6.2.1 ("rich text editor... font selection restricted to approved system fonts") | Names specific UI assets/controls (named letterhead images, a "rich text editor" widget) rather than the underlying business requirement ("branded header images can be inserted," "formatted text with ADA-compliant fonts") | Minor — same shape as the FDD 05 review's "Message – Rich text editor" finding (Altitude #2 in that review), not escalated to Boundary Violation there either. | No SDD action needed; note for future FDD authoring to describe the capability rather than the widget/asset name. |
| 5 | `MOECS_CommentsScreenShot.docx` (entire document — three sentences describing a legacy MORE-system screen: application remarks history, linked-action popup, "must then make new remarks") | Minimal narrative referencing a specific legacy-system screen, not a business rule | Not really an altitude violation (too thin to prescribe an implementation) — flagged here only because it's the kind of document that could be mistaken for an IDD. | No action — corroborates (does not conflict with) `communications-capability.md`'s `Comment` aggregate and `Emails and Remarks.md`'s remarks-history description. See Tagging. |

## 2. Discrepancies

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents disagree with each other.
- **FDD vs. itself** — the client's own source documents disagree with each other.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | **FDD vs. itself** | Does MiEdWorkforce have a direct, two-way integration with Microsoft Outlook? | Main FDD, Business Specifications: "The system will provide the ability to view actual email messages within the application, using an embedded viewer, **without requiring a direct or two-way integration with a Microsoft Outlook client**." | `Interface Design document - Microsoft 365 (Outlook).md`: "MiEdWorkforce can send emails from **dedicated email addresses** that are intended for **two-way communication** as opposed to alerts or notifications... Data Direction: **Two way**," using the Graph API `sendMail` endpoint from an actual Outlook mailbox. | Not a contradiction about the same channel — these describe two genuinely different capabilities that the FDD's own wording makes easy to conflate: (a) an **embedded viewer** for *system-generated* (SendGrid) emails, which correctly has no Outlook dependency, and (b) a **separate, dedicated-mailbox, two-way correspondence channel** (explicitly "as opposed to alerts or notifications," and explicitly excluding PII: "should not include PII since email is not a private communication method") that *does* use Outlook/Graph API directly. The main FDD's Business Specifications never mention this second channel at all — an omission, not a direct contradiction — but a reader relying on the main FDD alone would reasonably conclude no Outlook integration exists anywhere in the system, which is wrong. | Not resolvable as a simple pick-one-side call — both documents are accurate about different things. Recorded as an Open Question below (Internal) rather than corrected silently, and surfaced as Coverage Gap #1 since the SDD has no representation of channel (b) at all. |
| 2 | FDD vs. SDD | Historical email/attachment retention period | Main FDD, Business Specifications: "The system will store the historical email, including attachments, for a defined period of time. **6 months**. The MiEdWorkforce System Administrator will be able to modify the defined period of time." | `communications-capability.md`'s Ubiquitous Language ("Email Content Retention... typically **90 days**"), "Email Resend Availability Window" business rule, and Data Retention section all use **90 days** as the stated default for `content_retention_days`. | Both sides agree the value is admin-configurable, so this isn't a behavioral disagreement — but the SDD's chosen *default* (90 days / ~3 months) doesn't match the FDD's stated default (6 months / ~180 days), and nothing in the SDD documents this discrepancy or explains the choice. | **Not applied directly** (a configuration default is a judgment call, unlike the SendGrid rate-limit fact below, which was a stated external constraint). Recommend `communications-capability.md`'s default be reconciled to 180 days (or the SDD explicitly note it deliberately differs from the FDD's example and why) the next time this document is touched. Not promoted to `client-questions.md` — doesn't block anything since the value is configurable either way. |
| 3 | **SDD vs. itself — RESOLVED (2026-08-27)** | SendGrid account rate limit | `Interface Design document - SendGrid.md`: "Data Volume: As needed by the application. **Rate limits apply (10k calls per second max)**." | `communications-technical-design.md`'s "SendGrid Rate Limiting" section previously stated "Account Tier: **100 requests/second**" (with an in-memory queue throttled to 95 req/sec); `communications-capability.md`'s "SendGrid Email Delivery" integration pattern previously stated the same 100 req/sec constraint. Neither cited a source, and `communications-capability.md`'s own Open Question #1 ("What are the SendGrid rate limits and account quotas?") was still open despite the IDD sitting in the same FDD folder. | Two orders of magnitude apart — not a rounding difference. The IDD is the primary source for this fact (an interface spec naming the actual contracted account tier), and directly answers the SDD's own open question. | **Done.** `communications-technical-design.md`'s rate-limiting section and `communications-capability.md`'s SendGrid integration pattern updated to 10,000 req/sec (9,500 req/sec with safety margin), citing the SendGrid IDD. `communications-capability.md` Open Question #1 marked Resolved with the citation. No account *quota* (daily/monthly send cap) is stated anywhere in the IDD — only the per-second rate — so that half of the original question remains genuinely open if it matters later. |
| 4 | FDD vs. SDD (central cross-domain question — see full answer below) | Comment/remark visibility model | `Comment Logs.md`'s "Remark Category" table lists four categories (Professional Practices, Application, Account, Employment) and states: "Those with PPR categorization are only viewable to those with PPR access" (Professional Practices row's Notes) and "This is a remark viewable to MORE workers at the account level" (Account row's Notes). FDD 05 (System Admin - Credentialing, reviewed previously) separately described an admin-configurable "Internal Comment Group (i.e., visibility permissions)" with no further detail. | `communications-capability.md`'s `Comment` aggregate models visibility as a flat `Internal`/`External` enum (`COMMENTS.visibility`), with no category or additional-sensitivity concept; `communications-permissions.md` matches this — atomic `comments.view-internal`/`create-internal`/`create-external`/`edit`/`delete` permissions scoped to org hierarchy (Building/District/ISD, transitive), no "group" abstraction. `credentialing-domain.md`'s `APPLICATION_PROCESSING_NOTES` independently has only a flat `is_internal` boolean (per the FDD 05 review). | This FDD's own comment documents supply exactly the missing specificity the FDD 05 review lacked, and the answer favors the SDD's existing atomic-permission direction over FDD 05's vaguer "group" framing: there is evidence for **one** additional sensitivity gate (PPR content, gated by whether the viewer has PPR access) and **one** scoping dimension (Account-level, i.e. org-level, already covered by the existing Building/District/ISD scoping) — not a general multi-tier configurable visibility-group system. | **Resolved internally, not escalated to the client.** `credentialing-domain.md` Open Question #11 updated with this finding and a concrete recommendation (add a `category` label to `ProcessingNotes` for filtering; gate PPR-categorized notes behind Professional Practices' own restricted access model, e.g. an additional `profpractice.*`-style permission check, rather than a new `InternalCommentGroup` aggregate). Schema/permission drafting itself is tracked as Coverage Gap #2 below, not applied in this pass since it spans two domains' aggregates and the exact permission name is a design choice. See full Cross-Domain Question answer below. |
| 5 | FDD vs. SDD (terminology, positive match) | Manual-send customization does not alter the base template | `Emails and Remarks.md`, Application Evaluator role: "the evaluator may update the wording from the template for the specific email being sent to an applicant. THIS DOES NOT IMPACT THE ORIGINAL TEMPLATE WORDING IN THE LIBRARY." | `communications-sequences.md`'s "Manual Email Send with Template Customization" sequence: "Customization scope: User can modify subject/body but NOT template variables" and stores customizations only on the resulting `EmailInstance`, never on `EmailTemplate`. | Exact match — no drift. | Aligned — no action needed. Noted as a positive data point, same as the FDD 10 review's "Staffing Data Administrator" terminology match. |
| 6 | FDD vs. SDD | Historical email search criteria | Main FDD, Business Specifications: "The search criteria will be defined (by **individual, sponsor, school district**, etc.)"; also "types of communication (confirmation, greeting, questions, replies, etc.)" as a distinct filterable attribute. | `communications-api.yml`'s `GET /email-history` (`searchEmailHistory`) exposes only `functionalArea`, `recipientEmail` (exact match, not name search), `status`, and a date range — no organization/district filter, no applicant-name search, and no "communication type" classification anywhere in `EMAIL_INSTANCES`' schema. | Real, moderate gap — `EMAIL_INSTANCES` has no organization/district reference field at all, so filtering "by school district" isn't just a missing query parameter, it's a missing column. "By individual" is only partially covered (exact email match, not name search). "Type of communication" has no equivalent field or enum. | See Coverage Gap #3. |

## 3. Coverage Gaps (In Source Document, Not in SDD)

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | `Interface Design document - Microsoft 365 (Outlook).md`, entire document | A dedicated, two-way, non-PII Outlook/Graph API correspondence-mailbox channel, distinct from SendGrid's one-way templated notification channel — `communications-capability.md`'s Scope, Dependencies, and Integration Patterns sections mention only SendGrid; there is no Outlook/Graph API dependency, aggregate, or integration pattern anywhere in the domain. | `communications-capability.md` — new Dependency row (Microsoft Graph API) and Integration Pattern ("Outlook Two-Way Correspondence"), likely a distinct capability from `EmailInstance` since this channel is explicitly *not* templated/tracked the same way (no PII, human-authored, presumably not run through Variable Resolvers) | M | Not drafted — the IDD alone doesn't say which business workflows use this channel (the main FDD's Business Specifications never mention it), so drafting a full aggregate would be guessing. Flag as an Open Question (see below) rather than invent the shape. |
| 2 | `Comment Logs.md`'s "Remark Category" table (Professional Practices / Application / Account / Employment categories; PPR-specific access gate; Account-level scope) | A `category` concept on comments/notes for filtering and (for the PPR category specifically) an additional visibility gate beyond the existing `Internal`/`External` binary — see Discrepancy #4 and the full Cross-Domain Question answer below | `communications-capability.md`'s `Comment`/`COMMENTS` (add `category` field) and/or `credentialing-domain.md`'s `APPLICATION_PROCESSING_NOTES` (same); a new narrow permission gate for PPR-categorized notes, most likely built by requiring an existing Professional Practices permission rather than a new "visibility group" tier | H | Recommend drafting directly once it's decided which domain owns the `category` field (communications' generic `Comment`, since credentialing's `ProcessingNotes` looks like it should eventually just be an instance of the generic `Comment` aggregate rather than a parallel flat-boolean model — see Discrepancy #4 resolution note) |
| 3 | Main FDD, Business Specifications ("search criteria will be defined by individual, sponsor, school district"; "types of communication (confirmation, greeting, questions, replies, etc.)") | Organization/district-scoped email history search, applicant-name search (not just exact recipient email), and a "communication type" classification/filter — see Discrepancy #6 | `communications-capability.md`'s `EMAIL_INSTANCES` (add an optional `organization_code`/district reference, populated from the triggering domain event where applicable) and `communications-api.yml`'s `searchEmailHistory` (add `organizationCode` and `applicantName` query parameters); "communication type" may not need a new field if it maps 1:1 to `event_type`/template name — worth confirming before adding a redundant column | M | Not drafted — "sponsor" is undefined in the FDD (likely EPP-related, given "sponsor" also appears used for professional-learning/EPP program sponsors elsewhere in the corpus) and needs a light internal check against the EPP/organizations domains before modeling a field for it |
| 4 | Main FDD, Feature 6.2.1 Acceptance Criteria: "The Admin can select and insert tags including, but not limited to: `<<applicationNumber>>`, `<<denialReason>>`, `<<holdReason>>`, `<<fee>>`, and `<<paymentLink>>`" | These specific variable names aren't enumerated anywhere in `communications-technical-design.md`'s "Known Event Types and Their Variable Families" table — the mechanism (event-type-specific Variable Resolvers, extensible Variable Families) is correctly modeled and would support all of these, but `<<holdReason>>` and `<<paymentLink>>` in particular imply resolver logic (a "Hold" reason family, tied to credentialing's manual-review/hold states; a live per-application payment link, tied to `payments`) that isn't listed in the existing `CredentialApplicationApproved`/`CredentialApplicationDenied` resolver examples | `communications-technical-design.md` — extend the "Known Event Types and Their Variable Families" table with a `CredentialApplicationOnHold` (or similar) event type and its `HoldReason` variable family, and confirm `<<paymentLink>>` is sourced via the existing `payments` dependency (already listed generically, not down to the variable-name level) | L | Low priority — the mechanism already supports this; it's an enumeration gap, not a missing capability. Worth filling in once credentialing's Hold-state event names are finalized (ties to `credentialing-domain.md`'s existing "Requires Manual Review" / hold-adjacent states). |

## 4. Tagging

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| `6 - Alerts, Emails and Communication` §Business Specifications (alert create/modify/remove, versioning, Group/Title/Message/Description/Coded Inserts/Start-End Dates) | communications | `Alert` aggregate; `communications.alerts.create/edit/delete/view` | implements |
| `6 - Alerts, Emails and Communication` §Business Specifications, Feature 6.3.1 (email template CRUD, functional-area-restricted edit/delete) | communications | `EmailTemplate` aggregate; `communications.{fa}-templates.*` permission pattern | implements |
| `6 - Alerts, Emails and Communication` §Feature 6.2.1-6.2.3 (Coded Inserts / variables, merge order, action-item details) | communications | `TemplateVariable`, Variable Resolver / Variable Family model | implements (mechanism); see Coverage Gap #4 for enumeration gap |
| `6 - Alerts, Emails and Communication` §Feature 6.3.2 (rich text editor, ADA fonts, letterheads) | communications | `TemplateVersion.body_content`; "Template Editor Decision: Markdown with Custom Extensions" (technical-design) | implements — see Altitude #4 |
| `6 - Alerts, Emails and Communication` §Feature 6.3.3 (duplicate template validation) | communications | `communications-capability.md` Open Question #5 ("What constitutes a template 'duplicate'?") | informs open question #5 — FDD specifies "same Title or Subject within the same Group," which directly answers this; recommend closing that Open Question with this citation next time the doc is touched |
| `6 - Alerts, Emails and Communication` §Feature 6.3.4, Business Specifications (audit data: Modified Date/By, Version #, Begin/End Date) | communications | `TEMPLATE_EVENTS`; `TemplateVersion` fields | implements |
| `6 - Alerts, Emails and Communication` §Business Specifications (historical email search, view, resend with CC, embedded viewer) | communications | `EmailInstance`; Sequences: Email History Search and Details View, Email Resend with Recipient Override | implements — search-criteria gap noted (Coverage Gap #3) |
| `6 - Alerts, Emails and Communication` §Business Specifications ("without requiring a direct or two-way integration with a Microsoft Outlook client") | communications | (no Outlook dependency modeled) | conflicts with itself (see Discrepancy #1); gap — not yet modeled (Coverage Gap #1) |
| `6 - Alerts, Emails and Communication` §Business Specifications (6-month historical retention, admin-configurable) | communications | `content_retention_days` (default 90 days) | conflicts with (see Discrepancy #2) |
| `6 - Alerts, Emails and Communication` §Business Specifications (mass emails restricted to State/System Admins, not District users) | communications | `communications.email.send-adhoc` ("System Admin only") | implements |
| `6 - Alerts, Emails and Communication` §Feature 6.1.1-6.1.4 (To/CC, internal/external send, SendGrid integration, large volume) | communications | `EmailInstance`; Sequence: Event-Triggered Email Send / Manual Email Send | implements — see Altitude #3 |
| `6 - Alerts, Emails and Communication` §Feature 6.4.1-6.4.4 (comments in workflows, internal/external marking, remarks history table) | communications | `Comment` aggregate; `communications.comments.*`; Sequences: Add Comment to Credential Application, Include Comments in Outbound Email | implements (binary internal/external); gap — category/PPR gate not modeled (see Coverage Gap #2) |
| `6 - Alerts, Emails and Communication` §Feature 6.5.1-6.5.3 (reporting integration) | communications, reporting (unreviewed) | `communications.reports.view`; Dependency: reporting | implements |
| `Comment Logs.md` §Remark Category table (Professional Practices / Application / Account / Employment; PPR access gate) | communications, credentialing, profpractice | `Comment.visibility`; `APPLICATION_PROCESSING_NOTES.is_internal`; `credentialing-domain.md` Open Question #11 | resolves open question #11 (see Discrepancy #4); gap — category field not yet modeled (Coverage Gap #2) |
| `Emails and Remarks.md` §MORE Product Owner (template library, internal comments template library, event/role association) | communications | `EmailTemplate`; `PredefinedComment`/`PREDEFINED_COMMENTS`; `communications.predefined-comments.manage` | implements |
| `Emails and Remarks.md` §Application Evaluator (customize wording without impacting library template; Hold-action remarks + email) | communications, credentialing | Sequence: Manual Email Send with Template Customization; `Comment`/`CommentEmailHistory` | implements — exact match (see Discrepancy #5) |
| `MOECS_CommentsScreenShot.md` (remarks history popup on linked action) | communications, credentialing | `Comment`; Sequence: Add Comment to Credential Application | informs — corroborates, no new content |
| `Interface Design document - Microsoft 365 (Outlook)` (entire document) | communications | (no equivalent dependency/integration pattern) | gap — not yet modeled (see Coverage Gap #1); informs (see Altitude #1) |
| `Interface Design document - SendGrid` (entire document) | communications | "SendGrid Email Delivery" integration pattern (`communications-capability.md`); `communications-technical-design.md` SendGrid Rate Limiting section | resolved — rate limit corrected (see Discrepancy #3); informs (see Altitude #2) |
| **Cross-domain: FDD 05's "Internal Comment Group" / `credentialing-domain.md` Open Question #11** | credentialing, communications, profpractice | `credentialing-domain.md` Open Question #11; `communications-capability.md` `Comment.visibility` | resolved — see Discrepancy #4 and Cross-Domain Question below |

---

## Open Questions Raised by This Review

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | Which business workflows actually use the Outlook/Graph API two-way correspondence channel described in the Outlook IDD (e.g., is it used for anything today, or reserved for a future feature)? The main FDD's Business Specifications never mention it, so its scope and triggering workflows are unknown. | Internal (Beth Dontje / Caitlin Groom per the IDD's own contacts, if not resolvable from other FDD drops) | Open |
| 2 | Is "sponsor" (main FDD's email-history search-criteria list: "by individual, sponsor, school district") an EPP/professional-learning program-sponsor concept, or something else? Not defined anywhere in this FDD. | Internal (EPP/proflearning domain owner, once those FDDs are reviewed) | Open |
| 3 | Should the communications-side `content_retention_days` default be reconciled to the FDD's stated 6-month figure, or is 90 days a deliberate SDD-side choice worth documenting as such? | Internal | Open |

Note: none of these three clear the `client-questions.md` judiciousness bar. #1 and #2 are
internal cross-domain/scope questions resolvable by reading other FDD drops (EPP's own FDD
for "sponsor"; whichever document, if any, actually specifies the Outlook channel's use
cases) rather than needing the client to re-explain their own documents. #3 is a
non-blocking configuration default with no behavioral consequence either way. The
comment/remark visibility question (Discrepancy #4) is explicitly **not** listed here — it
was resolved by this review's own reading of `Comment Logs.md`, per the Cross-Domain
Question answer below, and `credentialing-domain.md` Open Question #11 has been updated
accordingly rather than left open.

## Related Documents

- Source document(s): `functional-design-docs/06 - Alerts, Emails, Communications/6 - Alerts, Emails and Communication.md`; `functional-design-docs/06 - Alerts, Emails, Communications/Comment Logs.md`; `functional-design-docs/06 - Alerts, Emails, Communications/Emails and Remarks.md`; `functional-design-docs/06 - Alerts, Emails, Communications/Interface Design document - Microsoft 365 (Outlook).md`; `functional-design-docs/06 - Alerts, Emails, Communications/Interface Design document - SendGrid.md`; `functional-design-docs/06 - Alerts, Emails, Communications/MOECS_CommentsScreenShot.md`
- Noted, out of scope, not converted (per tracker, unchanged): `06 - Alerts, Emails, Communications/101022 Email notifications research (1).pptx`
- SDD documents reviewed against: `solution-areas/communications/communications-capability.md` (updated), `solution-areas/communications/communications-permissions.md`, `solution-areas/communications/communications-sequences.md`, `solution-areas/communications/communications-technical-design.md` (updated), `solution-areas/communications/communications-api.yml`; cross-domain excerpts of `solution-areas/credentialing/credentialing-domain.md` (`APPLICATION_PROCESSING_NOTES`, Open Question #11, updated) and the prior [05-system-admin-credentialing](05-system-admin-credentialing.md) review (Coverage Gap #6, the original "Internal Comment Group" finding this review follows up on)

---

## Cross-Domain Question: Does the comment/remark visibility model need a "Comment Group" abstraction, or is atomic permissioning sufficient? (asked directly of this review)

**Short answer: atomic permissioning is sufficient. No generic "Comment Group"/visibility-tier
abstraction is warranted by the evidence in this FDD drop.**

The FDD 05 review flagged that "Internal Comments for credential processing" have a
configurable "Internal Comment Group (i.e., visibility permissions)" but the source text was
too thin to say what the tiers actually are, or whether a genuinely more-sensitive category
(e.g., PPR-related) exists distinct from general processing notes. This review's two comment-
specific documents answer that directly:

- **`Comment Logs.md`'s "Remark Category" table** lists exactly four categories in practice —
  Professional Practices, Application, Account, Employment — used primarily to organize/filter
  remarks by what they're about, not as a general permissions taxonomy. Three of the four
  (Application, Account, Employment) carry no stated access restriction beyond the ordinary
  internal/external distinction already in the SDD.
- **Exactly one category carries an explicit additional restriction**: the Professional
  Practices row's Notes column states, verbatim, "Those with PPR categorization are only
  viewable to those with PPR access." This is a **domain-boundary access check** (does the
  viewer have Professional Practices' own restricted access), not a configurable "visibility
  group" that an admin assigns arbitrary named tiers to.
- **The Account category's restriction is a scope statement, not a new tier**: "viewable to
  MORE workers at the account level" describes organizational scoping — exactly the
  Building/District/ISD (transitive) scoping model `communications-permissions.md` already
  applies to comment permissions, not a new mechanism.
- **`Emails and Remarks.md`** (the other comment-specific document) describes comment
  *authoring* workflows (a MORE Product Owner curates an "internal comments template
  library"; an Application Evaluator creates comments and can attach one to a "Hold" action,
  optionally emailing it) — it says nothing about visibility tiers at all, and its
  template-library concept maps cleanly onto the SDD's existing `PredefinedComment`
  aggregate.

**Conclusion for the open design question:** don't build a generic, admin-configurable
`InternalCommentGroup`/visibility-permission aggregate. Model this as: (1) a `category` label
on comments/notes, used for filtering and UI organization only (Application, Account,
Employment, Professional Practices, extensible), with (2) exactly one additional access gate —
PPR-categorized comments require the viewer to also hold Professional Practices' own
already-restricted access (most naturally, an existing `profpractice.*`-style permission, or a
narrowly-scoped new permission like `credentialing.application.comment.view-ppr` that checks
the same underlying access rule PPR itself uses) — rather than PPR content living inside a
generic multi-tier group system. This is consistent with the IAM "User Group" precedent the
repo owner has pushed back on elsewhere (Open Question #2 in `client-questions.md`): a named
"group" abstraction isn't needed where the real distinction is a single additional permission
check plus organizational scoping the platform already has.

**Does this resolve `credentialing-domain.md` Open Question #11?** Yes — updated in place in
this review (see the Discrepancies table, item #4, and the file itself). It does **not**
resolve the exact schema mechanics (whether `category` lives on `communications`'s generic
`Comment` aggregate with `credentialing-domain.md`'s `ProcessingNotes` eventually becoming an
instance of it, or whether `ProcessingNotes` grows its own parallel `category` field) — that's
tracked as Coverage Gap #2, a genuine but narrow follow-up, not an open question needing the
client.

## Process/Template Friction Noted

- This review is the first to compare a functional-area FDD against `communications` as the
  *primary* reviewed domain (prior reviews treated it as a recurring cross-domain dependency
  only). The main FDD's own Dependencies section lists eight other FDDs (5, 9, 10, 11/16, 12,
  13, 14, 15) as the actual source of "content management and creation" for alerts/templates —
  meaning this FDD describes the *mechanism* (how alerts/templates work) while each functional
  area's own FDD describes *what* gets sent. This split held up well against the SDD, which
  mirrors it closely (a generic `EmailTemplate`/`Alert` mechanism, functional-area-scoped).
  Worth noting for future reviews of the FDDs this one depends on (12, 13, 14, 15 — System
  Admin for MiEdWorkforce/EPP/Professional Learning, and Identity Management Integration): each
  will likely reference this FDD's alert/template mechanism the same way FDD 05, 09, and 10
  already have, without adding new communications-domain content of their own.
- As with the FDD 07 and FDD 10 reviews, this review's most consequential finding (the
  SendGrid rate-limit correction) came from cross-referencing an "obviously boundary-violation"
  IDD against the SDD's own stated Open Question, not from the main FDD narrative itself —
  reinforcing the standing note from those reviews that IDDs, even when correctly flagged as
  out-of-altitude for the functional spec, often contain load-bearing facts the SDD's
  technical-design authors haven't cross-checked. Recommend a lightweight habit: whenever a
  technical-design doc states an external constraint (rate limit, timeout, quota) without
  citing a source, check whether the corresponding vendor IDD already answers it before leaving
  it as an assumption or an Open Question.
