# RTM comparison against the solution design

Prepared 2026-10-06. Read-only analysis of `client-inputs/rtm/MiEdWorkforce Requirements Traceability Matrix.xlsx` against the graph (`design/graph/MiEdWorkforce.ttl`) and the working docs under `design/working-docs/solution-areas` and `design/working-docs/solution-level`. Open Questions and the technical-design docs are out of scope, with one exception noted in section 4 (a technical-design statement is cited where it is the only record of a decision). Companion file: `rtm-mapping.csv` (one row per RTM requirement row).

Standing stance applied throughout: our solutioning is a hard preference. This report lists differences. It does not propose changing our design, and the mismatches in section 4 are reported for awareness, not for reconciliation.

## 0. Headline findings

1. The RTM is a faithful restatement of the FDD stories we already hold in the graph. Of 676 requirement rows, 620 match a graph `sd:Requirement` or `sd:UserStory` node with an exact or near-exact statement (604 of those at text similarity 0.95 or higher). The join key is the FDD feature number (RTM "Feature ID", for example 24.4) plus row order inside the feature; our story ids (24.4.1, 24.4.2) follow the same order.
2. The RTM carries no requirement ids. The "Req ID" column (Azure DevOps story id) is blank on all 676 rows, and the "FDD Reference" column is a hyperlink to a whole FDD document, not to a section. Any RTM id we attach to the graph has to be a composite key until the DevOps ids are supplied (section 7).
3. About 24 RTM rows have no FDD story in our graph at all (RTM-only rows: split stories in 5.5, the 12.4 data dictionary, 17.2 permit feedback, 21.7 audit worklist, 10.16, 15.12, 18.6). A further 83 of our 736 graph requirement nodes (mostly unnumbered "Business Specification" items, and FDD 17 permit sub-flows) have no RTM row.
4. Design coverage is thinner than requirement coverage. Of the 676 rows, 337 trace to at least one design element with no open finding, 70 trace to design elements but carry an open finding (coverage gap, discrepancy, contradiction), 218 trace to a graph story that nothing in our design satisfies, 34 (mostly tab 08 integrations) are covered only by a section of `solution-integrations.md`, and 17 are pointer rows. Whole features with no design element: dashboards (tab 03), business rules and question sets (tab 04), data quality (tab 19), Rapback detail features 30.5, 30.7, 30.8, batch administration (5.4, 32.14), sponsor application (26.1), the public-portal admin features (12.2, 12.3), and several external interfaces.
5. 153 RTM rows reach one or more sequences through `satisfiedBy`; with 28 further sequence proposals (low confidence) 181 rows carry a sequence. 87 of our 142 sequences are reachable from an RTM row through `satisfiedBy`; the 28 proposals bring that to 115, leaving 27 sequences with no RTM row.
6. Real differences between how we solutioned and what the RTM states are concentrated in 29 places (section 4). The largest: no User Group layer (tab 01), worklists treated as a UI pattern rather than admin-managed objects (32.15, 13.3, 14.3), per-user customizable dashboards, admin-configurable business rules, sponsor applications (26.1, 28.1), the Rapback Admin actor (all of tab 30), rich-text email editing (6.3), the June 30 flag reset (9.1), and MTTC failure timing (11.8).
7. RTM quality issues worth sending back to the BA are in sections 5 and 6: tab 04 is headed "Dashboards", Feature IDs lose their trailing zero (10.10 shows as 10.1; also 11.10, 15.10, 32.10), a row title that does not match its text (5.1 r9), a stray editing tag in 27.3, and many UI-level or vendor-named stories.

## 1. Method and heuristics

Everything below was produced by scripts over the parsed workbook and the graph; judgement was applied only to the ambiguous rows listed in the CSV notes.

- RTM parse: every epic tab read cell by cell. A row counts as a requirement row when it has a title or a description. Feature id and feature name carry forward from the merged feature cell. Feature ids that Excel stored as numbers lost a trailing zero (a second "10.1" after 10.9 is really 10.10); these were repaired by position (10.10, 11.10, 15.10, 32.10).
- RTM id used in this report and in the CSV: `<tab>/<feature id>/r<excel row>`, for example `07/7.1/r5`. It is only a locator for this analysis; the workbook has no row ids.
- Graph side: `sd:Requirement` (217), `sd:UserStory` (510) and `sd:NonFunctionalRequirement` (9) were treated as one pool of 736 FDD requirement nodes; their `sd:requirementId` is the FDD story number. `sd:satisfiedBy` supplies the design links (Sequence, ApiOperation, Aggregate, BusinessRule, Permission, and others). `sd:Annotation` nodes with a `findingType` (coverage-gap 120, discrepancy 40, out-of-scope 34, unconfirmed-assumption 20, rejected 12, contradiction 4) supply the open findings.
- Matching: tokenised, lightly stemmed, TF-IDF cosine similarity between the RTM description and the graph story statement (the RTM title was not used because titles are rewritten by the BA). Candidate pool: graph requirements with the same FDD number as the RTM feature. A bonus of 0.05 when the feature id also matches. Greedy one-to-one assignment in score order, threshold 0.2; a second pass allowed reuse of an already-assigned graph node at 0.35 or higher. exact means similarity of 0.8 or more; anything lower is partial. 17 rows were set by hand (each has a reason in the CSV note).
- Rows that are notes or pointers (title "N/A" or blank; 35 rows) were mapped to the feature-level graph node where one exists.
- Rows with no graph requirement were searched against the title and summary text of every sequence, operation, aggregate, rule, batch job and technical process (cosine 0.2 or higher) and, for tab 08 and a few other rows, given the matching `solution-integrations.md` section. These are low confidence and marked partial.
- Sequences: taken from the matched requirement's `satisfiedBy` (ranked by text overlap with the RTM row, up to 4 listed). 28 further sequence proposals came from keyword overlap and judgement; these rows say so in the note and are low confidence. A sequence usually serves several RTM rows (mean 2.0, maximum 9 for Open New Collection).
- Confidence: high means the requirement text matched at 0.9 or more and the top linked sequence also overlaps the RTM text; medium means a good requirement match but links only through `satisfiedBy`, or a shared requirement; low means a hand override, weak text, keyword-only, or a proposed sequence.
- CSV extra columns after `note`: `rtm_sheet`, `rtm_feature_id`, `rtm_row`, `fdd_story_id`, `req_text_score`, `design_coverage` (linked, linked-open-finding, no-design-link, doc-only, n/a), `open_findings`, `rtm_ui_status`, `rtm_status`. In the CSV `our_artifact_kind` and `our_artifact_ref` are parallel semicolon lists (first entry normally the Requirement, then Sequences, then other elements, capped at 4 sequences and 5 others). match_kind describes the match between the RTM row and the artifacts listed, so a row can be exact to a Requirement node while `design_coverage` says nothing in the design satisfies it.
- Limits: `satisfiedBy` coverage is only as complete as the FDD review passes made it. "Unreached" design elements below mean "not linked to an RTM-matched requirement", which overstates the true gap for aggregates and operations (stories are often linked only to sequences). Where it matters, the text says so.

## 2. (a) How the RTM is structured

Workbook: 33 sheets.

