# Domain Doc Slice: Results

Third slice of the export loop, 2026-10-06, on the credentialing domain doc and the permissions scope column. The survey that led here is in [domain-doc-survey.md](domain-doc-survey.md).

## Source and style

- **Source:** `design/working-docs/` is the source for ingestion. It is identical in content to `baselines/as-ingested/` (see [baseline-findings.md](baseline-findings.md)).
- **Style of the generated doc:** it follows the client-delivered docs: `# <Display Name>`, then `Type`, `Identifier` and `Primary Sources` lines, then a rule. The "Display Name" line that the working docs added is dropped, since it repeats the title. The client's own titles were inconsistent ("X - Domain Documentation" on some docs, plain "X" on others), so the generated docs use the plain display name for all.

## The model: two layers

All 11 domain and capability docs share one template, so one parser and one exporter serve them all. What is stored, and how:

| Layer | Holds | Where |
|---|---|---|
| Structured, wording verbatim from the doc | Terms (label, definition, order), events (name, trigger, payload, consumers, order), rule fields (title, rule, rationale, enforced-by prose, example, order), aggregate fields (purpose, root entity, invariants, key states, order), purpose | The node's own properties |
| Authored markdown on the owning node | Extra labelled blocks inside rules and aggregates (with `sd:blockOrder` to keep their position), the entities and value objects list, "referenced in", the scope lists, the dependency tables, the entity relationship diagram, primary sources, event naming convention | `sd:extraSections`, `sd:entitiesAndValueObjects`, `sd:referencedIn`, `sd:owns`, `sd:doesNotOwn`, `sd:dependenciesMarkdown`, `sd:erDiagram`, `sd:primarySources`, `sd:eventNamingConvention` on the bounded context or aggregate |

Where a markdown block names things the graph also models as nodes (entities and value objects, sequences, event consumers), the node is the navigable index and the markdown is the text the doc shows. The event Consumers column can be derived from other contexts' subscriptions, and for credentialing the derived set matched the doc's set in all 20 rows; the doc's wording is stored anyway because the author's order is not derivable, and a validation query should check the two agree.

## What the credentialing graph held (before)

From the dry-run report:
- Aggregate purposes and root entities matched the doc word for word in all 7 aggregates. Event triggers matched in all 20 events.
- 19 of the 25 ubiquitous terms had no node. The 6 that existed were verbatim.
- Rules: 8 of 14 had a statement whose words differed from the doc (paraphrased or compressed), and rule headings existed only as IRIs.
- Event payloads lacked the braces and backticks, and no events or rules carried a name or title.
- Order was absent everywhere. Three rules carry an Example that the doc does not have.

## Result (tried on a scratch copy of the graph, live graph untouched)

Applying the re-ingest with doc text winning (255 changes, 19 new term nodes) and exporting the domain doc: the generated doc differs from the working doc by 8 added and 30 removed lines out of 971, ignoring whitespace. Of those:
- 27 removed lines are the Open Questions section (left out of scope for now).
- 1 removed line is the dropped Display Name line.
- The dash rows of the tables differ only in padding.
- 3 added lines are Examples on rules where the graph has an Example the doc does not (a "graph is ahead" case). The report lists these as graph-only and they are kept, so they appear as additions.

## Permissions: scopes and headings

Run for credentialing: the column headings are identical to the doc's (and to 9 of the 11 permissions docs); all 30 categories match; 27 of 30 scope cells already reproduce from the scope links. The other 3 (two with the scopes in a different order, one with the qualifier "(via application)") are stored in a new `sd:scopeText`, set only where the doc's cell differs from what the links generate, so the links stay the queryable truth. After that all 30 scope and category cells match the doc. One baseline row, `credentialing.reports.view`, has no graph permission (the graph holds a Reporting-owned permission instead).

Across all 11 permissions docs there are 34 distinct scope cell values; several name scopes that are not modeled as scope nodes at all (`EPP-specific`, `Worklist-specific`, `Individual`, `Building`). The reconcile tool lists those per domain.

## Tools

In `supporting-artifacts/solution-design-editor/editor/scripts/`:

| Tool | Purpose |
|---|---|
| `domain-doc.ts` | Parser for the shared domain doc template. Keeps anything it does not recognize rather than dropping it |
| `reconcile-domain.ts` | Compares a doc with the graph section by section, reports same words, conflict (and which side has extra words), new, and graph-only; `--apply` writes, `--take-doc` lets the doc win on conflicts |
| `reconcile-permissions.ts` | Checks headings, categories and scope cells; `--apply` stores scope text and row order |
| `graph-edit.ts`, `text-compare.ts` | Shared helpers |

The exporter is `editor/src/lib/export/domainDoc.ts`, registered as the `domain` doc kind, so it also shows up in the editor's Export dialog.

## Not applied to the live graph yet

The sequences re-ingest from the previous slice was still uncommitted when this was built, so applying more would have mixed two reviews into one diff. After committing `design`:

```bash
node scripts/reconcile-domain.ts credentialing --apply --take-doc
node scripts/reconcile-permissions.ts credentialing --apply
```

## Not done yet

- **Open Questions** are not exported (out of scope for now).
- **Optional prose sections** of the capability docs (Classification Rationale, Technical Considerations, Integration Patterns, Workflows) have no home yet. Credentialing has none, so the planned `sd:DocSection` node was deferred to the first capability domain.
- **Other domains:** EPP and communications place the entity relationship diagram differently from the other nine (not under Domain Model), and the parser reports a few aggregates and a rule it could not fully read in EPP, profpractice and staffing. These are for when each domain is run.
- The aggregate heading is taken from the root entity. If another domain's heading differs from its root entity, aggregates need a title property.