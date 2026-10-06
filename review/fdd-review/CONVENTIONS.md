# Phase 2 conventions — FDD review

Read [`../../CONVENTIONS.md`](../../CONVENTIONS.md) FIRST — the domain prefix registry
and naming rules from Phase 1 all still apply. This file covers what's specific to
Phase 2: ingesting a raw client FDD and reconciling it against the graph Phase 1 built.

## Goal

For every distinct requirement in an FDD (a numbered user story with acceptance
criteria — NOT the same content restated in a "high level scenarios" paraphrase table,
which most of these FDDs also have and which should be skipped as duplicative), model
it as an `sd:UserStory` (an `sd:Requirement` subclass — see the ontology's own comment
on it) rather than plain `sd:Requirement`, unless the item genuinely isn't in "As a
<role>, I want <goal>, so that <benefit>" narrative form (a bare NFR line item → model
as `sd:NonFunctionalRequirement` instead; some other FDD content that's neither → plain
`sd:Requirement` is still fine). `sd:statement` carries the narrative text,
`sd:acceptanceCriteria` the AC — same properties either way, only the `rdf:type`
changes. Files already written before `sd:UserStory` existed (05, 11-16, 17, 29, 31)
still say plain `sd:Requirement`; retype them to `sd:UserStory` opportunistically when
you're touching that file for another reason, but it's not worth a dedicated pass on
its own.

Then decide exactly one of:
1. **Satisfied** — a real node already in `solution/MiEdWorkforce.ttl` implements it.
   Link it with `sd:satisfiedBy`. For an `sd:UserStory` specifically, prefer linking to
   the `sd:Sequence` (or `sd:BatchJob`) that actually carries the story out end-to-end
   over a narrower node (a lone `sd:ApiOperation` or `sd:Aggregate`) when a relevant
   Sequence exists — `sd:satisfiedBy` is multi-valued, so link the Sequence AND
   whatever else materially implements it (a Permission gating it, a BusinessRule it
   embodies) rather than picking just one. This is the traceability path a product
   owner's expectation gets checked against later, so the Sequence link is the one
   that matters most for that purpose.
2. **Out of scope / rejected** — the FDD itself says this belongs elsewhere, or a review
   comment shows it was explicitly declined. An `sd:Annotation` with
   `sd:findingType "out-of-scope"` or `"rejected"`, `oa:hasTarget` the Requirement.