| Sheet | Content |
|---|---|
| Introduction | Purpose, how to navigate, field descriptions. Says each epic tab holds requirements mapped to FDDs, design references and test cases, and that each requirement is added once approved by the product owners. |
| Target Dates | 69 rows of FDD groups (epic or feature ranges) with Submitted date (21 distinct dates from 2025-11-07 to 2026-02-20), Reviewed by SOM (all 69 filled), SOM Comments (68), BA Comments on suggested FDD edits (62), BA Comments on user stories (19). The footer says 70 FDD groups. A submission and review log, not a requirement list. |
| 01 to 22, 24 to 32 | 31 epic tabs, one per FDD (there is no tab 23). Each tab covers the FDD with the same number. |

Epic tab layout (identical on all 31 tabs, 19 columns, two header rows): Feature ID, Feature, Req ID, Requirement Title, Requirement Description, FDD Reference, Dependency, Status, Status Date, UI Design Status, Figma Reference, DEV, UAT, PROD, Test Case ID, Test Case Reference, Bug Reference, Priority, Requirement Change/Modification.

Counts: 676 requirement rows (one per user story), in 196 feature blocks; 179 blocks have at least one story and 17 are feature-only rows with no stories (10.14, 15.7, 15.13, 15.14, 15.15, 28.2, 28.5, 30.3, 32.1, 32.4 to 32.10, 32.12, 32.13). 35 requirement rows have title "N/A" or no title: 13 scope pointers to Business Specs 2, 3 and 6 ("underlying logic is derived from..."), 7 dashboard widget catalog placeholders ("to be determined", tab 03 features 3.1 to 3.7), 5 "no longer needed" (13.7, 19.5, 26.4, 31.3, 32.3), and 10 cross-references or notes (for example 1.7, 9.6 "Merged with 9.1", 11.4, 13.2, 22.4).

Rows per tab: 01 27, 02 6, 03 14, 04 20, 05 34, 06 18, 07 10, 08 49, 09 22, 10 23, 11 18, 12 20, 13 16, 14 15, 15 43, 16 8, 17 16, 18 49, 19 21, 20 4, 21 24, 22 20, 24 11, 25 11, 26 21, 27 11, 28 19, 29 36, 30 19, 31 39, 32 32. Tab 08 holds three MTTC rows that belong to feature 11.8 (rows 49 to 51); the FDD 8 sub-documents (8.1 to 8.9) are in tab 08.

Id scheme: there is none at row level. Feature ID is the FDD feature number (for example 5.2). Req ID, described in the Introduction as the unique Azure DevOps user-story id, is empty on all 676 rows. Row identity is therefore feature plus row order plus title. Four Feature IDs were truncated by Excel (see section 1).

How it links to the FDD sections: the FDD Reference column holds a hyperlink labelled "Link to <FDD name>" to the whole FDD Word document on SharePoint (615 of 676 rows; 61 blank). There is no section, page or story anchor, so the real link to an FDD story is Feature ID plus order. The Dependency column (606 rows filled, 31 contain "-") lists other epics as "<tab number> - <tab name>" followed by a parenthetical reason in prose.

Status columns: the Introduction defines Status as New, In Dev, In UAT, In Prod, Closed. In practice Status holds only "FSD Addendum Added" and only on 22 rows, all in tab 09; it is blank on the other 654. Status Date, PROD, Test Case ID, Test Case Reference, Bug Reference, Priority and Requirement Change/Modification are blank on every row. DEV and UAT say "Not Started" on the first row of each of 34 features and are otherwise blank. UI Design Status is filled on 179 rows: Not Started 59, Visual Design 45, Sandbox 33, N/A 28, Wireframes 7, Covered 7. Figma Reference holds five values (one "Link to Spec 1" and four "Working Page"). In short, the matrix is structurally complete but it is a requirements and design-status register; it has no delivery or test data yet.

## 3. (b) How RTM requirements map to our artifacts

### 3.1 Totals

- 620 rows exact, 47 partial, 9 none (match_kind).
- Design-level view (`design_coverage` in the CSV): 337 linked with no open finding, 70 linked with an open finding, 218 story-only (the graph holds the story, nothing satisfies it), 34 doc-only, 17 pointer rows.
- 153 rows trace to one or more sequences through `satisfiedBy`; 181 including the 28 low-confidence sequence proposals.

### 3.2 Coverage by RTM tab

| RTM tab and epic | Rows | exact | partial | none | high | medium | low | design linked | linked, open finding | story only, no design | doc section only | pointer row | with sequence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 01 User Management | 27 | 25 | 2 | 0 | 15 | 11 | 1 | 11 | 4 | 10 | 2 | 0 | 9 |
| 02 Reporting | 6 | 5 | 1 | 0 | 3 | 3 | 0 | 4 | 1 | 1 | 0 | 0 | 4 |
| 03 Dashboards | 14 | 14 | 0 | 0 | 14 | 0 | 0 | 0 | 0 | 14 | 0 | 0 | 0 |
| 04 Business Rules Management (tab header says Dashboards) | 20 | 19 | 1 | 0 | 19 | 1 | 0 | 0 | 0 | 20 | 0 | 0 | 0 |
| 05 System Admin Credentialing | 34 | 26 | 8 | 0 | 13 | 14 | 7 | 14 | 7 | 13 | 0 | 0 | 4 |
| 06 Alerts, Emails, Communications | 18 | 17 | 1 | 0 | 2 | 16 | 0 | 11 | 6 | 1 | 0 | 0 | 6 |
| 07 Credit Card Payments and Refunds | 10 | 10 | 0 | 0 | 7 | 3 | 0 | 9 | 1 | 0 | 0 | 0 | 7 |
| 08 Interface and Integration Setup | 49 | 48 | 1 | 0 | 32 | 16 | 1 | 15 | 4 | 3 | 27 | 0 | 11 |
| 09 Professional Practice | 22 | 21 | 0 | 1 | 12 | 10 | 0 | 19 | 1 | 1 | 0 | 1 | 12 |
| 10 Staffing Administration | 23 | 18 | 5 | 0 | 4 | 18 | 1 | 15 | 3 | 5 | 0 | 0 | 1 |
| 11 Educator Credentialing | 18 | 15 | 2 | 1 | 5 | 13 | 0 | 10 | 3 | 4 | 0 | 1 | 6 |
| 12 System Admin Users | 20 | 15 | 2 | 3 | 16 | 3 | 1 | 2 | 0 | 14 | 0 | 4 | 2 |
| 13 EPP Admin | 16 | 14 | 1 | 1 | 9 | 6 | 1 | 9 | 1 | 5 | 0 | 1 | 7 |
| 14 Professional Learning Admin | 15 | 15 | 0 | 0 | 7 | 8 | 0 | 7 | 1 | 7 | 0 | 0 | 0 |
| 15 Identity Management Integration | 43 | 42 | 1 | 0 | 23 | 19 | 1 | 32 | 2 | 3 | 5 | 1 | 26 |
| 16 Credential Application Processing | 8 | 8 | 0 | 0 | 2 | 6 | 0 | 5 | 1 | 2 | 0 | 0 | 2 |
| 17 Temporary Credentials | 16 | 9 | 5 | 2 | 3 | 8 | 5 | 5 | 5 | 1 | 0 | 5 | 4 |
| 18 Roster of Positions | 49 | 48 | 0 | 1 | 34 | 15 | 0 | 29 | 0 | 19 | 0 | 1 | 17 |
| 19 Data Quality Review | 21 | 16 | 5 | 0 | 12 | 8 | 1 | 5 | 0 | 16 | 0 | 0 | 2 |
| 20 DQ Comments and Flags | 4 | 4 | 0 | 0 | 0 | 4 | 0 | 1 | 3 | 0 | 0 | 0 | 1 |
| 21 Roster of Employees | 24 | 21 | 3 | 0 | 9 | 12 | 3 | 16 | 2 | 3 | 0 | 3 | 10 |
| 22 Employee Assignment Details | 20 | 20 | 0 | 0 | 3 | 17 | 0 | 13 | 4 | 3 | 0 | 0 | 2 |
| 24 Request for ID Collection | 11 | 10 | 1 | 0 | 4 | 7 | 0 | 10 | 0 | 1 | 0 | 0 | 4 |
| 25 EPP User | 11 | 11 | 0 | 0 | 11 | 0 | 0 | 8 | 3 | 0 | 0 | 0 | 11 |
| 26 Professional Learning Sponsor | 21 | 20 | 1 | 0 | 12 | 9 | 0 | 13 | 0 | 8 | 0 | 0 | 5 |
| 27 Professional Learning Educator | 11 | 11 | 0 | 0 | 9 | 2 | 0 | 5 | 1 | 5 | 0 | 0 | 6 |
| 28 Professional Learning Processing | 19 | 19 | 0 | 0 | 12 | 7 | 0 | 10 | 0 | 9 | 0 | 0 | 3 |
| 29 Certificates | 36 | 35 | 1 | 0 | 10 | 25 | 1 | 27 | 4 | 5 | 0 | 0 | 8 |
| 30 Rapbacks | 19 | 18 | 1 | 0 | 15 | 3 | 1 | 3 | 2 | 14 | 0 | 0 | 3 |
| 31 Public Search | 39 | 38 | 1 | 0 | 15 | 23 | 1 | 14 | 9 | 16 | 0 | 0 | 0 |
| 32 Technology, Standards, Authorizations | 32 | 28 | 4 | 0 | 20 | 11 | 1 | 15 | 2 | 15 | 0 | 0 | 8 |
| **Total** | 676 | 620 | 47 | 9 | 352 | 298 | 26 | 337 | 70 | 218 | 34 | 17 | 181 |

