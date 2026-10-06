# Graph

`MiEdWorkforce.ttl` is the whole solution design graph. It is written by the Solution Design Editor's Save, so change it through the editor rather than by hand. In the editor, pick this folder (not `design/`) when loading, so the review data under `../review/` is not loaded a second time.

Other files here:
- `wireframes/` holds wireframe images referenced by `sd:imagePath`, relative to this folder.

The file is written in canonical form (sorted, one value per line), so a git diff of it shows only what changed. The editor's Save produces the same text. To check or restore canonical form from the command line, run `node scripts/canonicalize-graph.ts --check` from the editor folder.
