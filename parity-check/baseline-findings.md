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

## Correction: what the client folder actually is (2026-10-06)

The client repo (`solution-design-documentation`, now at `C:\Git\MiEdWorkforce\solution-design-documentation`) holds its content in a single commit dated 2026-02-24, "Add updated solution design documentation", on the branch `NicholsM11/solution_docs`. Its `main` branch has only a README. So it is the version cleaned and delivered in February, not a lighter copy of the current docs.

Compared with `design/working-docs/` file by file (whitespace ignored), measured 2026-10-06:
- It has no files the working docs lack. 24 of 56 common files are identical.
- The rest differ in both directions, and the differences are design changes made after February, not formatting. Examples: Professional Practices status values changed from `Cleared/UnderReview/Blocked` to `Clear/ConditionalClearance/Hold/Blocked`; the event `ProfessionalPracticeClearanceUpdated` became `PPRClearanceAssessmentChanged`; the Worklist platform capability was removed (routing is now "the application list endpoint filtered by status", not a separate service); Staffing no longer calls Mi-Key directly and goes through IAM's identity-resolution API; EPP's worklist aggregates became filtered views. The titles also changed from "X - Domain Documentation" to "X" plus a Display Name line.
- The largest gaps are IAM (about 430 more lines of sequences, 230 of domain doc and 550 of API in the working docs) and the credentialing and staffing API specs.
- The graph follows the newer design: the old event name and `EPPWorklist` appear 0 times in it, against 2 and 3 times in the client docs.

So the client has an earlier design than the graph and the working docs. The working docs (identical in content to `as-ingested`) are the right source for the graph. The client repo is the right diff target when delivering the next version, as a record of what has changed since February.
