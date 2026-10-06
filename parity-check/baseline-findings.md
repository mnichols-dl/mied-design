# Baseline Findings

Comparison of the three locations that hold solution design docs, run 2026-10-04 by content hash over `.md`, `.yml`, and `.csv` files (excluding archive, generated, templates, drafts, and review folders).

| Label | Path |
|---|---|
| Original | `C:\Git\SolutionDesign\solutions\MiEdWorkforce\inputs\original-solutioning` |
| Solutioning | `C:\Git\MiEdWorkforce Solutioning\solution-areas` (plus `solution-level`) |
| Client | `C:\Git\SOM-AzDO\solution-design-documentation` |

The graph (`solutions/MiEdWorkforce/solution/MiEdWorkforce.ttl`) was ingested from Original.

## Original vs Solutioning

Effectively identical. Only two files hash differently, `proflearning-sequences.md` and `staffing-technical-design.md`, and a line-by-line comparison shows no content difference in either, so the cause is most likely line endings. Solutioning has many additional files (FDDs, patterns, misc notes, drafts) that are outside the domain docs and do not affect the parity check.

**Conclusion:** Original is the baseline. No newer edits are hiding in Solutioning.

## Original vs Client

The Client folder is behind. About 30 files differ, and the Original version is larger in nearly every case.

| File | Original | Client |
|---|---|---|
| `iam-sequences.md` | 60 KB | 37 KB |
| `iam-domain.md` | 60 KB | 41 KB |
| `iam-api.yml` | 85 KB | 63 KB |
| `iam-permissions.md` | 10.5 KB | 8.4 KB |
| `credentialing-sequences.md` | 65 KB | 53 KB |
| `credentialing-domain.md` | 73 KB | 64 KB |
| `credentialing-api.yml` | 80 KB | 72 KB |
| `staffing-domain.md` | 48 KB | 42 KB |
| `epp-domain.md` | 44 KB | 39 KB |

Other differing files: communications (capability, technical design), documents (capability, sequences), epp (api, permissions, sequences), organizations capability, payments (capability, sequences), proflearning (api, domain, sequences), profpractice (domain, sequences), reporting capability, staffing (api, permissions, sequences), and the solution-level `solution-architecture.md` and `solution-tech-standards.md`.

Present in Original but absent from Client: `credentialing-technical-design.md`, `staffing-technical-design.md`, `solution-integrations.md`.

Present in Client but not in the Original folder: `api-design.md`, `event-sourcing.md`, `README.md`, `solution-api-catalog.md`, `solution-permissions.csv`. The first four also exist in the Solutioning repo under its own layout; they are patterns or solution-level files rather than domain docs.

Heading drift example: credentialing sequences has "Issue Temporary Permit - Exceptional Cases (Admin)" in Original and the plainer "Issue Temporary Permit (Admin)" in Client.

File timestamps on the Client side are January and February 2026, versus August and September 2026 for Original. Timestamps from clones can mislead, so "older" is likely but not proven by this alone.

## Implications

1. The parity baseline is Original.
2. The client has not seen much recent work. IAM looks the furthest behind.
3. Once a domain diffs clean against Original, a diff of the generated output against the Client folder is a free changelog of what has been delivered since.
4. Much of the divergence from the Client copy is content growth, not layout. Check section structure against the Client copy before changing any layout, since restructuring something the client has already seen is a harder call than adding content.
