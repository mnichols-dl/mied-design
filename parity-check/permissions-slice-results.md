# Permissions Slice: First Results

First end-to-end run of the export loop, 2026-10-05, on the credentialing permissions catalog.

- Baseline: `SolutionDesign/solutions/MiEdWorkforce/inputs/original-solutioning/credentialing/credentialing-permissions.md`
- Generated: `SolutionDesign/solutions/MiEdWorkforce/export/credentialing/credentialing-permissions.md`

## What was built

All in the `SolutionDesign` repo.

| Piece | Where |
|---|---|
| `sd:order` property (layout order only, spaced by 10, sibling-scoped) | `ontology/solution-design.ontology.ttl`, with shape entries on `Permission` and `PermissionScope` in `shapes/solution-design.shapes.ttl` |
| Pure exporter core, no browser or Node dependencies | `editor/src/lib/export/` (`graphReader.ts`, `markdown.ts`, `permissionsDoc.ts`, `index.ts`, `types.ts`) |
| Headless CLI | `editor/scripts/export-docs.ts`, loader in `editor/scripts/nodeStore.ts` |
| Order seeding from a baseline doc | `editor/scripts/seed-permission-order.ts` |
| Order data, as sidecar files the editor will fold into the main file on its next Save | `solutions/MiEdWorkforce/solution/_order-credentialing-permissions.ttl`, `_order-scopes.ttl` |

Run from `editor/`:

```bash
node scripts/export-docs.ts credentialing
```

The exporter prints a coverage report: nodes emitted, and any properties on those nodes that the template does not render. For permissions, nothing was left unrendered.

The editor UI button is not built yet. The core takes a store as a parameter, so wiring it in is a thin step. The CLI currently has its own small loader, which should become shared with the editor's loader at that point.

## Result

A whitespace-insensitive diff finds no difference outside the table: the header, intro, and section heading match. Row order matches. Table content, compared cell by cell:

| Where | Baseline | Generated | What it tells us |
|---|---|---|---|
| `credentialing.reports.view` | Row present | Absent | Graph has a Reporting-owned permission instead; relocation, not loss |
| `credentialing.endorsement.add` scopes | `Self-only (via application)` | `Self-only` | The qualifier was moved into the notes cell when the graph was built. A scope with a qualifier has no home |
| `credentialing.endorsement.add` notes | `Educators submit applications; auto or manual approval` | `Self-only (via application); educators submit applications, auto or manual approval` | Same item, other half |
| `credential.print`, `endorsement.view` scopes | `Self-only, System-wide` | `System-wide, Self-only` | One global scope order cannot match both orders the doc uses. Cosmetic |
| Notes on `application.approve`, `deny`, `place-hold` | Contains `` `credentialing-domain.md` `` | Backticks gone | Inline formatting was dropped when the notes were stored as plain text |
| Notes on `application.manual-submit` | Ends with a pointer to the sequences doc | Pointer removed | Graph text is shorter than the baseline |
| Notes on `credential.issue` | Longer, with bold text and a sentence about removed framing | Shorter | Graph text is shorter than the baseline |
| Notes on `application.place-hold`, `credential.reinstate` | Short | Longer, with extra sentences about missing API operations | Graph is ahead of the baseline (review findings written into notes) |

## Takeaways

1. Substance parity is high for this doc. Of 31 baseline rows, 30 reproduce with no differences in category, ID, or description, and the differences above are all in the scopes and notes columns.
2. Three different kinds of difference show up and call for different responses:
   - **Graph is behind or reworded** (formatting lost, text shortened, scope qualifier merged into notes). These are ingestion losses or edits to review against the baseline.
   - **Graph is ahead** (extra review sentences in notes). Needs a decision on whether review findings belong in the client-facing notes or on an `sd:Annotation`.
   - **Template limit** (global scope order). Fine to accept, or a per-row ordering to add later.
3. Notes holding rich text (backticks, bold) suggests string attributes need to allow markdown in the editor, and the exporter must pass them through untouched.
4. The permissions header is not uniform across domains. Only credentialing has the `Domain` and `Version` lines, titles vary ("(DRAFT)", dash style), and IAM and Communications have extra sections. `Version` has no home in the graph, so it sits in template data for now.
