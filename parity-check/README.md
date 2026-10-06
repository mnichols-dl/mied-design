# Solution Parity Check

A plan and working log for verifying that the Solution Design Graph can reproduce the delivered solution design docs, and for getting the design to a state we are confident calling complete enough to start development.

Started 2026-10-04. Status: credentialing survey done ([credentialing-survey.md](credentialing-survey.md)); permissions slice ([permissions-slice-results.md](permissions-slice-results.md)) and sequences slice ([sequences-slice-results.md](sequences-slice-results.md)) built and diffed; editor export dialog added. domain doc slice built and tried on a scratch copy ([domain-slice-results.md](domain-slice-results.md)). All 11 areas applied to the live graph ([all-domains-results.md](all-domains-results.md)). Next: review the graph-ahead fields, the remaining sequences differences, then API specs.

Related tracked content, kept in its proper home rather than repeated here:
- Decisions made for this effort: see [DECISIONS.md](../../hub/DECISIONS.md), entry dated 2026-10-04.
- Private concerns: see [tracking/uncertainty-log.md](../../hub/tracking/uncertainty-log.md).
- Work items: see [tracking/tasks/committed.md](../../hub/tracking/tasks/committed.md) and [backlog.md](../../hub/tracking/tasks/backlog.md).
- Baseline comparison of the three doc locations: [baseline-findings.md](baseline-findings.md).

## Goal

Call the solution design complete and comprehensive enough to initiate development. Getting there requires five things, in rough priority order:

1. **Parity.** Prove the graph holds everything the delivered docs hold, by regenerating the docs from the graph and diffing against the originals. This is the top priority and the gate for everything else.
2. **Confidence in editing through the graph.** Once parity is shown, edit in graph form and regenerate updated artifacts, rather than editing docs by hand.
3. **Editor reusability.** Evolve the Solution Design Editor so a missing item (for example an API endpoint) can be added intuitively. Gaps in the editor are not fully known and will surface through use, mostly through the parity loop itself.
4. **Sequence consistency.** Make sequences consistent in granularity and style, in particular by not repeating cross-cutting plumbing (for example the cached authz permission check) in every sequence that touches the API.
5. **LLM-reviewable slices.** Be able to export a targeted slice of the design (one sequence plus the operations, permissions, entities, and rules around it) in a form that makes it easy for an LLM to review and suggest changes.

## Starting situation

- The client-facing "solution docs" are partially finished. They hold a lot of content, but we are not confident they are fully shored up. We lack latitude to reorganize them without solid justification.
- The graph (in the `SolutionDesign` repo) was built by ingesting these docs. We are not confident it captured everything.
- A single `.ttl` file holds the solution (`solutions/MiEdWorkforce/solution/MiEdWorkforce.ttl`, around 4 MB). The ontology has about 33 classes. A SHACL shapes file drives editor forms and validation, and a set of SPARQL validations exist for orphans, unresolved references, and similar checks.
- The graph has since grown beyond the docs, through the FDD review pass (requirements, `satisfiedBy` links, annotations, gaps).

## Approach

### Opinionated export, diffed visually

Build an export in the editor tooling that generates the doc set from the graph, using one template per doc type. Then diff the generated files against the baseline originals with a visual diff tool, by hand.

The point of reviewing by hand is judgment. The aim is **substance parity**, not byte parity. Whitespace, wrapping, table padding, and minor phrasing differences are expected and should not be treated as divergence. To support that:

- Normalize on export where it is cheap (consistent wrapping and table formatting), and run both sides through the same markdown formatter before diffing.
- Use whitespace-ignoring modes in the diff tool.
- Have the exporter print a **coverage summary**: node counts by class that were emitted, and node counts that were skipped. Content that is in the graph but not yet exported should be visible as a known to-do, not look lost.
- Produce a categorized residue report: sections present in the original but absent from the export, and the reverse. This points the review at where to look.

### Two layers of equivalence

1. **Semantic.** Parse both sides into sections, tables, sequence blocks, and API operations and compare. This is the real gate. For the OpenAPI yml and the permissions CSV, compare parsed, not as text.
2. **Textual, after normalization.** For tuning layout and templates.

### Ordering in the graph

RDF is unordered, but documents are not. Section order, table row order, and operation order within a tag group all need a home. The ontology has no ordering property today, and sequence `sd:content` is opaque Mermaid text (see the comment on `sd:Sequence` in the ontology).

Options considered:

| Option | Fit |
|---|---|
| `sd:order` integer, scoped to a parent | Preferred for document layout. Simple, easy to edit in forms, diff-friendly. Insertion means renumbering, so space values by 10 or have the editor renumber on save. |
| `rdf:List` or `rdf:Seq` | Native, but awkward in Turtle, awkward in a form UI, and noisy in git diffs. |
| `sd:precedes` or `sd:next` edges | Better where order is itself the content, for example steps in a sequence. |

Plan: add `sd:order` for layout ordering first. Keep layout order separate from semantics, so it says how an export arranges things and says nothing about the design. Decide on step-level ordering for sequences only after seeing what the credentialing sequences diff shows.

### Templates

Expect the templating, organization, and layout not to be fully modeled by the graph. That is fine. Parts of the graph not yet exported are fine too; the exporter can grow into them. The templates in the older solutioning repo (`docs/templates/template-*.md`) are natural seeds for per-doc-type export templates.

