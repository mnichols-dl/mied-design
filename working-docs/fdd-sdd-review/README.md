# FDD ↔ SDD Review Process

## Purpose

The client has produced **Functional Design Documents (FDDs)** independently of this
repository's **Solution Design Documents (SDDs)** — the domain, permissions, sequences,
technical-design, and API docs under `solution-areas/`. The two were not written against
each other, so they need to be reconciled.

This process treats FDDs as **input, not ground truth**. FDDs may describe things at the
wrong altitude (prescribing a technical implementation instead of a business rule), may
use different terminology than what's been locked in during solutioning, and may describe
scope that already exists, doesn't exist yet, or was deliberately excluded. The review
exists to surface all of that systematically, one source document at a time, so gaps in
the SDDs get found — not to make the SDDs conform to the FDDs by default.

## Identifiers — use the source file, don't invent one

There is no surrogate review ID (no `FDD-001`-style numbering). A document's identifier
**is its path**, relative to `functional-design-docs/` — e.g.:

```
01 - User Management/1 - User Management.docx
01 - User Management/MiLogin Interface/1.1 - MiLogin Citizen and Business/1.1 - MiLogin Business IDD.docx
15 - User Management - Identity Management Integration/15.4 - Person Search.docx
```

The client's own numbering embedded in these names (01, 1.1, 8.2, 15.4, …) is already a
real identifier — reuse it when referring to a document informally (e.g. "the 15.4 doc"),
but the authoritative key for tracking and tagging is the relative path, since numbering
alone isn't always unique or present (some supporting files — spreadsheets, IDDs — have no
number at all).

