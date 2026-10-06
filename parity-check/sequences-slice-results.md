# Sequences Slice: Results

Second slice of the export loop, 2026-10-05, on the credentialing sequences doc.

- Baseline: `design/baselines/as-ingested/credentialing/credentialing-sequences.md`
- Generated (from the live graph, before re-ingest): `design/generated-from-graph/credentialing/credentialing-sequences.md`

## What was found

The graph kept the text of every sequence section but lost its structure. The baseline has the same template on all 18 sequences (What, When, Who, a Mermaid diagram, then Key Decisions, State Changes, Events Published, Error Scenarios). In the graph, each of the text sections was flattened into one run-on sentence: bullets, bold labels, backticks and line breaks are gone, and list items are joined with periods. That cannot be reliably split back apart, so the fix is to store the text as authored (markdown) and re-ingest it from the baseline.

Two other gaps:
- "State Changes" had no property at all. Added `sd:stateChanges` to the ontology.
- Sequence order had no home. Uses the `sd:order` added in the permissions slice.

## Comparing the words, ignoring formatting

Per field across the 18 sequences, comparing letters and digits only:

| Field | Same words | Different words |
|---|---|---|
| What / When / Who | 16 | 2 |
| Key Decisions | 12 | 6 |
| Events Published | 18 | 0 |
| Error Scenarios | 18 | 0 |
| State Changes | none in graph | 18 to add |
| Mermaid diagrams | same after trimming trailing whitespace | 0 |

All 8 differing fields have the same character: the graph lost cross-references and review notes that the doc carries, for example "see manual review triggering", "(added 2026-08-27 per FDD 29's apply/renew landing page action...)", "Open item (2026-08-27)" notes, and "Note: this sequence represents...". Three are pure losses on the graph side. Five also have a few words the graph has and the doc does not.

Other items the report surfaces:
- One sequence ("Issue Temporary Permit - Exceptional Cases (Admin)") has an `unconfirmed` status callout before the diagram. The graph folded that text into Key Decisions, and the status is not shown in the doc. Its Mermaid title uses a hyphen in the doc and an em dash in the graph.
- On "Application Manual Review and Approval", the doc has a separate "Note:" paragraph that the graph merged into the Who line.

## Effect of re-ingesting (tried on a scratch copy of the graph, live graph untouched)

Applying the safe changes (same-words fields as markdown, State Changes, sequence order, and the three conflicts where the doc only adds words):

| | Lines differing from the baseline, ignoring whitespace |
|---|---|
| Before | 692 added, 849 removed |
| After | 10 added, 38 removed |

All 18 State Changes sections appear. What remains is the 5 two-way conflicts, the callout and Note paragraph, and the dash in one Mermaid title.

## What was built

All in `supporting-artifacts/solution-design-editor/`.

| Piece | Where |
|---|---|
| `sd:stateChanges`, and a note that sequence text fields hold markdown as authored | `ontology/solution-design.ontology.ttl` |
| Sequences exporter in the shared core | `editor/src/lib/export/sequencesDoc.ts` |
| Baseline parser | `editor/scripts/sequences-doc.ts` |
| Compare and re-ingest tool | `editor/scripts/reconcile-sequences.ts` |
| Editor export dialog ("Export docs..." in the header) | `editor/src/components/ExportDialog.tsx`, `editor/src/lib/exportDir.ts` |

The reconcile tool reports first and changes nothing without `--apply`. It classifies each field as same words, conflict (and which way: doc has more, graph has more, or both), or new. Conflicts are never overwritten unless asked: `--take-doc-superset` overwrites only the cases where the doc strictly adds words, `--take-doc` overwrites all.

```bash
node scripts/reconcile-sequences.ts credentialing
node scripts/reconcile-sequences.ts credentialing --apply --take-doc-superset
```

## Graph file serialization

A side finding that matters for tracking the graph in git. The editor's old Save wrote triples in oxigraph's internal order, which changes with how the store was built. Re-dumping an unchanged file reshuffled nearly every line, and the old repository history shows single edits rewriting thousands of lines. Save now writes canonical Turtle (sorted subjects, predicates and values, one value per line), so an edit changes only its own lines. Measured on the graph: lossless (27,814 of 27,814 triples), idempotent, independent of how the store was built, and changing one value changes 1 line out of about 32,000. The graph file was rewritten once in canonical form before its first commit, and the two `sd:order` sidecar files were folded into it.

## Not done yet

- The re-ingest has not been applied to the live graph. It is best applied after the first commit of `design`, so the change shows up as a reviewable diff.
- The 8 conflicts need a decision on which text is right.
- The export dialog has not been run through a real export from the browser, since the folder picker is a native dialog. The same export code was verified from the command line, and the dialog was checked for rendering against the demo graph.