"Story only, no design" counts rows whose FDD story is in the graph but has no `satisfiedBy` link. "Doc section only" counts tab 08 interface rows that are described in `solution-integrations.md` (with several `[NEEDS INPUT]` markers) but have no graph element behind them. "Pointer row" counts rows that are notes or cross-references.

### 3.3 Coverage by our solution area

The area is taken from the area of the linked design elements (the most common one); where nothing is linked it falls back to the area that owns the FDD. Areas marked "no area doc" are bounded contexts that exist in the graph as empty stubs (dashboards, dataquality, businessruleengine, publicportal) or are staged outside the solution-areas folder.

| Our area | Rows | exact | partial | none | high | medium | low | design linked | linked, open finding | story only, no design | doc section only | pointer row | with sequence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| business rules (no area doc) | 20 | 19 | 1 | 0 | 19 | 1 | 0 | 0 | 0 | 20 | 0 | 0 | 0 |
| communications | 30 | 27 | 3 | 0 | 2 | 28 | 0 | 21 | 8 | 1 | 0 | 0 | 10 |
| credentialing | 124 | 111 | 10 | 3 | 43 | 73 | 8 | 64 | 18 | 35 | 1 | 6 | 26 |
| credentialing/iam | 4 | 0 | 1 | 3 | 3 | 0 | 1 | 0 | 0 | 0 | 0 | 4 | 0 |
| dashboards (no area doc) | 20 | 19 | 1 | 0 | 19 | 1 | 0 | 0 | 0 | 20 | 0 | 0 | 0 |
| dataquality (no area doc) | 16 | 12 | 4 | 0 | 12 | 3 | 1 | 0 | 0 | 16 | 0 | 0 | 0 |
| documents | 17 | 16 | 1 | 0 | 6 | 11 | 0 | 15 | 2 | 0 | 0 | 0 | 8 |
| epp | 39 | 36 | 2 | 1 | 26 | 12 | 1 | 21 | 6 | 11 | 0 | 1 | 18 |
| iam | 89 | 79 | 10 | 0 | 41 | 40 | 8 | 54 | 14 | 13 | 7 | 1 | 38 |
| organizations | 7 | 7 | 0 | 0 | 0 | 7 | 0 | 4 | 3 | 0 | 0 | 0 | 3 |
| payments | 11 | 11 | 0 | 0 | 7 | 4 | 0 | 10 | 1 | 0 | 0 | 0 | 7 |
| proflearning | 82 | 80 | 2 | 0 | 47 | 35 | 0 | 41 | 5 | 34 | 2 | 0 | 14 |
| profpractice | 48 | 46 | 1 | 1 | 30 | 17 | 1 | 27 | 5 | 15 | 0 | 1 | 21 |
| public portal (no area doc) | 3 | 3 | 0 | 0 | 3 | 0 | 0 | 0 | 0 | 3 | 0 | 0 | 0 |
| reporting | 16 | 13 | 3 | 0 | 8 | 7 | 1 | 9 | 1 | 1 | 5 | 0 | 5 |
| solution-level | 15 | 14 | 1 | 0 | 14 | 0 | 1 | 0 | 0 | 15 | 0 | 0 | 0 |
| staffing | 135 | 127 | 7 | 1 | 72 | 59 | 4 | 71 | 7 | 34 | 19 | 4 | 31 |
| **Total** | 676 | 620 | 47 | 9 | 352 | 298 | 26 | 337 | 70 | 218 | 34 | 17 | 181 |

Notes: the graph has 18 bounded contexts. Eleven have area docs and sequences (staffing, reporting, profpractices, proflearning, payments, organizations, iam, epp, documents, credentialing, communications). Seven have no sequences or API content: refdata, questionsets, publicportal, dataquality, dashboards, businessruleengine, audit. A `questionsets` capability (api, capability, permissions, sequences, technical design) is staged in `design/working-docs/question-set-updates` and is not merged into solution-areas or the graph; tab 04 feature 4.1 (rows r3 to r11) and the question-set parts of tabs 05, 09, 13, 14, 26 to 28 and 29 would pick up design coverage when it merges. The public search rows (tab 31, 39 rows) are served by credentialing, proflearning and EPP read operations rather than a public portal area.

### 3.4 RTM requirements with no counterpart in our design

Rows with no FDD story in the graph (RTM only). These are the strongest candidates for new graph requirement nodes:

