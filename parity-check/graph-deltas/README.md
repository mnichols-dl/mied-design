# Graph Deltas: How the Graph Differs from the Source Docs

Extracted 2026-10-06 from the original graph (the first commit of `design`, before any re-ingest) compared with `design/working-docs/`. Each JSON file here holds, for one area and doc kind, every field where the graph had words the source doc does not: the full graph text, the full doc text, and the word runs that differ. After this extraction the doc's text was taken into the graph everywhere (doc wins), so these files are the record of what the graph had added, and they can be re-applied.

Only fields where the graph has words the doc lacks are recorded. Where the doc simply has more, or the same words with different formatting, the graph loses nothing and nothing is recorded.

## How many, by area

Counts of fields, with in parentheses how many are substantive (the graph added more than 25 words).

| Area | Domain or capability doc | Permissions doc | Sequences doc | Total |
|---|---|---|---|---|
| communications | 18 (4) | 41 (0) | 16 (0) | 75 (4) |
| credentialing | 10 (3) | 3 (1) | 5 (1) | 18 (5) |
| documents | 12 (0) | 5 (3) | 8 (0) | 25 (3) |
| epp | 12 (5) | 5 (1) | 0 (0) | 17 (6) |
| iam | 13 (0) | 12 (2) | 2 (0) | 27 (2) |
| organizations | 1 (1) | 1 (0) | 3 (0) | 5 (1) |
| payments | 35 (4) | 12 (7) | 2 (0) | 49 (11) |
| proflearning | 18 (1) | 27 (11) | 0 (0) | 45 (12) |
| profpractices | 35 (1) | 22 (11) | 6 (1) | 63 (13) |
| reporting | 2 (0) | 3 (2) | 4 (0) | 9 (2) |
| staffing | 9 (0) | 24 (6) | 1 (0) | 34 (6) |
| **Total** | | | | **367 (65)** |

## What the graph changed, in short

189 of the 367 fields (51%) differ by 5 words or fewer: swapped articles, tense (`modify` became `modifies`), a changed preposition, or a couple of words of paraphrase. These are rewordings, not new information, and are safe to ignore.

The substantive ones (65) fall into a few recognizable kinds:

1. **Review findings written into client-facing notes.** The largest group by far, concentrated in the permissions notes (153 of the 367 fields are permissions notes). The graph's note for a permission often carries a sentence or paragraph about a *gap in the design*, for example that no API endpoint or sequence models the permission yet, or that it contradicts another doc. These come from the FDD review pass, not from the original docs. Examples are `reporting.catalog.manage`, `proflearning.sponsor.approve` and `proflearning.evaluation.submit` below. They are valuable findings, but they are review notes, and they probably belong on an annotation or in the gaps list rather than in a client-facing permissions table.
2. **Business-rule cross-references added to aggregate invariants.** The graph's invariants for an aggregate often append `(Business Rule: ...)` pointers and open-question numbers (EPP's `EducatorPreparationProvider` and `CredentialApplicationReview`, for example). The doc's bullet list states the invariants without those links; the graph made them explicit.
3. **Expanded detail in rule statements and examples.** 50 rule examples and 34 rule statements differ. In several the graph folded a doc's sub-lists or extra paragraphs into one expanded statement (for example communications' Template Variable Mandatory Resolution, which gained a numbered failure-handling procedure), and a few rules have an example the doc does not.
4. **Field-level additions.** Event triggers (18), glossary definitions (11) and aggregate descriptions (6) where the graph is slightly fuller than the doc.
5. **Sequences.** 47 fields, mostly Key Decisions (27) and What/When/Who (16), plus a few Error Scenarios.

Language used in the graph's added text, counted over all fields where the graph text has it and the doc does not: a review finding, gap or open item (61 fields), an API or endpoint detail (67), a cross-reference to another doc or rule (35), a dated or decision note (7). A field can be in several.

## The most substantive additions

Word counts are of the words the graph has that the doc lacks.