3. **A genuine gap** — nothing satisfies it and nothing rejects it. Leave
   `sd:satisfiedBy` unasserted and add a `coverage-gap` Annotation. This is the actual
   point of the exercise — don't let a Requirement end up in neither state 1, 2, nor 3
   (i.e. never leave a Requirement "unaccounted for" — every one gets an Annotation or a
   `satisfiedBy`, per the ontology's own comment on `sd:Requirement`).

## Source material per FDD folder

Every `.docx` already has a pre-generated `.md` sibling (plain-text extraction) —
read the `.md` first for the fast pass. But the `.docx` itself usually carries much
more, and it's often where the real signal is:
- **Review comments** (`word/comments.xml` inside the unzipped docx) — dated back-and-forth
  between product owners and the vendor team. Often contains explicit approve/reject
  decisions, corrections to the visible text, and cross-references to other documents
  (a Role Matrix, an "Emails and Remarks.docx", etc.) that resolve otherwise-open questions.
- **Embedded images** (`word/media/*.png`) — wireframes. Extract and actually LOOK at
  them (Read tool on the image) — don't assume an image-to-caption mapping from
  document order is correct; confirm by viewing. A first-pass heuristic that assumes
  captions precede their image was wrong for FDD 11/16 (captions there followed their
  image) — always verify a couple before trusting the mapping.
- **Tracked changes** (`w:ins`/`w:del` in `word/document.xml`) — not worth reconstructing
  as a full diff, but if a comment thread references a stale vs. corrected version of the
  text, note which one you're actually working from.

To inspect a docx: `unzip -q "file.docx" -d unpacked/` then read `unpacked/word/comments.xml`
and list `unpacked/word/media/`. See the credentialing FDD 11/16 review (this session's
prior work, referenced in git history / conversation) for the exact extraction scripts
used — the same node one-liners work here (regex-split on `<w:comment ` for comments;
`_rels/document.xml.rels` + blip `r:embed` order for image-to-relationship-ID mapping).

Prioritize `.docx`. Skip `.vsdx`/`.vsd`/`.vsdm`/`.xmind`/`.pptx` (per the project owner's
instruction) unless a comment or the FDD text directly names one as load-bearing
reference data (e.g. a Role Matrix `.xlsx` — that's the one confirmed exception; treat
similarly-load-bearing `.xlsx`/`.xls` workbooks named by title in FDD text the same way).

## Depth calibration

Don't hold every FDD to the exact same depth uniformly — calibrate to size and apparent
importance (a 1-page pointer doc vs. an 8-story core feature FDD are not the same amount
of work). It's fine to:
- Model every numbered user story as its own `sd:Requirement`, but skip the redundant
  "high level scenarios"/paraphrase table most of these FDDs also carry.
- Dedupe near-identical repeated review comments into one Annotation rather than
  transcribing each (a recurring "define role" thread, for example) — the repetition
  itself is the finding, worth stating once with a note on how many times it recurred.
- Not view every embedded image if there are many near-duplicates (e.g. 18 similar
  screenshots under one caption) — view enough to confirm the general shape, note how
  many weren't individually reviewed, and move on.
State explicitly, in the file's header comment, what was skipped and why (matches the
style already established in `credentialing-fdd-11-16.ttl`'s pilot header, and every
Phase 1 domain file's header).

## Cross-document FDD groups

Several numbered FDD folders are companions to another — read them together, not
independently:
- **FDD 16's own folder** is a one-page pointer to **FDD 11's folder** ("11,16 - Educator
  Credentialing.docx") — treat as ONE ingestion producing Requirements numbered per the
  FDD's own `11.x`/`16.x` scheme, not two separate files.
- Folders with several docx by design/sub-feature (e.g. FDD 17's eight permit-type docs,
  FDD 31's four sub-numbered docs spanning different domains) may need to be split
  across domains or kept together depending on what they actually cover — check which
  bounded context(s) each sub-document's content actually belongs to before assuming the
  whole folder maps to one domain.

## Output location

`review/fdd-review/<fdd-number>-<slug>/<fdd-number>-<slug>.ttl` — one file per FDD
ingestion (or per logical sub-document group, if a folder genuinely spans multiple
domains — use judgment, note the split in the header). Naming pattern for new IRIs:
`Req_<FddNumber>_<StoryNumber>` (e.g. `cred:Req_11_1_1`), `Question_<Topic>` for
Annotations (same as Phase 1). Wireframe images go in a `wireframes/` subfolder next to
the ttl file; SourceDocument nodes for them use `sd:docType "wireframe"`.

## Things NOT yet in the solution design at all

If a whole FDD's subject area doesn't correspond to ANY existing bounded context (Phase
1 only converted 11 domains — several FDD folders cover areas like User Management/
MiLogin (01), Reporting/Dashboards (02-03), Business Rule Management (04), Interface &
Integrations Setup (08), Admin Users (12), Data Collection & Compliance
(10/18/19/20/21/22/24), Rapbacks (30), Tech Standards (32) — some of these may map to
domains that DO exist (iam, reporting, organizations) and some may not map to anything
converted yet at all) — don't force a `Requirement` node with no possible
`satisfiedBy`/rejection target. Instead, add an entry to the top-level
[`../GAPS.md`](../GAPS.md) log (one Markdown bullet list, grouped by FDD number, of
"this whole area has no corresponding solution-design artifact yet") rather than
forcing graph nodes for something the graph has no vocabulary position for at all.

## Validation

Same oxigraph approach as Phase 1 (see `../../CONVENTIONS.md`'s snippet) — load the
ontology, the shapes, `solution/MiEdWorkforce.ttl`, and every `fdd-review/*.ttl` file,
and confirm it parses and every `sd:satisfiedBy`/`oa:hasTarget` target resolves to
something real and typed.
Also run the "unaccounted Requirements" check (every `sd:Requirement` must have either
an `sd:satisfiedBy` or an Annotation `oa:hasTarget`-ing it) — this query is in the
credentialing FDD pilot's own validation output, reproduced here:

```sparql
PREFIX sd: <urn:solutiondesign:ontology:>
PREFIX oa: <http://www.w3.org/ns/oa#>
SELECT ?id WHERE {
  ?r a sd:Requirement ; sd:requirementId ?id .
  FILTER NOT EXISTS { ?r sd:satisfiedBy ?x }
  FILTER NOT EXISTS { ?ann oa:hasTarget ?r }
}
```
Should return empty for every Requirement you write.