| RTM id | Title (short) | Note |
|---|---|---|
| 05/5.5/r29, r30, r31, r34, r35 | Search users by type, conditional filters, type-specific columns, modify roles, track access changes | RTM splits one narrative story (our 5.5.1) into five; partial match only |
| 12/12.4/r18 to r22 | Non-CEDS data elements, reference manual user guides, process flow maps, data dictionary export, audit of manual updates | No counterpart anywhere (Feature 12.4 Data Dictionary) |
| 10/10.16/r26 | Access legacy staffing records and audit logs | Our 10.16 is a folded pointer with no story |
| 15/15.12/r46 | Request retirement of non-person and test IDs | Closest is our Split/Retire ID sequence (Mi-Key direct) |
| 17/17.2/r13, r14, r16, r17, r18 (and r12, r15 partial) | Application reference number, Conviction Review status, PPR questions by email link, fee payment by email link, permit issued and paid in full | Our 17.2 is a feature-level pointer; the behaviour is spread inside the permit submit stories |
| 18/18.6/r49 | Define standardised staffing communication templates | No story; template admin lives in communications |
| 21/21.7/r24 to r26 | District audit task worklist, report navigation, integrated employment and credential view | FDD 21 Feature 21.7 is not in our graph; a staffing sequence for ISD Auditor review exists (proposed link, low) |
| 29/29.1/r3 | View centralised certificates landing page | Partial to 29.1.1 |
| 09/9.6/r24 | "Merged with 9.1" | Pointer only |
| 11/11.4/r9, 13/13.2/r8 | Notes | Pointers only |

Rows where the graph holds the story but our design has no element satisfying it (218 rows). The features where every row is in this state or in the doc-only state:

- Dashboards: 3.x user-customizable dashboards (tab 03 r10 to r16 and the 7 catalog placeholders), 10.6, 11.7, 13.5, 14.5.
- Business rules and question sets: 4.1 (9 rows), 4.2 (11 rows). The graph `businessruleengine` context is a stub; question sets are staged, not merged.
- Data quality: 19.1 (9), 19.2 (6) and 19.3 (3) are mostly story-only; the only related design element is the Run Quality Review step inside the staffing Certify Collection sequence.
- Batch and worklist administration: 5.4 (4 rows, credential admin batch job history and run), 5.3 credentialing controls (5 rows), 32.14 (7), 32.15 (partly), 14.3, 13.3.
- Identity and administration: 1.7 user authorization reports, 1.8 emulate user (4), 5.6.
- Public portal and admin pages: 12.2 (3), 12.3 (5), public search tab 31 (16 story-only rows).
- Sponsor application: 26.1 (6 rows); the matching admin review 28.1 depends on a SponsorApplication entity we do not have.
- Rapbacks: 30.5 (3), 30.7 (3), 30.8 (3), plus the Rapback Admin role (section 4).
- Interfaces and ingestion: 8.8 Secure Site (3, deliberately out of scope in our graph), bulk and API upload (15.5, 18.5, 21.5, 22.4, 24.3 r9, 25.4, 26.5), 32.11 legacy migration, 32.3 auditing, 32.2 device and UI standards.
- Communications and reporting: 6.5 (3 rows, communications history into reporting). The graph also holds a story for two-way Outlook email (FDD 6 interface) that has no RTM row.
- Integrations without graph elements but with an integration doc section (34 rows): 8.4 DataHub, 8.5 CTEIS, 8.6 NexSys, 8.7 TSDL, 8.9 CEPI warehouse, and the 15.5 bulk identity rows.

Exact lists are in the CSV (filter `design_coverage` equal to `no-design-link` or `doc-only`).

### 3.5 Our design elements with no RTM row

By element type, counting an element as reached when an RTM-matched requirement lists it in `satisfiedBy` (sequences also reach the operations, events and entities they reference):

| Element type | Total | Linked to an RTM-matched requirement | Reached via a linked sequence only | Linked only to FDD stories with no RTM row | No requirement link |
|---|---|---|---|---|---|
| Sequence | 142 | 87 | 0 | 8 | 47 |
| ApiOperation | 303 | 186 | 15 | 14 | 88 |
| Aggregate | 51 | 27 | 0 | 2 | 22 |
| BusinessRule | 105 | 67 | 0 | 5 | 33 |
| Permission | 305 | 38 | 112 (via operations) | 2 | 154 |
| BatchJob | 12 | 5 | 0 | 0 | 7 |
| TechnicalProcess | 23 | 8 | 0 | 1 | 14 |
| ExternalSystem | 20 | 8 | 3 | 0 | 9 |

After adding the 28 proposed sequence links, 27 sequences still have no RTM row: Add Internal Remarks to Application Review, Admin Configures Evaluation Questions, Admin Manages Program Categories, Authorization Request Link Expiration, Automated Inactivity Deactivation, Business User Request Authorization Update, Business User Self-Remove Authorization, Business User Withdraws Pending Request, Configure Assessment Definition, Degraded Operation (organizations sync failure), Email Content Purge Job, Email Resend with Recipient Override, Exit Candidate from Program, External Party Request Removal, Embed Token parameter-resolution failure, Hierarchy Resolution, Integration Pattern for Domain API Calling Documents, Mass Email with Consolidation, Organization Detail Lookup, Reconciliation Discrepancy Investigation, Request Collection Exception, Search Approval Applications for Review, System Admin Reviews Removal Request, Template Variable Resolution (internal), Update EPP Configuration, Validate Assessment Results, View Approval Application Detail for EPP Review.

By capability, we design more than the RTM asks for in these places (with the caveat that many aggregates and operations are only linked through their sequences):

- IAM account lifecycle: inactivity deactivation, request expiry, update, withdraw, self-remove and external-party removal flows. RTM 1.6 covers only the initial request and approval.
- Documents: legal hold, soft and hard delete, SAS-token download, quarantine and scan patterns, storage tiering. RTM 32.16 states the outcomes in general terms; most of our mechanism has no row.
- Payments: the dual-approval refund threshold, circuit breaker and concurrency control, discrepancy investigation. RTM 7.x is silent on approval thresholds.
- Communications: delivery status webhook, content purge and archival, resend with override, mass email consolidation.
- Organizations: CEPI nightly sync, degraded operation, hierarchy resolution. The 8.1 rows reference EEM load only in general terms.
- Cross-cutting: authorization evaluation and org data caching (IAM), embed token generation (reporting), electronic signature capture (credentialing technical process; RTM 17.1 r6 and 26.1 r7 ask for signature behaviour).
- External systems in the graph with no RTM link: NexSys, MiDataHub, CTEIS, MSDS TSDL and CEPI appear in tab 08 rows but are not linked in the graph; Microsoft 365 Graph (two-way Outlook email) has no RTM row at all; Azure Synapse, Defender for Storage and Blob Storage are technology elements.
- The audit bounded context (stub) and refdata have no RTM rows of their own, though 32.3 and 32.14 assume audit exists.

FDD stories in the graph with no RTM row: 83 of 736 nodes. By FDD: 2 (6), 3 (4), 6 (2), 7 (1), 9 (3), 10 (4), 11 (1), 14 (4), 15 (5), 17 (19), 18 (1), 21 (2), 24 (1), 27 (3), 28 (8), 29 (5), 30 (4), 32 (10). Of the 83, 39 are unnumbered "Business Specification" narrative items (for example the six capabilities of FDD 2 beyond 2.1.1 to 2.1.6, the Rapback matching and search items in FDD 30, the sponsor worklist and mass email items in FDD 28). 19 are the FDD 17 permit type sub-flows (Extended Daily, Full Year Basic, School Administrator, School Social Worker and so on), where the RTM carries only 16 rows. 9 are FDD 32 non-functional statements (32.1, 32.4 to 32.10, 32.13) which the RTM lists as feature-only rows with no story. The rest are withdrawn features we model as rejected (15.7, 15.14, 15.15, 10.14, 28.2) and a handful of story splits. Whether the missing RTM rows reflect the client's approval process ("made available once approved by the POs") or omission is a question for the user (section 8).

