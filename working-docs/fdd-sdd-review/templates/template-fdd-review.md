<!--
Copy this file to fdd-sdd-review/reviews/<topic-slug>.md (name it after the FDD
folder/topic, e.g. 01-user-management.md — no surrogate ID) and fill it in.
Delete instructional comments (like this one) as you complete each section.
Keep every table even if a section has no findings — write "None identified" as the
only row/line so it's clear the section was actually reviewed, not skipped.
-->

# Review: <FDD Topic Title> (<client folder number, e.g. "01">)

| Field | Value |
|---|---|
| Source Document(s) | `functional-design-docs/<file>` (list every file this review covers, one per line if more than one — each also needs its own row in `tracker.md` with its own status) |
| Related Domain(s) | <e.g. credentialing, iam> |
| Reviewer | <name> |
| Date Reviewed | <YYYY-MM-DD> |
| **Overall Status (per document)** | <one Overall Status value per Source Document above, from README.md's table — do not collapse to a single status if the documents warrant different ones; state each explicitly, e.g. "Main doc: Discrepancy — Needs Decision; IDD sub-docs: Boundary Violation; Helpdesk guide: Not Applicable"> |

## Summary

<!-- 2-4 sentences: what the source document(s) cover, and the headline conclusion(s) per
document if they differ. This is the part someone skimming the tracker should be able to
read to decide whether to open the full file. -->

---

## 1. Altitude / Boundary Check

Content that is more technical or prescriptive than a functional design doc should be —
implementation choices, schemas, algorithms, specific vendor/library selections, API
contracts, etc. This is about whether the document *itself* is written at the wrong
altitude, independent of whether the content is otherwise correct.

| # | Source Reference (doc §/heading) | What It Prescribes | Why It's Out of Bounds | Recommendation |
|---|---|---|---|---|
| 1 | | | | e.g. "Reframe as a business rule and let SDD choose the mechanism" / "Push back to client — this belongs in a technical design, not a functional spec" |

<!-- If none: single row "None identified." across the table, or just state it in prose. -->

## 2. Discrepancies

Places where two sources describe the same thing differently — contradicting rules,
mismatched states, different terminology for the same concept, different actors/
permissions for the same action, etc. Label which shape each row is, since the follow-up
action differs (see README's Overall Status note):

- **FDD vs. SDD** — the ordinary case.
- **SDD vs. itself** — two SDD documents (e.g. a permissions catalog and a
  technical-design doc) disagree with each other. Flag this explicitly; don't silently
  pick one side to compare the FDD against.
- **FDD vs. itself** — the client's own source documents disagree with each other. Not
  an SDD action item — record as an Open Question to the client instead.

| # | Shape | Topic | Side A Says (doc:section) | Side B Says (doc:section) | Assessment | Resolution / Decision |
|---|---|---|---|---|---|---|
| 1 | FDD vs. SDD / SDD vs. itself / FDD vs. itself | | | | Which is correct, or is it a genuine open question? | What was decided, or "Needs Client Clarification" |

<!-- If none: single row "None identified." -->

## 3. Coverage Gaps (In Source Document, Not in SDD)

Anything the document describes — a rule, workflow, entity, state, integration,
permission — that isn't represented in any `solution-areas/*` doc today. This is the
primary signal this whole process exists to produce.

| # | Source Reference (doc §/heading) | What's Missing | Likely Home in SDD | Priority (H/M/L) | Follow-up |
|---|---|---|---|---|---|
| 1 | | | e.g. `credentialing-domain.md` new aggregate, or `credentialing-sequences.md` new flow | | Link to an Open Question added, or note that it was drafted directly |

<!-- If none: single row "None identified." -->

## 4. Tagging

Every section/requirement mapped to a domain, aggregate, permission, or sequence,
regardless of the outcome above. Use the `<source-doc-path-or-number> §ref → domain/target`
format from the process README. Every row here should also be copied into
[../tag-index.md](../tag-index.md).

| Source Reference (doc §/heading) | Domain(s) | Aggregate / Permission / Sequence | Relationship |
|---|---|---|---|
| | | | e.g. "implements", "conflicts with", "gap — not yet modeled", "informs open question #N" |

---

## Open Questions Raised by This Review

<!-- Only questions that need a person (client or internal) to answer before the review
can be considered fully resolved. Distinct from Coverage Gaps, which are known work —
these are genuine unknowns. Include FDD-vs-itself inconsistencies here (see §2). -->

| # | Question | Raised To | Status |
|---|---|---|---|
| 1 | | Client / Internal | Open |

## Related Documents

- Source document(s): `functional-design-docs/<file>` (repeat for each — see header table)
- SDD documents reviewed against: <list the specific `-domain.md`, `-permissions.md`,
  `-sequences.md`, `-technical-design.md` files consulted>
