# Design

The MiEdWorkforce solution design: the graph, what is generated from it, and the working docs it was built from. This folder is meant to be a git repository, so changes to the graph and to the generated docs show up as diffs.

| Folder | What it is | Edit it? |
|---|---|---|
| [graph/](graph/README.md) | The Solution Design Graph (`MiEdWorkforce.ttl`), edited through the Solution Design Editor | Through the editor, not by hand |
| [generated-from-graph/](generated-from-graph/README.md) | Docs regenerated from the graph by the exporter. This is the diff target for the parity check | No. Regenerate instead |
| [working-docs/](working-docs/) | The current working solution docs (markdown, OpenAPI, CSV) that the graph was ingested from | Yes, until the graph replaces them |
| [review/](review/) | FDD-versus-design reconciliation: gaps, PO feedback, design spikes, per-FDD review data and wireframes | Yes |
| [CONVENTIONS.md](CONVENTIONS.md) | Naming and IRI conventions for the graph | Yes |

Client-owned source material (FDDs, RTM) is deliberately not in this repository. It lives in the sibling `client-inputs/` folder, which is not under git.

The plan and findings for verifying that the graph reproduces the docs are in [parity-check/](parity-check/) once that folder is moved here.