### Headless exporter

Build the exporter as a CLI-callable module, not only an editor button, so it can run in a diff loop or in CI. The editor then calls into it. One exporter serves three uses: round-trip verification, client-facing docs, and scoped LLM review slices.

## Baseline

The baseline for diffing is `SolutionDesign/solutions/MiEdWorkforce/inputs/original-solutioning`, the exact content the graph was ingested from. It is effectively identical to the `MiEdWorkforce Solutioning` repo. The client-facing folder is behind both. Details and file lists are in [baseline-findings.md](baseline-findings.md).

## Staged plan

Pilot domain: **credentialing**. It is one of the larger domains, it had a later v2 pass that populated business rules and annotations, and it differs from the client copy in every file, so it will exercise the ontology well.

Work in stages and diff after each, rather than exporting everything and diffing once:

1. **Survey (done).** Line the graph up against the five `credentialing-*` files in the baseline. Result in [credentialing-survey.md](credentialing-survey.md). It changed the order below: the API file turned out to be the least reproducible, not the most mechanical, because the graph holds only operation summaries and links to domain shapes, not parameters, responses, or schemas.
2. **Ordering.** Add `sd:order` to the ontology and shapes, and set it on the credentialing nodes.
3. **Permissions.** Headless export of `credentialing-permissions.md`, plus the credentialing slice of `solution-permissions.csv`. Most mechanical, so the cheapest first signal. Review the diff and the coverage summary before moving on.
4. **Sequences.** `credentialing-sequences.md` and the generated diagrams. Structured and ID-complete (18 of 18). Known gap: no home for "State Changes". Decide here how much sequence structure should be modeled.
5. **Domain doc.** `credentialing-domain.md`. Prose-heavy, with the largest set of gaps (ubiquitous terms, open questions, scope list, entity relationship diagram).
6. **Technical design.** `credentialing-technical-design.md`. Mostly prose; likely needs a way to carry ordered prose sections.
7. **API spec.** `credentialing-api.yml`, last, because it needs a design decision about how much OpenAPI detail the graph should own versus link to or store as text.
8. **Retrospective.** Record what the loop revealed about ontology, templates, and editor gaps. Then repeat for the next domain.

## Beyond the pilot

### Sequence consistency (item 4 above)

Do not fix this by hand-editing every sequence. Model cross-cutting behavior once, for example as a declared policy on the API or the solution ("all API calls check the cached authz result"). The exporter renders it as a legend note, and sequence generation leaves it out. A lint query can then flag sequences that include it explicitly. That makes consistency enforceable rather than a matter of discipline. This depends on how sequences end up being modeled; see stage 5.

### LLM review slices (item 5 above)

A scoped export of one sequence plus its operations, permissions, entities, and rules, produced by the same exporter, is the review package. Likely implemented as a SPARQL CONSTRUCT neighborhood rendered through the same templates.

### What parity does and does not prove

A clean round trip proves the graph contains what the docs contain. It does not prove the design is complete. Completeness still comes from the existing review work in the `SolutionDesign` repo (`review/GAPS.md`, `PO-FEEDBACK.md`, `DESIGN-SPIKES.md`, `STATUS.md`) and the SPARQL validations. Keep the two separate so a clean diff is not mistaken for "ready for development".

### Scope boundary for export

The graph holds more than the docs, including FDD review state, annotations, and gaps. The exporter needs an explicit line for what is client-facing, so draft review state does not leak into deliverables. Default: export only what the baseline docs contain, and add more deliberately.

### Presenting results to the client

Once a domain diffs clean against the baseline, commit the generated set as the new baseline. From then on every change is a reviewable diff. A diff of the generated set against the client folder gives a ready-made changelog of everything delivered since their last copy. If a restructure is wanted later, the layout-only diff is the concrete justification for it.

## Out of scope for now

- Folding the `MiEdWorkforce Solutioning` repo into this one. Parity work uses its content as reference only; migration stays on the backlog.
- Rewriting how the client docs are organized.
- Importing docs into the graph (the other direction). Only the export direction is being built here, apart from coverage checks.

## Locations (updated 2026-10-05)

This folder moved here from `hub/solution-parity-check/`, and the other pieces moved with the layout change recorded in [DECISIONS.md](../../hub/DECISIONS.md). Where the older notes in this folder say a location, read it as follows:

| Older notes say | Now |
|---|---|
| `SolutionDesign/solutions/MiEdWorkforce/inputs/original-solutioning` | the `baseline-docs` git tag of `design` (the folder was removed; `working-docs/` was identical in content) |
| `SolutionDesign/solutions/MiEdWorkforce/solution/MiEdWorkforce.ttl` | `design/graph/MiEdWorkforce.ttl` |
| `SolutionDesign/solutions/MiEdWorkforce/export/` | `design/generated-from-graph/` |
| `SolutionDesign/editor` and `SolutionDesign/ontology` | `supporting-artifacts/solution-design-editor/` |
| `MiEdWorkforce Solutioning` solution docs | `design/working-docs/` |
| `SOM-AzDO/solution-design-documentation` (client folder) | `solution-design-documentation/` |

Run the export from `supporting-artifacts/solution-design-editor/editor/`:

```bash
node scripts/export-docs.ts credentialing
```
