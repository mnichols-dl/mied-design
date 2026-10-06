Comments around FDD review

Note, here "AO" refers to our UI/UX teaming partners, and "POs" refers to the key business product owners on the client's side.

Expected next steps:

1) update the MiEdWorkforce solution design to accurately account for a reasonable change with no pause (e.g. just missed one workflow or sequence or property on a domain object)
2) identify a question to resolve with the POs. This might be a suggestion for improvement of UX, or a clarification needed about what the actual goals are (e.g. maybe the story was too vague or unclear what the intent was). These should be tracked somewhere in the repo too, probably in a PO feedback markdown doc.
3) propose removing a story or requirement. (same as 2 just an affirmative recommendation from us rather than request for more input)
4) flag something that needs deeper design work and POC work to concretely propose a way to implement it (may incur the need for new stories to be created if the POs like it)

Cross cutting notes first
  - Some things may be duplicated in the FDDs, in particular I suspect "worklists" is one of the most frequently restated and redefined. We may want to do a pass to spotlight any stories or leaky duplication like this so the POs can decide whether they want to proceed. Email templates is another. Reports as well.

FDD-specific comments
- FDD 01 - User Management
  - Impersonation: May need to review areas where they suspected impersonation was necessary (always readonly? existing views enough?)
    - Need to review SDD items and ensure there are no lingering impersonation references if designed away
  - Groups: I believe this is all synonymous with "roles", as long as they can see users with a given role, is that enough?
  - The technical implementation tends toward transitivity (ISD to District to Building in particular) -- is this acceptable? (Should we make certain permissions transitive vs intransitive?)
  - (internal) Does a state user need to log in the first time for another state user to grant them a role? If not, how can we validate that the user exists and is active? (e.g. search?) Is that workflow adequately described in sequences?
  - (AO) Screens exist showing granular permission selection when granting a user access e.g. for an EPP. This seems extremely tedious. We should discuss further with the POs if they want fixed and straightforward roles based on user input (more consistency in docs and conversation), fixed roles PLUS optional custom ones (and at what level does someone manage this), or fully dynamically managed roles
- FDD 02 - Reporting
  - Need to confirm assumption that it seems the MiEd app integration here is simply to hold a reference to a Power BI report and pass through the authorization information.
  - Question now is how does report management tie in to user permissions? They mention entity/group/role... is "credentialing.reports.view" and "credentialing.reports.manage" enough?
  - Versioning workflow should be clear in POwer BI and in application. If they want to preview a report before making it broadly available, should we include a permission check for viewing unpublished reports? Is the report technically "published" in Power BI just not surfaced on the UI? I think the answer here is probably yes and that all report usage is done THROUGH MiEd. This would lend itself well to better flexibility for things like active dates for reports and such. One point of entry for the general public. Admins can see more.
  - Do we need a deeper spec on Power BI embedded report row-level filtering and such?
- FDD 03 - Dashboards
  - Inconsistencies found between functional area widget ownership and such. Need to check in with AO on this. Is this a separate "functional area" or simply a technical pattern in the UI that surfaces different summary counts and UI areas? I lean toward it just being a UI concept.
- FDD 04 - Business Rule Management
  - Is the "BRE" here truly synchronously involved in user-driven workflows for things like credentialing, or is it always a passive "engine" that just reuses some of the same logic or configuration?
  - Related to this is "Question Sets". This is more of a blend of UI concepts, configurability, and potentially related to the above question. May want to bring Question Set workflow POC demo back to AO and POs to discuss further.
  - In terms of business rules too... we will have versioned rules, but how do we handle historical data? Is this a massive risk area full of scope creep?
- FDD 05 - Credential Admin
  - Are credential types inbuilt and hardcoded as intrinsic types in the design, or are they all truly "dynamic" and all rules around them dynamic?
  - Relates to Question Set ideas and POC I put together. Essentially that requirements on a cred have a standardized way of attempting to resolve through data integrations, uploaded documents, or resorting to manual question set response review. Main goal is to not ask questions we have an answer to.
  - Why are "batch jobs" here? They're just kind of generic....
  - I think we may want another review pass here.
- FDD 06 - Communications
  - Should email templates be freely HTML, or is markdown support good enough?
  - Do we need to sign off that Outlook will not be used provided that SendGrid and application state may include satisfactory audit-trail email history?
  - Is "two-way" comms out of scope here?? Seems likely, I would guess that a shared inbox would be there for support staff but that it doesn't and shouldn't matter here?
  - I'm not sure I fully follow what kind of "alerts" they are envisioning here that might be dynamically created and managed in the app's UI. Should elaborate more on that.
- FDD 07 - Payments
  - Do we have even things like "paying on behalf of a person" for district users modeled here?
- FDD 08 - Interfaces
  - This area is generally cross cutting and not a "functional area" like most of the others...
  - This should more or less map on to the external systems in our solution design spec I think. That will give a better idea. Ways that things are integrated with should be on that external system's node, but we should have references from sequences to the external systems to define expected interaction purposes and workflows (e.g. business value added)
- FDD 09 - PPR
  - Is "calculated compliance" vs "stateful flag" agreeable to the POs? I'm thinking maybe the "flag" is just a visual thing and that we should reframe their language from "sets a flag" to "is flagged" or something, so it is less prescriptive and leaking technical implementation.
  - Is disclosure here something we should model with Question Set logic?
  - Is the "escalation" flow here elegant? Do we have full line of sight on how a PPR reviewer can request an escalation? Shoiuld there be comments with the escalation option and such?
  - Invariant alert on the whole "stop-check" thing they mention.