## 4. (c) Mismatches: where we solutioned something differently

Each entry gives the RTM id, our artifact and what differs. Sources are our docs and graph notes; where an existing graph annotation records the same finding it is named. One technical-design statement (impersonation) is cited because it is the only record of that decision. Entries are ordered roughly by impact.

| # | RTM id(s) | Our artifact | What differs |
|---|---|---|---|
| 1 | 01/1.3/r10, r11, r12; 01/1.4/r14, r15; 01/1.5/r17 (and r18) | iam-domain.md "No User Group Layer" (decision 2026-08-14); iam-permissions.md; Req_01_3_x, Req_01_4_x, Req_01_5_1 | RTM defines User Groups as a first-class list, create and edit, a MiLogin-type consistency check on groups, and roles shown by group. Ours has Role plus Scope only; the MiLogin type constraint is a grant-time check on `RoleDefinition.ApplicableMiLoginTypes`. Annotation (contradiction, open, high) records the same. |
| 2 | 01/1.5/r18, r19, r20 | Op_ListRoles, Op_GetRole (iam-api.yml) | RTM wants admins to create roles, add functions to a role and edit mappings. The IAM API exposes read access to roles only; no create, update or delete role operation. Annotation: coverage-gap, open, high. |
| 3 | 01/1.8/r26, r27, r28, r29 | Perm_UserImpersonate (iam-permissions.md) | RTM wants a read-only "view as" for admins with no saving. Ours catalogues the permission iam.user.impersonate but has no operation or sequence; the IAM technical design records impersonation as deliberately excluded. Annotation: contradiction, open, high. |
| 4 | 32/32.15/r25 to r33; 13/13.3/r9, r10; 14/14.3/r9, r10; 05/5.5/r29 to r35; 28/28.5 | solution-architecture.md "Worklist / Pending-Items Pattern" | RTM has admin screens to create and manage worklists, customise columns and fields, and allocate view, process, reassign and configure privileges per role or per user. Ours treats a worklist as a filtered list endpoint per domain with no first-class worklist, no cross-domain worklist permission and no admin-defined groupings; the doc says to revisit if admin-defined groupings appear, which these rows do. Annotations (coverage-gap, open, high) exist on 13.3 and the EPP domain doc. |
| 5 | 03/3/r10 to r16; 29/29.1/r10; 11/11.7/r15 | reporting-capability.md "Not Every Dashboard Is a Power BI Report" | RTM has per-user customisable dashboards (add, remove and arrange sections and widgets, persisted layout, refresh on load). Ours treats dashboards as UI concerns per domain; no dashboard-preference entity or endpoint, no preset-dashboard admin, and the `dashboards` context is a stub. Annotation: coverage-gap, open, high. |
| 6 | 04/4.2/r12 to r22; 10/10.3/r11; 10/10.13/r22, r23; 19/19.2/r12 to r17 | solution-tech-standards.md CQRS section (rules enforced in domain aggregates); BusinessRuleEngineContext stub | RTM expects an admin-facing rule management capability (turn rules on and off, effective-date ranges, error versus warning, justification options, external-source validation, versioning, test against live data, extract of rules and file schemas). Ours enforces rules in code inside each domain; there is no rule-management capability. Annotation: coverage-gap, open. |
| 7 | 04/4.1/r3 to r11 | Staged `questionsets` capability (not merged) | RTM wants admin authoring and versioning of question sets. Not in solution-areas or the graph; a staged capability exists and addresses most of it. |
| 8 | 19/19.1/r3 to r11; 19/19.2/r12 to r17; 19/19.3/r18 to r20 | Seq_CertifyCollection (Run Quality Review step) | RTM describes a cross-domain, admin-configurable data quality service (dashboard, on-demand run, acknowledge with justification, action items, scheduled checks). Ours has quality review only inside the staffing collection certification flow, and the `dataquality` context is a stub. Annotation: coverage-gap, open, high. |
| 9 | 26/26.1/r3 to r8; 28/28.1/r3 to r7; 28 sponsor worklist | ProfessionalLearningSponsorAggregate (design note) | RTM has a sponsor application lifecycle (initial application, coordinator designation, lock on submit, pending status, Lead Administrator e-signature, review by PL Admin: approve, reject, edit, delete). Ours says sponsor creation is EEM-only, MiEdWorkforce only manages existing sponsors and there is no PendingApproval state. Our permission catalogue still defines proflearning.sponsor.approve, so our own docs disagree. Annotation: contradiction, open, high. |
| 10 | 30/30.1 to 30.8, all 19 rows | profpractice-permissions.md | Every RTM Rapback story is for a "Rapback Admin". Our permissions define PPR Reviewer, PPR Admin and Credentialing Admin only; no Rapback Admin role. Also 30.7 requires profpractice to query staffing and credentialing, the reverse of the current cross-domain direction. Annotation: coverage-gap, open, high. |
| 11 | 06/6.3/r11 | communications-capability.md (templates are Markdown with variable placeholders; HTML sanitised) | RTM asks for a rich text editor with images and ADA-compliant fonts. Ours authors templates in Markdown with a limited allowlist and has no WYSIWYG editor, image insertion or font selection. Annotation: discrepancy, open, high. |
| 12 | 09/9.1/r9 (and 17/17.2, 29/29.2/r19, r20) | profpractice-domain.md "Annual PPR Compliance" | RTM says a scheduled reset sets a Needs Responses flag to Yes each June 30. Ours computes compliance from the latest PPR response date and stores no marker. The RTM and several permit stories also use a Needs Responses status; ours has no such field. Annotation: discrepancy, open, medium. |
| 13 | 09/9.2/r11 | Req_09_2_2 (rejected) | RTM still carries per-status visibility configuration for PPR Reviewers. The vendor team marked it outdated and we model it as rejected; the RTM row is stale. |
| 14 | 11/11.8/r17; 29/29.2/r18; 29/29.3/r30 | Rule_ManualReviewTriggering (credentialing-domain.md) | Review comments on the FDD say a missing or failing MTTC score should block submission. Ours flags the application for manual review after submission. RTM 11.8 r17 says "prevent issuance or flag for review", so it spans both. Annotation: contradiction, open, high. |
| 15 | 17/17.1/r9, r10, r11 | Seq_RenewCredential (actor Educator, Op_SubmitRenewal self-service) | RTM says district, ISD or staffing agency users renew Full Year permits and enforce renewal limits. Ours models renewal as educator self-service. The RTM also lists a "Full Year Expert" permit type that our permit model does not have. Annotation: discrepancy, open, medium. |
| 16 | 22/22.1/r8 | staff:Rule_EvaluationOutcomeAppealWindow | RTM states the appeal window as 5 years for the citizen. Ours lets citizens appeal any outcome with no time limit and applies 5 years to district appeals only (this follows the FDD narrative, not the Addendum row). Annotation: discrepancy, open, low. |
| 17 | 07/7.1/r4, r7 | Seq_CepasServiceUnavailable | RTM r7 says the user sees an error pop-up on the CEPAS website when CEPAS is down; that is outside our control. Ours shows an unavailable error before the redirect. RTM r4 and the FDD author disagree on whether MiEdWorkforce displays any payment message. Annotation: discrepancy, open, medium. |
| 18 | 08/8.1/r3 to r9 | Seq_CEPINightlySync, BatchJob_CepiNightlySync (organizations) | RTM r7 says "via ETL", r5 scheduled full loads, r9 incremental updates. Ours is a nightly full-snapshot upsert from the CEPI CEDS JSON-LD layer; delta support is pending CEPI. The two EEM interface documents describe different mechanisms (bearer-token REST and a Synapse pipeline), so RTM r3 to r9 do not map to one design. Annotations: discrepancy, open (two). |
| 19 | 08/8.4/r22 to r26 | solution-integrations.md section 11 | RTM covers SendToHub and ReceiveFromHub only. Ours documents two mechanisms and treats the Ed-Fi Identities API (MiDH initiates, 20 use cases) as the main one; MiEdWorkforce is passive. The RTM has no rows for the 20 use cases. |
| 20 | 29/29.4/r35 | Req_29_4_2 | RTM keeps a story for rule-based downloadable cover letters. Review comments from the product owner and vendor agree it is not needed (the cover letter goes in the email). Annotation: discrepancy, open, high. |
| 21 | 31/31.5/r30; 31/31.2/r11 | Rule_OnlyActiveProgramsInCatalog; Rule_PublicEducatorCredentialSearch | RTM r30 applies Active-only filtering to Sponsors as well as Programs; the rule covers programs only. RTM 31.2 groups credential history into three sets (Active, Active Temporary, Inactive) per the FDD acceptance criteria; our rule has two (Active, Inactive). Annotations: discrepancy, open. |
| 22 | 13/13.3/r9 | Perm_ApprovalsView, Perm_ApplicationsView (epp-domain.md note) | RTM presents the EPP worklist admin page as an EPP admin feature; the wireframe breadcrumb shows it as a credentialing admin screen, and ours has no worklist admin at all (see 4). Annotation: discrepancy, open, high. |
| 23 | 27/27.2/r8; 14/14.x | proflearning-domain.md (ProgramAttendance, SCECH correction) | RTM has citizens submit a required evaluation to trigger automatic SCECH awarding. Review comments say surveys are off at launch and educators need an attestation path. Our design settles neither. Annotation: discrepancy, open, high. |
| 24 | 20/20.0/r3 to r6 | communications internal and external remarks | RTM wants one generic internal and external comment and flag framework reused everywhere (with mandatory comments on negative flags). Ours has comments inside communications but not a generic flag framework spanning domains. Annotation: discrepancy, open, medium. |
| 25 | 05/5.4/r25 to r28; 32/32.14/r18 to r24 | BatchJob nodes per domain (12) | RTM wants a cross-cutting admin surface for batch job history, schedule create, modify, cancel and on-demand runs. Ours describes what each batch job does; there is no schedule administration surface. Annotation: coverage-gap, open, high. |
| 26 | 15/15.5/r18 to r21; 18/18.5/r37 to r45; 21/21.5/r22; 22/22.4; 24/24.3/r9; 25/25.4/r13; 26/26.5/r17 | Seq_StagedFileUploadForSynapseProcessing (documents); Perm_EmployeeRosterBulkUpload | RTM expects bulk file and API entry paths with staging, partial success and a review step. Ours has a file staging pattern in Documents and a Synapse pipeline, but staffing, iam and other domains have no bulk upload operations, and the roster bulk-upload permission has no backing operation. Annotations: coverage-gap, open, high (several). |
| 27 | 32/32.3/r6 versus 05/5.1/r6, 05/5.3/r24, 28/28.4/r12, r13, 30/30.6/r14 and others | audit bounded context (stub) | RTM marks 32.3 Auditing "no longer needed" while many stories need audit trails. Ours has an audit context stub with no aggregates. Annotation: coverage-gap, open, high. Also an RTM-internal contradiction (section 5). |
| 28 | 02/2.1/r4; 09/9.5/r20 to r22 | reporting-api.yml ReportCreate schema (Power BI workspaceId and reportId) | Our model holds two Power BI identifiers; the wireframe behind 2.1 shows a single free-text link field. The RTM row says only "register a new embedded report". Low confidence that this is a visible mismatch. |
| 29 | 08/8.8/r41 to r43 | solution-integrations.md (no section) | RTM keeps three Secure Site rows (future-ready structure, remove obsolete dependency, document). Our graph records Secure Site as not an active build; there is no integration section. Annotation: out-of-scope, resolved. |