| Area | Doc | Item | Field | Words | The graph added (word runs, lower-cased) |
|---|---|---|---|---|---|
| organizations | domain | Organization | invariants | 119 | enforced by the sync pipeline sets lead admin email from cepi data only no write endpoint exists for this field ... with ... new internal id same organization code enforced by sync pipeline  |
| reporting | permissions | reporting.catalog.manage | notes | 105 | this remains the only permission gating reporting api ttl s admin catalog operations op admincreatereport et al there is not yet a per domain equivalent of communications create edit activat |
| proflearning | permissions | proflearning.program.edit | notes | 102 | entity resolves to program scoped partial gap ... the ... initiated path this is effectively covered by post program applications with applicationtype modify plrn op submitprogramapplication |
| communications | domain | Template Variable Mandatory Resolution | statement | 101 | failed resolution handling 1 send blocked error logged with full context template id version id event id missing variable names 2 event moved to dead letter queue with retry metadata 3 dashb |
| epp | domain | EducatorPreparationProvider | invariants | 101 | business rule epp must reference valid eem organization core organization attributes such as name address cannot be modified in miedworkforce they sync from eem ... must add certificate cate |
| epp | domain | CandidateEnrollment | invariants | 94 | business rule candidate program assignment required for placement status ... business rule duplicate program prevention ... a ... and the valid status set is itself route restricted ... fdd  |
| proflearning | permissions | proflearning.sponsor.approve | notes | 91 | genuine gap already flagged in proflearning domain ttl s own aggregate design note and plrn question sponsorapprovalworkflowcontradiction open question 9 the professionallearningsponsor aggr |
| credentialing | sequences | Issue Temporary Permit - Exceptional Cases (Admi | keyDecisions | 87 | status unconfirmed capability models an expedited bypass payment auto approved ... issuance path with no fdd evidence found fdd 17 s eight permit family documents and fdd 11 s bulk permit re |
| epp | domain | CredentialApplicationReview | invariants | 81 | business rule epp program approval scope ... business rule conviction flag manual review per fdd 25 1 25 3 confirmed as a red font ui indicator on the search results table and review guidanc |
| profpractices | domain | Annual PPR Compliance Requirement | statement | 78 | is determined ... if lastpprresponsedate is null new educator compliance is required after first credential application if before june 30 of the current year compliance is required if on aft |
| iam | permissions | iam.user.impersonate | notes | 75 | documented discrepancy this permission is cataloged here iam permissions md but iam technical design md s omission of user impersonation section states impersonation was designed evaluated a |
| proflearning | permissions | proflearning.evaluation.submit | notes | 70 | significant gap neither proflearning api yml no evaluation submission endpoint nor proflearning sequences md no attendee submits evaluation sequence despite the evaluationtemplate aggregate  |
| proflearning | permissions | proflearning.admin.questions | notes | 70 | likely stale gap proflearning domain md s own scope section explicitly says the question set framework is a cross cutting capability this domain does not own and no api operation in proflear |
| epp | permissions | epp.admin.system | notes | 66 | a blanket super permission not tied to any single operation epp api yml s per operation x permissions required lists never name it directly e g createeppprovider is gated by epp provider cre |
| reporting | permissions | reporting.embed-token.generate | notes | 63 | permissionkey e g reporting credentialing reports view ... a technical documentation gate on the embed token endpoint s behavior ... the real authorization decision does this user hold the p |
| payments | domain | PaymentTransaction | invariants | 62 | paid refunded partiallyrefunded transactions cannot revert to pending or failed enforced at the database layer via triggers not just application logic refunded is a terminal state bulk payme |
| proflearning | permissions | proflearning.attendance.override | notes | 62 | gap no api operation is gated by this permission specifically post certification adjustment is instead handled entirely through the scechcorrectionrequest workflow submitcorrectionrequest ap |
| profpractices | permissions | profpractice.markers.manage | notes | 58 | profpractice sequences md s manage account markers sequence instead names finer grained permission ids per marker account marker set mandatory hold account marker clear mandatory hold accoun |
| staffing | permissions | staffing.employee-roster.bulk-upload | notes | 58 | confirmed by staffing domain md s resolved open question 3 citing fdd 15 5 no matching bulk upload endpoint exists anywhere in staffing api yml left with no permissionrequirement an intentio |
| profpractices | permissions | profpractice.status.update | notes | 55 | source lists worklist specific as an applicable scope see file header note profpractice sequences md s review disclosure and update status sequence header instead names the permission id pro |
| documents | permissions | documents.admin.force-hard-delete | notes | 54 | an ... no matching endpoint in documents api yml the closest existing endpoint post documents override requests requestid execute performs a soft delete per its own spec performs soft delete |
| profpractices | permissions | profpractice.worklist.assign | notes | 49 | no dedicated api operation models this permission separately profpractice api yml s put worklists worklistid assignedreviewerids field of worklistcreate schema is the closest match but is ga |
| proflearning | permissions | proflearning.catalog.search | notes | 45 | unauthenticated ... consistent with searchcatalog s x permissions required in proflearning api yml and plrn op searchcatalog s sd requiresauth false no permissionrequirement is modeled for t |
| staffing | permissions | staffing.audit.decertify | notes | 45 | isd resa entity scope no decertify endpoint exists anywhere in staffing api yml though the aggregate s own de certified statedefinition and finalized audit becomes read only except during de |

## Re-applying

The files are named `<area>-<doc kind>.json`. To put the graph's text back for chosen fields:

```bash
node scripts/reapply-deltas.ts <area> --kind permissions --property notes --min-words 25
node scripts/reapply-deltas.ts <area> --kind permissions --property notes --min-words 25 --apply
```

Run from the editor folder. The first command lists what would be restored; `--apply` writes it. Filters are `--kind`, `--property`, `--item` (matches part of the item name) and `--min-words`. Restoring the graph's text for a field puts it back in front of the doc's wording in the next export, so the generated doc will differ from the working doc there.

Because the doc text now wins everywhere, a sensible next step is to look at the review-finding group (kind 1) and decide where those belong: as annotations on the permission, or as entries in the gaps list.