**Markdown siblings are not separate documents.** Every `.docx`/`.doc`/`.xlsx`/`.xls` file
has (or will have) a `.md` file written alongside it, same name, same folder — an
extracted-text working copy of the same content, produced mechanically (see
[Converting source files](#converting-source-files) below). Treat `foo.docx` and `foo.md`
as **one identity, two representations** — never a reason to add a second tracker row or
a second tag. Read the `.md` for review work (faster, greppable); fall back to the
original binary only when you need something the extraction lost (embedded images,
cell formatting/formulas, a diagram's visual layout).

## Scope of each review

For every source document (or closely-related set) reviewed, answer four questions:

1. **Altitude check** — Does the document contain content that is too technical or overly
   prescriptive for a *functional* design doc (e.g. it specifies a database schema, a
   caching strategy, a specific library, an API contract, or an algorithm)? That content
   may belong in an SDD technical-design doc instead — or may be a boundary violation
   worth pushing back to the client on, rather than adopting.
2. **Discrepancy check** — Where the FDD and the SDD both describe the same behavior,
   entity, rule, or terminology, do they agree? Flag contradictions, terminology drift,
   and cases where the FDD's description no longer matches a decision already made in
   solutioning. This includes discrepancies that turn out to be **internal** — two SDD
   docs disagreeing with each other, or the FDD material contradicting itself across its
   own source files — see the note under Overall Status.
3. **Coverage check** — Does the FDD describe anything — a rule, a workflow, an entity,
   an integration, a permission — that isn't represented anywhere in the SDDs yet? These
   are the actual gaps this process exists to find.
4. **Tagging** — Record which solution-area domain(s), aggregates, and sequences the
   document maps to, so it's traceable from the SDD side too, regardless of what the
   review concludes.

## Folder contents

| File | Purpose |
|---|---|
| [tracker.md](tracker.md) | Master list of every source document in scope, one row each, with its own status and a link to its review record. Start here to see what's left to review. |
| [templates/template-fdd-review.md](templates/template-fdd-review.md) | The template used to produce one review record, covering one document or a closely-related set. |
| [reviews/](reviews/) | One completed review file per document or document set, named after the FDD folder/topic (e.g. `01-user-management.md`), not a surrogate ID. |
| [tag-index.md](tag-index.md) | Aggregated, queryable index of every FDD ↔ domain/aggregate/sequence tag produced across all reviews. Regenerated/appended to as reviews complete — this is the answer to "what source document backs this part of the design?" and vice versa. |
| [client-questions.md](client-questions.md) | Flat, prioritized list of every open question raised **to the client** across all reviews — the working list for client conversations. Internal-only open questions stay in their review file and aren't duplicated here. |

Source documents themselves live in [functional-design-docs/](../functional-design-docs/)
— that folder is inputs only; this folder (`fdd-sdd-review/`) is the working record of
reviewing them.

## Converting source files

Before review, `.docx`/`.doc`/`.xlsx`/`.xls` files get a `.md` sibling extracted
mechanically (paragraphs/headings/tables for Word, one section per sheet for Excel).
This has already been run once across the full `functional-design-docs/` tree. To
convert newly-added files, rerun the same extraction (python-docx for Word, openpyxl for
`.xlsx`, xlrd for legacy `.xls`) against any file missing a `.md` sibling — it's
idempotent, so re-running over the whole tree only fills in gaps.

Not every format converts automatically today:
- Legacy binary **`.doc`** (pre-2007 Word) has no installed converter (would need
  LibreOffice or antiword) — currently unconverted, listed in the tracker.
- Other formats present in the corpus (`.pdf`, `.pptx`, `.vsdx`/`.vsd`/`.vsdm` diagrams,
  `.xmind`, `.msg` emails, `.url` shortcuts) are **out of scope for automatic conversion**
  by design decision — review these directly from the original file when a tracker row
  needs one, rather than adding more converters speculatively.

## Workflow

1. **Intake.** Drop the new file(s) into `functional-design-docs/`, in a subfolder
   matching the client's existing numbering scheme where one exists. Convert it (see
   above). Add one row per source file to [tracker.md](tracker.md) with status
   `Not Started`. Do not invent a grouping ID — the path is the identifier. It's fine
   (expected, even) for many rows under the same folder to end up with different
   statuses once reviewed (e.g. a folder's main narrative doc might be `Aligned` while an
   interface-design sub-doc bundled into the same folder is a `Boundary Violation`).

2. **Review.** Copy `templates/template-fdd-review.md` to `reviews/<topic-slug>.md`
   (named after the FDD folder/topic, e.g. `01-user-management.md`) and work through its
   four sections against the relevant `solution-areas/*/*-domain.md` (or `-capability.md`),
   `-permissions.md`, `-sequences.md`, and `-technical-design.md` files. One review file
   may cover several related source documents (a main FDD plus its interface-design
   sub-docs and supporting spreadsheets) — list all of them in the review's header table.
   Update each covered row's tracker status to `In Progress` when you start.

3. **Determine overall status per document.** Status is tracked **per source document**,
   not once per review file — a review file covering 7 source files can and often will
   produce 7 different statuses (see the tie-breaking note below only applies *within* a
   single document's own mixed findings, not across documents in the same review file).

4. **Act on findings.** Depending on what's found:
   - A **coverage gap** becomes SDD work — either flag it for follow-up in the relevant
     `-domain.md`'s Open Questions table, or go ahead and draft the missing section if
     it's straightforward.
   - A **discrepancy** gets resolved by deciding which side is correct and updating the
     other — record the decision in the review file, don't leave it silently fixed.
   - An **internal SDD contradiction** (two SDD docs disagree with each other, surfaced
     while comparing against the FDD) gets flagged explicitly as its own finding, not
     folded silently into a normal Discrepancy row — see the Overall Status note below.
   - A **boundary violation / altitude issue** gets flagged back to the client rather than
     silently absorbed — note the specific recommended terminology or reframing in the
     review file so it's ready to raise.
   - **Tags** get added to [tag-index.md](tag-index.md) regardless of the other outcomes.
   - Any Open Question with `Raised To: Client` also gets a row in
     [client-questions.md](client-questions.md), with a Priority set — that file is the
     working list for client conversations, separate from the full per-review record.

5. **Close out.** Update each covered tracker row's status to its final per-document
   Overall Status and link the completed review file.

## Overall Status values

Use exactly one of these per source document:

| Status | Meaning |
|---|---|
| `Not Started` | Document is logged in the tracker but review hasn't begun. |
| `In Progress` | Review is underway. |
| `Aligned` | FDD and SDD agree; no discrepancies, no material gaps, no boundary concerns worth raising. Nothing to do. |
| `Aligned — Minor SDD Revisions Needed` | Fundamentally consistent; small clarifications, terminology tweaks, or additions needed in the SDD. |
| `Aligned — Minor FDD Feedback Recommended` | Fundamentally consistent; small terminology or clarity issues worth flagging back to the client, but not blocking. |
| `Gaps Identified` | The FDD describes meaningful scope (a rule, workflow, entity, integration) not yet represented anywhere in the SDDs. SDD work is required. |
| `Discrepancy — Needs Decision` | FDD and SDD conflict on a substantive point (a rule, a state, a terminology meaning) and a call must be made on which is correct before proceeding. |
| `Boundary Violation` | The document, as written, prescribes implementation-level detail (schema, algorithm, specific technology, API contract) that isn't appropriate for a functional design doc. Recommend reframing/pushback to the client rather than adopting as-is. |
| `Needs Client Clarification` | Can't be resolved from the FDD text, the existing SDDs, or internal team knowledge alone; an open question must go back to the client before the review can conclude. |
| `Not Applicable / Superseded` | Document content is out of scope, withdrawn, describes a different system entirely (e.g. an ops runbook for a third-party platform), or is superseded by another document — record why. |

**Tie-breaking when multiple findings apply to the same document:** `Boundary Violation`
and `Discrepancy — Needs Decision` outrank `Gaps Identified`, which outranks the
`Aligned — Minor *` variants, which outrank plain `Aligned`. In other words, pick the
status representing the thing most likely to require someone's attention next. The
review file's individual sections still record every finding regardless of which one
"wins" as that document's overall status.

**A discrepancy can be internal, not just FDD-vs-SDD.** Two shapes show up in practice
and both get filed under the Discrepancies section (with the review's Overall Status set
accordingly for the affected document), but should be **labeled explicitly** so a reader
doesn't mistake one for the other:
- *FDD vs. SDD* — the ordinary case this process is built for.
- *SDD vs. itself* — e.g. one domain doc's permissions catalog says a capability exists,
  its technical-design doc says the capability was deliberately cut. This blocks scoring
  the FDD content against that capability until the SDD is internally reconciled — note
  that explicitly rather than picking one SDD doc arbitrarily and comparing against it.
- *FDD vs. itself* — client source documents assembled over time (a main narrative plus
  older working spreadsheets, etc.) can disagree with each other. This isn't an SDD action
  item; record it as an Open Question to the client, and don't "correct" the SDD to match
  whichever internally-inconsistent FDD reading seems more authoritative.

## Tagging convention

Tag each section/requirement you map to the SDD side using the source document's own
path (or its client-numbering shorthand, e.g. `15.4`, when unambiguous within a review):

```
<source-doc-path-or-number> §<section-or-heading-reference> → <domain>[/<aggregate-or-permission-or-sequence>]
```

Examples:
```
15.4 - Person Search.docx §3.2 (Match Confidence Thresholds) → iam / PersonSearch (aggregate)
06 - Alerts, Emails, Communications §4.1 (Notify on Expiration) → communications, credentialing (cross-domain)
01 - User Management §Feature 1.5 (Roles admin) → iam / iam-permissions.md#approve-scopes
```

Record every tag in the review file's Tagging section, and mirror it into
[tag-index.md](tag-index.md) so the full set is queryable in one place without opening
every review file. See `tag-index.md` for the exact table format.