Where the RTM and our design agree, so no mismatch: payments (tab 07; bulk payment, bulk refund and daily posting reconciliation are all designed), the permit-application flow in tab 29 (credential landing, eligibility, MTTC check, fee collection through CEPAS), Mi-Key identity flows in tab 15 (26 of 43 rows trace to sequences), EPP candidate tracking in tab 25 (all 11 rows reach sequences), SendGrid and Power BI as the chosen vendors, the `<<variable>>` placeholder style, and the four CTEIS method names (which match the names in our integration doc).

## 5. (d) Over-fitted, badly worded, ambiguous or untestable RTM rows

Several vendor-specific rows agree with our chosen technology (SendGrid, Power BI, Synapse, Mi-Key); they are still stated as implementation rather than need.

### 5.1 Dictate technology, vendor, interface name, table or field

| RTM id | Reason | Suggested wording |
|---|---|---|
| 06/6.1/r5 | Names SendGrid and its API | The system can send email through the state-approved delivery service and record delivery status |
| 02/2.1/r3, r4, r7, r8; 09/9.5/r20, r22 | Names Power BI; r8 says export formats "defined within Power BI" | Users can open administrator-registered embedded reports and export them to PDF, Excel and CSV |
| 08/8.1/r7 | Says process "via ETL" | The system loads and transforms EEM entity data so that hierarchy and operational details are accurate |
| 08/8.2/r10 | Mandates an API key | Calls to the NASDTEC clearinghouse are authenticated and only authorised requests are processed |
| 08/8.4/r23, r26; 08/8.5/r28 to r30, r32 | Names SendToHub, JSON, and the interface methods (GetPersonnelBy..., GetCompleteAssignmentCodeList) | The system lets CTEIS retrieve personnel by district, by identifier and by core demographic fields, and retrieve the valid assignment code list |
| 05/5.2/r16, r18 | Table with Add, Edit, Delete rows; comma-separated test codes | The admin can maintain endorsement codes and map one or more required tests to an endorsement |
| 05/5.1/r4, r6; 05/5.2/r14; 05/5.3/r21, r24; 05/5.4/r28; 05/5.5/r35 | Name audit fields (Modified By, Modified On) and results columns on each screen | One cross-cutting rule: every configuration change records who changed what and when |
| 05/5.2/r13 | Filter option list stated as UI detail (Teacher and School Administrator only) | Endorsement search is limited to certificate types that carry endorsements |
| 06/6.2/r8 | "Order of database fields" | The admin can control how recipient names and details are formatted in a merged email |
| 07/7.2/r10 | "Update the respective MiEdWorkforce tables" | Payment status in MiEdWorkforce is corrected when reconciliation finds a missed payment |
| 15/15.5/r18, r20, r21; 15/15.1/r5; 15/15.8/r29 | Names the common file staging area, a sandbox view, the Mi-Key Asynchronous Assignment Service, probabilistic matching | State the user outcomes: bulk submit, review validation results before committing, unresolved matches routed to an administrator |
| 18/18.1/r5, r7, r8 | Describes screens: empty Assignments page, three-step workflow, tab refresh with health indicators | The district user can assign an eligible employee to a position and sees updated FTE totals and quality status |
| 21/21.2/r13 | "Update button" and read-only toggling | Finalised records are protected; authorised users can reopen them for change |
| 31/31.1/r3; 31/31.4/r19; 31/31.5/r29 | Link placement "below the banner" | The public landing page offers a way to open each public search without logging in |
| 31/31.2/r15, r16, r17; 31/31.4/r20, r25; 31/31.5/r37, r38 | Exclamation icon, "Back to ..." button, new-window link, hover icons, dropdown versus dynamic filters, sortable and paginated grid | Express as findability and clarity needs; leave controls to UI design |
| 31/31.1/r6 | "Filter out educators whose Code ID has a NULL value" is a data detail | Educators without an issued credential identifier do not appear in public results |
| 30/30.6/r16 | The "No Referral Required" item must not be clickable | Action history items link to their artefact where one exists |
| 15/15.10/r41 | Title contains the start of the description text | Fix title |
| 08/8.1 to 8.9 (r4, r13, r24, r35, r48 and others) | Same audit-and-log story repeated per interface | One non-functional requirement: every integration exchange is logged for audit |
| 08/8.2/r16, 8.3/r21, 8.5/r31, 8.6/r36, 8.7/r39, 8.9/r47, 8.4/r25 | "Secure and environment-aware" per interface | One cross-cutting deployment standard |

