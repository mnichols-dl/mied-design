# All Domains: Results

The full run across the 11 areas, 2026-10-06, applied to the live graph. Builds on [domain-slice-results.md](domain-slice-results.md) (the credentialing pilot) and [sequences-slice-results.md](sequences-slice-results.md). The baseline for every comparison is `design/working-docs/solution-areas/`; the generated docs are in `design/generated-from-graph/`.

## What was applied

For each of the 11 areas, the domain or capability doc, the permissions doc and the sequences doc were reconciled into the graph and exported. Credentialing used the "doc wins" rule you approved. For the other 10 the rule was **doc wins everywhere except where the graph has strictly more words than the doc** (see "Graph is ahead" below). Everything the doc has and the graph lacks was added. The graph diff against your commit is 4,399 lines added and 1,386 removed, and the file is still canonical.

Nodes created by the domain step (glossary terms, business rules the graph lacked, and doc sections), by area: credentialing 19, communications 11, documents 15, epp 12, iam 5, organizations 4, payments 18, proflearning 15, profpractice 19, reporting 6, staffing 9. The permissions step created one permission in each of 7 areas, for rows the doc lists and the graph lacked. The sequences step created 5 nodes in communications and 1 each in IAM and organizations.

## Results by doc kind

Differences from the working docs, ignoring whitespace. For the domain and capability docs the Open Questions section and the Display Name line are excluded, since both are intentional. "Added" is text only in the generated doc, "removed" is text only in the baseline.

| Area | Domain/capability doc (added / removed) | Permissions doc (added / removed) | Sequences doc (added / removed) |
|---|---|---|---|
| credentialing | 8 / 4 | 1 / 1 | 0 / 0 |
| communications | 21 / 26 | 39 / 38 | 25 / 51 |
| documents | 15 / 7 | 7 / 6 | 21 / 29 |
| epp | 7 / 25 | 5 / 5 | 26 / 28 |
| iam | 8 / 8 | 20 / 12 | 31 / 97 |
| organizations | 26 / 2 | 3 / 2 | 19 / 10 |
| payments | 27 / 51 | 13 / 13 | 29 / 61 |
| proflearning | 5 / 13 | 21 / 21 | 70 / 53 |
| profpractice | 22 / 21 | 22 / 22 | 28 / 59 |
| reporting | 1 / 9 | 10 / 4 | 92 / 68 |
| staffing | 8 / 10 | 24 / 24 | 27 / 0 |

### Permissions docs: headings, categories and scopes match everywhere

Compared cell by cell across all 11 docs: **the Category and Applicable Scopes columns match the doc in every row** (0 differing cells), the table headings match, and descriptions match except 3 cells. All remaining permissions differences are in the Notes column. The permissions doc frame (title, preamble, text around the table, extra sections such as Scope Definitions or Permission Patterns, bold category style) is stored and reproduced. Where the doc lists a permission the graph lacked, it was created: one in each of credentialing, communications, documents, epp, payments, proflearning and staffing (`credentialing.reports.view` being the one you confirmed). The other four areas needed none.

### Domain and capability docs

All 11 are within a few dozen lines of the baseline after the known exclusions. What remains is mostly "graph is ahead" text (below), plus content the graph has that the doc does not: extra aggregates (organizations has 2 and documents 1 that the doc does not list), extra rules (communications), and Example text on rules.

### Sequences docs

Most variation, because each doc deviates from the template in its own way:
- **communications**: contains non-sequence sections (standards, a decision matrix, integration flows) and a purge job the graph lacked. These are now doc sections and a created sequence.
- **proflearning**: the graph's sequence titles did not match the doc's headings at all (ingestion renamed them). They were matched by diagram text instead and the doc's headings were adopted.
- **reporting**: one doc section is split into three scenario sequences in the graph (happy path and two denial paths), so the doc section was created and the three graph scenarios remain as extras.
- **IAM, payments, profpractice, EPP**: sequences present on both sides but with differing diagram or section text; not yet examined section by section.

## Graph is ahead

Where the graph has strictly more words than the doc (typically review findings or cross-references added after ingestion), the graph text was kept and the field is listed for you. Counts of fields kept, measured against your commit:

| Area | Domain/capability | Permissions notes | Sequences |
|---|---|---|---|
| communications | 10 | 36 | 13 |
| documents | 5 | 5 | 8 |
| epp | 5 | 4 | 0 |
| iam | 4 | 11 | 0 |
| organizations | 0 | 1 | 0 |
| payments | 22 | 12 | 1 |
| proflearning | 4 | 20 | 0 |
| profpractice | 17 | 21 | 5 |
| reporting | 1 | 3 | 0 |
| staffing | 6 | 23 | 0 |

The cost of keeping them is formatting: the graph's text for those fields is a flattened sentence, so the export shows it as one paragraph where the doc has a bullet list (for example EPP's Key Invariants, which in the graph also carry added "(Business Rule: ...)" references). To take the doc's version for an area instead, run the tool with `--take-doc` (the previous graph text stays in git history).

## Model additions in this pass

All in the ontology, all described there:
- **`sd:DocSection`**: ordered authored prose with a doc kind, for the optional capability sections (Classification Rationale, Technical Considerations, Integration Patterns, Workflows, an appendix), non-aggregate blocks under Domain Model, extra permissions sections, and non-sequence sections of the sequences docs.
- **Per-doc text kept as authored on the context**: events table intro, column headings and footer; scope wording and extras; the permissions and sequences doc titles and preambles; the permissions table lead and footer; category style.
- **Aggregate title** (the doc's heading, which can differ from the root entity), **display name** from the doc's own Display Name line, **`sd:docSlug`** for the one area whose doc name differs from the graph identifier (profpractice vs profpractices).
- **Block order**, with a numbering scheme for repeated labels, so extra blocks stay in position and a label repeated inside one block is kept instead of overwriting.

## Tools

In `supporting-artifacts/solution-design-editor/editor/scripts/`:

| Tool | Flags |
|---|---|
| `reconcile-domain.ts`, `reconcile-permissions.ts`, `reconcile-sequences.ts` | `--apply` to write; `--keep-graph-ahead` (doc wins except where the graph has strictly more); `--take-doc` (doc wins everywhere); `--take-doc-superset` (only where the doc has all the graph has) |

Run one area: `node scripts/reconcile-domain.ts <namespace> --apply --keep-graph-ahead`.

## Not covered yet

- **Open Questions** are not exported (out of scope for now).
- **API specs** (`*-api.yml`) are not in the graph at the detail needed; planned separately.
- **Technical design docs** exist for credentialing and staffing only and are being treated as a one-off for now.
- The **sequences docs** for IAM, payments, profpractice and EPP have differences not yet examined.
- **Consumers column**: for most areas the consumers the doc lists differ from what the graph's subscriptions imply (credentialing matched in all rows; for example IAM differs in 16 of 16 events). The doc's wording is stored; the disagreement is a design question about which is right and needs a validation query.