- FDD 10 - Staffing
  - May need to fill the bulk upload workflow on the backend side and see if AO has any new wireframes on this process
  - How much fo this FDD is open? Seems things like historical data connections and conversions?
  - This seems quite heavily related to the MiDataHub integration via Ed-Fi API. Districts may submit this via file upload (I think? described here it seems), Ed-Fi API (separate doc with SME filling out mappings and such), and UI entry too.
  - User Story 10.2.3 seems insane. "Auto-generate CEDS-aligned data definition documents"? Okay, but if the CEDS standard changes in a way that impacts a field the application depends on.... Would recommend asking more about what the goal is here, what this data definition document would look like, if a file template produced by the system for users to upload using is enough.
  - Also sounds like we have a distinction about document types worth following up on. Here there is mention of admins uploading documents that will be presented to users. Different interaction mode entirely than user submitting a personal document for review.
  - Lots of vaguery around "defining and managing data requirements"... What exactly does "data categories" mean here? How would a user technically accomplish this in our browser-based app without a stupid amount of scope being added?
  - I suspect some of these requests were drafted with JSON-LD and SHACL in mind. E.g. "a list of validation codes and text" sounds like "SHACL shapes being used to produce documentation"
  - Should we NOT have user-facing UI actions for file processing and resets and such? Is this always and only to be a backend system admin task or dev task?
- FDD 32 - Technology, Standards and Auditing
  - This FDD is really two unrelated documents stapled together under one number: a set of NFR/standards placeholders (32.1, 32.3-32.10, 32.12) with no real requirements, and a set of substantive functional features (32.11 data migration, 32.13 reporting, 32.14 batch processing, 32.15 worklist, 32.16 document management) that arguably belong as their own FDDs or under existing ones.
  - 32.5-32.10 are all titled "Software" with no distinguishing content beyond a parenthetical subtitle in the addendum (Frameworks, Industry Standards, Licensing, Security Updates, Error Reporting, Integration Capability) - functionally one NFR section split into six feature IDs for no clear reason. Recommend collapsing to one.
  - 32.1 is duplicated as a feature ID (used for both "Application standards" and, seemingly by typo, one of the "Software" rows meant to be 32.10).
  - 32.3 (Auditing) is explicitly marked "no longer considered necessary" but still occupies a feature slot - recommend removing rather than leaving a dead placeholder.
  - 32.13 (Reporting) is explicitly a restatement of FDD 02 ("user stories defined under Epic 2 - Reporting represent the detailed functional implementation... of Feature 32.13") - confirms the cross-cutting duplication concern above. Should just be removed here and referenced, not restated.
  - 32.15 (Worklist) duplicates content likely already captured per-functional-area (it cross-references FDD 05.5 and FDD 13.3 directly in the addendum) - another instance of the "worklist repeated across FDDs" problem flagged in cross-cutting notes. Recommend consolidating worklist behavior into one canonical spec referenced elsewhere, not restated as its own feature with 9 near-identical "create and manage the worklist for X" stories.
  - 32.4 (Database) locks in implementation/vendor detail inside a requirement ("CEDS IDS... SQL Server... may convert to NoSQL... JSON-LD... likely Azure Cosmos DB") while simultaneously saying the state hasn't decided. This reads as scope/tech creep from the RFP response bleeding into the FDD rather than a stable requirement. Recommend rewriting as "primary data store must conform to CEDS for [specific data domains]" and moving all vendor/schema-format speculation to an architecture decision record.
    - Deeper concern: JSON-LD/SHACL/Cosmos is being proposed for the *whole* data model, but that standard is really about the interoperable/shareable subset (educator identity, certification, employment history) - not internal operational state (draft EPP recommendations, PPR review workflow state, worklist assignments, document metadata). Recommend proposing a two-layer model (internal operational store + CEDS-conformant published/exchange layer) and get that split explicitly agreed with the POs/leadership before it's assumed as one data model end-to-end.
  - Several "requirements" are pseudo-user-stories for things that aren't really user-driven asks - they're mandates dressed as stories (e.g. 32.2.1 "As a Developer, I want the application to adhere to State of Michigan Digital Standards," 32.16.1 "As a System Administrator, I want the system to allow for integration with malware scanning tools"). No actor is actually choosing this; it's a constraint. Inconsistent with 32.1/32.3-32.10 which correctly opt out of the user-story format as "guideline rather than functional deliverable" - 32.2 and 32.16.1 should get the same treatment.
  - Several requirement statements just restate the feature title without a testable rule (e.g. 32.16 "must allow for integration with malware scanning tools" - allow how, required at upload time or just architecturally possible? 32.13 "must integrate with analytics tools such as Power BI and SAS" - what does "integrate" mean operationally?). The acceptance criteria under 32.16.9 actually nail this down (scan before storage) - the top-level requirement language should match that precision instead of vague restatement.
  - Recommendation: split this FDD into (a) a short NFR/Standards & Compliance reference (32.1, 32.3-32.10, 32.12, non-CEDS parts of 32.4) written as plain constraints with no story format and no vendor-specific detail, and (b) fold 32.11, 32.14, 32.15, 32.16 into their own FDDs or merge into the functional areas they actually belong to, removing 32.13 entirely in favor of FDD 02.
- 