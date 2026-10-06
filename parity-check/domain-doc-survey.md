# Domain Doc Survey: Baseline vs Graph, All 11 Domains

Read-only survey, 2026-10-05, before building the domain doc export. It covers the six `*-domain.md` docs and five `*-capability.md` docs in `design/baselines/as-ingested/`, compared with the graph.

The survey read design/baselines/as-ingested/. A follow-up check found design/working-docs/ identical to it in content: all 52 domain-area files and the 3 solution-level files match once line endings are normalized (two files in s-ingested use CRLF), so the results below hold for either. working-docs/solution-level/ also holds files that were never ingested (API catalog, permissions CSV, databases and pipelines notes, two feature docs).

## The docs share one template

All 11 docs have the same 8 sections: Purpose, Scope, Ubiquitous Language, Domain Model, Domain Events, Dependencies, Business Rules, Open Questions. The platform-capability docs (communications, documents, organizations, payments, reporting) add some of: Classification Rationale (5 docs), Technical Considerations (5), Integration Patterns (3), Workflows (3). Profpractice has an extra appendix on pre-existing terminology, and communications has its entity relationship diagram as a top-level section where the others nest it under Domain Model. Every doc has one Mermaid entity relationship diagram (profpractice has two).

One exporter and one parser can therefore serve all 11 domains, with the optional sections handled generically.

## Counts, doc versus graph

Doc count first, then graph count. Open questions are left out of scope for now.

| Domain | Terms | Aggregates | Events | Rules | Dependency rows vs nodes |
|---|---|---|---|---|---|
| communications | 16 / 9 | 4 / 4 | 12 / 15 | 6 / 7 | 12 / 11 |
| credentialing | 25 / 6 | 7 / 7 | 20 / 22 | 14 / 14 | 15 / 22 |
| documents | 20 / 7 | 4 / 5 | 13 / 16 | 7 / 7 | 14 / 13 |
| epp | 16 / 5 | 5 / 4 | 12 / 13 | 10 / 10 | 13 / 12 |
| iam | 13 / 9 | 4 / 4 | 16 / 17 | 13 / 13 | 11 / 25 |
| organizations | 10 / 10 | 1 / 3 | 3 / 3 | 4 / 4 | 6 / 12 |
| payments | 23 / 9 | 3 / 3 | 12 / 15 | 7 / 7 | 10 / 9 |
| proflearning | 23 / 8 | 7 / 7 | 12 / 13 | 10 / 10 | 14 / 12 |
| profpractice | 24 / 8 | 8 / 6 | 12 / 12 | 15 / 15 | 15 / 12 |
| reporting | 10 / 6 | 2 / 2 | 0 / 0 | 6 / 6 | 4 / 4 |
| staffing | 17 / 8 | 6 / 6 | 9 / 19 | 12 / 12 | 9 / 12 |

Aggregates, events, and rules are close in count. Ubiquitous terms are the clear gap: the graph holds a third to a half of them in most domains. Dependency counts differ in grain, not just in number (see below). Events in the graph often exceed the doc, since the graph picked up events from the technical design docs and the FDD review.

## What the graph captured, section by section

| Section | What the graph holds | Gaps |
|---|---|---|
| Header (Type, Identifier, Display Name) | `BoundedContext` kind, identifier, displayName | Primary Sources line (BRD references) has no home |
| Purpose | `BoundedContext.purpose` | Rewritten, not verbatim. 0 of 2 sentences match in credentialing, and match rates are low in nearly all domains |
| Scope, owns and does-not-own lists | Does-not-own as one paraphrased blob in `notes`. The owns list has no home | Both lists. 0 of 15 sentences match in credentialing |
| Ubiquitous Language | `UbiquitousTerm` with `prefLabel` and `definition`. Wording is verbatim where a term exists | A third to a half of terms missing, and no order |
| Aggregates | Description, root entity, entities and value objects (name and description), states, invariants | Order lost. Entities and value objects are in two separate properties though the doc interleaves them. Invariants are one blob. The "Referenced In" lists are probably derivable, not checked |
| Entity relationship diagram | Nothing (no `erDiagram` text anywhere in the graph) | Missing in all 11 |
| Domain Events | `trigger`, `payloadHighlights`, `publishedToTopic` | No name property: the event name is the IRI minus a trailing `Event`. Payload braces dropped. No order |
| Dependencies | `Dependency` nodes with mechanism and links to operation or event | The doc's "What We Need" and "How We Get It" columns have no home. Grain differs: the doc has one row per counterpart, the graph one node per mechanism (IAM has 11 doc rows and 25 nodes) |
| Business Rules | `statement`, `rationale`, `example`, links for applies-to and enforced-by | No title property: the section heading is the IRI. The text is compressed, not verbatim ("90 days per assignment" became "90 days/assignment", "Additional 90 days" became "+90 days"). The doc's "Enforced By" prose is replaced by links |
| Classification Rationale, Technical Considerations, Integration Patterns, Workflows | Essentially nothing: 0 to 2 sentences found of 4 to 62 in each section | Whole sections, a few hundred to about 5,000 characters each |

Match rates for the prose sections were measured by whether the first 60 letters and digits of each sentence appear anywhere in the graph file, which is a conservative test.

## Permissions docs

The main table has identical headings in 9 of 11 domains (`Category | Permission ID | Description | Applicable Scopes | Notes`), so heading fidelity is straightforward there. The other tables and sections vary: Permission Patterns, Scope Definitions, Implicit Baseline, Upload Permission Model, Cross-Domain Permissions, Permission Combinations, Implementation Notes, and a Notes section.

The scope cell has 34 distinct values across the 11 docs. The six standard scopes the graph links to cover the common ones, but many carry a qualifier the links cannot hold, for example `Entity (session-scoped)`, `Building, District, ISD (transitive)`, `System-wide (unauthenticated)`, `Self-only (via application)`, and others, plus some scopes beyond the standard six (`EPP-specific`, `Worklist-specific`, `Individual`, `Building`).

## Reading of the results

The graph is a good semantic index of these docs, but it is not a faithful representation of them. It captured most of the items and many of the relationships, and it added structure the docs lack (links from rules to aggregates and sequences, for example). It did not capture the docs' words reliably: text was paraphrased or compressed, headings were left to the IRI, order was dropped, about half the glossary was skipped, and the optional prose sections were left out.

That is the same pattern found in sequences, where it turned out to be recoverable by re-ingesting the doc text with the doc winning on conflicts (see [sequences-slice-results.md](sequences-slice-results.md)).