### 5.2 Ambiguous, untestable or unmeasurable

| RTM id | Reason |
|---|---|
| 06/6.1/r6 | "Large volume" with no volume or time target; add numbers (messages per hour, recipients per send) |
| 32/32.2/r4 | "Adhere to State of Michigan Digital Standards" names no document or version, actor is "Developer" |
| 32/32.2/r5 | "Accessible and operational on my mobile device" with no device, browser or accessibility target |
| 32/32.11/r14, r15 | "All personal and certification information" aligned to CEDS is unbounded; "or appropriate platform (e.g., cold storage)" is a design suggestion, not a need |
| 08/8.9/r45, r46 | "Avoid duplicating migration work", "where appropriate", "if required": hedged and not testable |
| 08/8.8/r41 to r43 | A "future-ready structure", "avoid obsolete dependencies" and "document the potential" are not testable system behaviour |
| 08/8.7/r38, r40 | A change task for TSDL and "frequent queries" with no frequency |
| 07/7.1/r5, r6 | "Information necessary" and "all necessary confirmation data" do not list the data; reference the CEPAS interface definition |
| 07/7.1/r7 | Describes a CEPAS-hosted error pop-up that MiEdWorkforce cannot deliver |
| 17/17.1/r11 | "A max number of renewals" with no number |
| 22/22.1/r8 | Embeds a parameter value ("currently 5 years") in the requirement; make the window configurable and state it in a rule |
| 22/22.3/r14 | Position list is garbled ("Teacher positions on a Temporary Credential positions") |
| 29/29.2/r15, 29/29.3/r29 | "Field-level and application-level validations for all certificate types" is too broad to test; point to the rule set |
| 29/29.2/r19, r20 | Specify a flag name and its logic rather than the screening outcome |
| 19/19.1/r5, 19/19.2/r17 | "(i.e., Excel)" and "(i.e., Excel, SQL)": a SQL extract is a design choice; "i.e." should be "e.g." |
| 19/19.2/r15 | Analysis across "other data systems" with no systems named |
| 04/4.2/r16, r21 | No benefit clause; "validate against current system data" undefined |
| 30/30.5/r11 | "Defined file formats" are not defined |
| 12/12.4/r19, r20 | Documentation tasks written as system requirements |
| 03/3/r10 | "Immediately upon login" without a time target |
| Many rows in tabs 16, 19, 24 and 30 | Missing "so that" clause, so the need is not stated |
| 27/27.3/r11 | Contains a stray editing tag "[BD1]" |
| 09/9.1/r5 | Typo in title ("diclosure") |
| 15/15.3/r9 | Blank title |

### 5.3 Contradictory or duplicated inside the RTM

| RTM id | Reason |
|---|---|
| 05/5.1/r9 | Title says "Version Permit Records by Effective Date" but the description is the rename-Type story, a duplicate of r10 |
| 03/3/r11, r12 | Title of r11 says manage and arrange sections but text covers add, remove, modify; r12 title says sections but text says widgets; r13 and r14 repeat the same pair for widgets |
| 32/32.3/r6 versus audit stories in 05, 28, 30, 32.14 | Auditing marked not needed while many rows require audit trails |
| 10/10.3/r11 | Feature titled "Manage System Alerts/Notifications" but text is business rule management, a copy of 10.13 |
| 17/17.2 (7 rows) | Feature 17.2 is titled "Temporary Credentials User Feedback" but reads like end-to-end permit status messaging |
| 31/31.2/r10 and r12 | One row hyperlinks the whole result, the other the Credential Number |
| 09/9.6/r24 | "Merged with 9.1" |
| 13/13.7, 19.5, 26.4, 31.3, 32.3 | "No longer needed" rows still listed as requirements |
| Tab 04 sheet heading | Business Rules Management tab has the heading "Dashboards" (copied from tab 03) |
| Tab 08 rows r49 to r51 | MTTC 11.8 rows sit in the interfaces tab |
| Feature IDs 10.10, 11.10, 15.10, 32.10 | Stored as numbers, so they read as 10.1, 11.1, 15.1, 32.1 and collide with real features |
| 08/8.1/r6 | "Synchronise data between EEM and MiEdWorkforce" overlaps r5 (full loads), r9 (incremental) and r4 |

## 6. (e) Near-duplicate or overlapping RTM rows

Groups of rows that state one thing in different words or on different screens. Similarity is TF-IDF cosine on the description; groups marked "manual" were found by reading.

| Group | Rows | Note |
|---|---|---|
| "No results found" message | 31/31.1/r8, 31.4/r27, 31.4/r28, 31.5/r33, 31.5/r34, 28/28.5/r22 | One requirement for all public and admin searches (similarity 1.0 for r8 and r33) |
| Ignore special characters in search | 31/31.1/r4, 31.5/r31 | Identical text |
| Full and partial match search | 31/31.1/r5, 31.4/r21, 31.5/r32 | Same behaviour three times |
| Landing page links to a public search | 31/31.1/r3, 31.4/r19, 31.5/r29 | Three links, one need |
| Dashboard sections and widgets | 03/3/r11, r12, r13, r14 | Two pairs that read as duplicates |
| Rename Type, permit | 05/5.1/r9, r10 | Same text |
| Audit metadata on save | 05/5.1/r6, 05/5.3/r24, 05/5.5/r35, 05/5.1/r4, 05/5.3/r21, 05/5.4/r28 | Same audit rule repeated per screen |
| Toggle report active status | 02/2.1/r5, 09/9.5/r21 | Same |
| Export reports to PDF, Excel, CSV | 02/2.1/r8, 09/9.5/r23 | Same |
| Embedded report register and manage | 02/2.1/r3, r4; 09/9.5/r20, r22 | r3 and r4 overlap; 9.5 repeats 2.1 for PPR |
| Auditor worklists | 21/21.7/r24 to r26 and 22/22.6/r18 to r20 | ISD versus SOM versions of the same three stories |
| Worklist column customisation | 13/13.3/r10, 14/14.3/r10 | Same text, different epic |
| Worklist administration | 13/13.3/r9, 14/14.3/r9, 32/32.15/r25 to r33, 05/5.5/r29 to r35 | One worklist-admin capability stated in four places (manual) |
| Batch job history and run | 05/5.4/r25 to r28 and 32/32.14/r18 to r24 | Same capability stated twice (manual) |
| Mapping change notifications | 05/5.2/r17 and 12/12.1/r7 | Notify admin when a new endorsement needs assignment mappings (manual) |
| Roster requirements | 10/10.9/r18, 10/10.10/r19 | Position and employee roster; one story with two collections |
| Employment record creation | 21/21.1/r4, r5 | Validated ID versus no ID; same shape |
| Renewals | 17/17.1/r9, r10, r11 | Renew, do not renew, maximum renewals: three views of one permit rule set |
| Guidance on dashboard | 11/11.2/r6, r7 | Same need for two roles |
| Validate demographic data before Mi-Key | 15/15.1/r4, 15.8/r29, 15.5/r19, 24/24.1/r3 | One validation requirement in four contexts |
| Process results from Mi-Key | 15/15.1/r5, 15.5/r21 | Same |
| Notify near-match hold | 15/15.3/r9, r10; 15/15.8/r31; 15/15.10/r42 | Same notification in three features |
| Internal and external comments | 06/6.4/r14, r15; 20/20.0/r3, r4 | 6.4 and 20.0 define the same comment and visibility behaviour (manual) |
| Rule versioning and audit | 04/4.2/r19, 19/19.2/r13, 10/10.13/r22, 10/10.3/r11 | Same rule governance stated four times |
| Business rule enforcement across channels | 04/4.2/r17, 18/18.5/r44, 21/21.5/r22 | Same (manual) |
| Credential auto-approval | 11/11.5/r10, 16/16.1/r4 | Same need in two tabs |
| Persist landing preferences | 29/29.1/r10, 03/3/r15 | Same (manual) |
| Certificate print or download | 29/29.1/r8, 29/29.4/r34 | Same need, two screens |
| Additional endorsement lists | 29/29.3/r25, r26 | Teacher and administrator versions |
| Bulk file and API entry | 18/18.5/r37 to r45, 21/21.5/r22, 22/22.4, 24/24.4/r10, 15/15.5/r18 to r21, 25/25.4/r13 | One ingestion pattern repeated per roster (manual) |
| Audit logs of actions | 28/28.4/r13, 28/28.5/r23 | Same audit log shown from two places |
| NASDTEC query and store | 08/8.2/r11, r12 | One exchange stated twice |

## 7. How the graph should carry RTM ids (proposal, not implemented)

Findings that shape the proposal:
- The RTM has no row id yet (Req ID blank), only Feature ID and row order. Joining on feature plus text works now, but the key will change once DevOps ids arrive.
- The relationship between RTM rows and graph requirement nodes is almost one to one (649 graph nodes are matched by exactly one row, 3 by two, 1 by six pointer rows).
- A sequence relates to many RTM rows (mean 2.0, maximum 9) and an RTM row can relate to several sequences. Storing RTM ids on a sequence would duplicate and go stale; the sequence link should be derived.
- About 24 RTM rows have no graph story and 83 graph stories have no RTM row, so both directions need a representation.

Proposal:
1. Add one multi-valued string property, `sd:rtmId`, on `sd:Requirement`, `sd:UserStory` and `sd:NonFunctionalRequirement`. Value: the DevOps story id when it is supplied (for example `ADO-12345`); until then the composite `<tab>/<feature>/<title slug>` (for example `07/7.1/redirect-to-secure-cepas-payment-gateway`). Keep `sd:requirementId` as the FDD story number it is today. A second optional property `sd:rtmFeatureId` on the same nodes would hold the RTM feature id when it differs from the FDD one (for example the tab 08 rows for 11.8).
2. Do not add `sd:rtmId` to Sequence, ApiOperation, Aggregate, BusinessRule or Permission nodes. Derive it at export time: a sequence's RTM ids are the `sd:rtmId` values of the requirements that list it in `sd:satisfiedBy`. Where a sequence is linked to a requirement only by this analysis (the 28 proposals), add the `sd:satisfiedBy` edge on the requirement after review; it then flows to the sequence view with no new property.
3. For RTM rows with no graph story (about 24), create `sd:Requirement` nodes with `sd:rtmId`, statement from the RTM text, `sd:definedIn` pointing at a new `sd:SourceDocument` for the RTM workbook (with the download date), and a note that the row is RTM-only. That keeps the rule "every client requirement is a node" and lets the existing coverage queries flag the 17.2, 21.7, 12.4 and 5.5 rows as gaps.
4. For graph stories with no RTM row, leave `sd:rtmId` empty; a SPARQL check "requirement without rtmId" lists the 83.
5. Optional later step: model the RTM status columns (UI design status, DEV, UAT, PROD, test case id) as properties on the same nodes only if the team wants the graph to answer delivery questions; they are blank in the RTM today, so this is not needed now.
6. Add a SHACL shape for `sd:rtmId` (string, non-empty) and a validation query for duplicates across nodes. Expect one allowed exception: a single pointer story that several RTM pointer rows share.

## 8. (f) Questions for the user

1. Can we get the Azure DevOps story ids (the blank Req ID column), either from the BA or from a DevOps export? Without them the proposed `sd:rtmId` stays a composite key and will need re-keying.
2. Do you want the roughly 24 RTM-only rows added to the graph as requirement nodes, or kept in a side list for the BA conversation?
3. The 83 graph stories with no RTM row (39 unnumbered Business Specification items, 19 FDD 17 permit sub-flows, 9 non-functional statements, withdrawn features) may be outside the client's approved set. Do we treat them as out of scope for traceability, or ask the BA whether they are intended?
4. Pointer rows ("N/A" title, 35 rows) and "no longer needed" rows (5): should they carry an RTM id on the graph, or be excluded from the mapping?
5. Section 4 lists 29 places where our solutioning differs from RTM text, most of them already open as annotations. Should these be logged in the tracking files, taken to the client as design clarifications, or both? None are proposed as design changes.
6. Tab 04 is titled "Dashboards", Feature IDs lose a trailing zero, row 5.1 r9 has a title that does not match its text, and 27.3 r11 has a stray editing tag. Do you want a short list of RTM corrections sent to the BA?
7. 28 sequences were proposed against RTM rows by keyword and judgement only. Do you want those edges added to `satisfiedBy` after your review, or left as report-only?
8. The `questionsets` capability is staged and not in the graph. Should the next RTM pass count it, so 4.1 and related rows pick up design coverage?
9. Several of the over-fitted RTM rows agree with our chosen technology (SendGrid, Power BI, Synapse). Should we treat them as stated constraints, or as wording to be neutralised in the RTM?
10. Dashboards, business rules, data quality and worklist administration are the largest design gaps the RTM exposes. Is the plan to handle them in new areas, or are they intentionally deferred?
