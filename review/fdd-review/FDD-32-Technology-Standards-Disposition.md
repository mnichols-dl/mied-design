# FDD 32 (Technology, Standards & Auditing)

Source: `32 - Technology standards and Audit`

**Scope note:** Editing, consolidating, or trimming content within FDD 32 - or moving a requirement to an existing FDD that already owns it (02, 05, 10, 13, 14, etc.) - is in scope and expected. What's explicitly *not* being proposed anywhere in this review is a new FDD number (e.g., an "FDD 33") to house split-out NFRs or features - the client's 1–32 numbering/scheduling stays as-is, and any consolidated content (NFRs included) gets a home within that existing structure rather than a new slot.

## Framing issue

FDD 32 isn't a functional design - it's a catch-all mixing NFRs, contract terms, and a few genuine cross-cutting features. Half the feature IDs (32.1, 32.3–32.10, 32.12) have no user stories by their own admission ("guideline rather than a functional deliverable"), 32.13 is explicitly FDD 02 restated, and most of 32.15 points back to other FDDs as its real home.

## Disposition legend

- **Extract as NFR** - not a feature; modeled in our cross-cutting non-functional requirement reference instead of as a capability requirement, no story format.
- **Drop** - recommend not carrying into our requirements/backlog model at all (contractual, SOW, generic sign-off checklist - out of scope for a functional/NFR spec).
- **Duplicated** - appears to be modeled or owned elsewhere in the FDDs.
- **Revise** - a real requirement, but the wording/scope we carry into our model needs to be tightened.

## Summary table

| ID | Title | Disposition | Notes |
|---|---|---|---|
| 32.1 | Application Standards | **Extract as NFR** | No functional content; purely guidelines |
| 32.2 | Device UI/UX | **Extract as NFR** | Mobile-responsiveness is an NFR; call out separately only if a mobile-specific (not just responsive) feature is identified |
| 32.3 | Auditing | **Extract as NFR** | Change-auditing and retention are cross-cutting; other features already assume they exist |
| 32.4 | Database | **Extract as NFR** | CEDS-conformance is the real requirement; vendor/schema detail doesn't belong here |
| 32.5–32.10 | Software (×6) | **Extract as NFR** | One paragraph of content repeated across 6 IDs; collapse to one |
| 32.11 | Data Migration and Conversion | **Revise** | .1 duplicates the 32.4 NFR; .2 mixes a real functional need with migration-plan content that may not belong in an FDD at all |
| 32.12 | Integration | **Drop** | Contractual exit-plan + generic security sign-off, not a system requirement |
| 32.13 | Reporting | **Duplicated** | Addendum confirms this is FDD 02's content restated |
| 32.14 | Batch Processing | **Revise** | Legitimate feature; confirm build-vs-cloud-native approach; check overlap with FDD 05 |
| 32.15 | Worklist | **Revise** | .3/.8/.9 are the canonical framework; .1/.2/.4–.7 duplicate their owning FDDs |
| 32.16 | Electronic Document Management | **Revise** | Genuine reusable infrastructure; .1 (malware scanning) should drop the story format |

## Detail by feature

### Extract as NFR

- **32.1** - no scenario content beyond a blank UAT placeholder row.
- **32.3 Auditing** - two concerns hiding in one deprecated feature:
  - *Change auditing* (actor/timestamp/before-after, no hard deletes, admin history view) is already assumed elsewhere - worklist (32.15.3, 32.15.9) and batch job history (32.14.1) both reference "audit controls" without redefining them. State it once as a standing NFR.
  - *Retention* is currently just "governed by SOM guidelines," yet real numbers already leak into other sections uncoordinated - 32.11.2 cites "issue date plus 99 years," 32.16.6 requires granular retention/legal holds per document type. Our NFR model should state actual schedules/categories once, sourced from all of these, rather than treat each as independent - this has real data-model and lifecycle-job implications for us regardless of how the client's document is organized.
- **32.4 Database** - the real requirement is "primary data store for educator/certification data must conform to CEDS." The rest (SQL Server today, possible Cosmos/JSON-LD, dbo schema handling) is undecided implementation speculation ("the state retains the right to determine") - we track that separately as an internal architecture decision, not as part of the NFR we carry forward.
- **32.5–32.10 Software** - six IDs, one paragraph, addendum repeats "user stories not required" six times differing only by parenthetical. We model this as a single "Technology Stack & Maintainability Standards" NFR internally, traced back to all six IDs.
- **32.2 Device UI/UX** - both .1 (standards adherence) and .2 (mobile accessibility "with particular focus for educators and non-certified staff") are NFR-shaped: neither describes a distinct feature, just an expected quality of every screen. Fold into the NFR set as "the application is mobile-responsive throughout." **Caveat:** if there turns out to be a genuinely mobile-specific feature - not just a responsive layout, but different behavior on mobile (e.g., camera-based document capture, an offline-capable submission flow, a deliberately reduced mobile-only task set for a specific role) - that piece should be pulled back out and written as its own functional requirement under the relevant feature area, not left inside the NFR. Nothing in the current text suggests one exists, but "particular focus for educators and non-certified staff" is close enough to that shape that it's worth a specific question to the client rather than assuming it's just emphasis.

### Drop (not carried into our backlog)

- **32.12 Integration** - a contractual exit-plan obligation plus a generic pre-go-live security checklist (DAST/SAST/SCA, no default passwords, smoke test). This is SOW/contract and security-signoff territory we track elsewhere - it never becomes a backlog item.

### Duplicated - confirm the owning location covers it, then model once

- **32.11.1** - restates the 32.4 CEDS NFR as a story with a different narrator. We model the CEDS NFR once and trace both IDs to it.
- **32.13 Reporting** - addendum says outright these stories implement FDD 02. Before treating 32.13 as fully covered by FDD 02's backlog items, confirm FDD 02 covers SAS integration (its notes so far only mention Power BI pass-through) and "all legacy reports must be available" - if there's a gap, that content still needs a home in our model even though it originates from 32.13.
- **32.15.1, .2, .4–.7** (credential, identity, EPP, professional learning, professional practice/rapback, staffing) - each cites the FDD that actually owns it (5.5, 13.3, 14.3, etc.). We model the requirement once under the owning FDD and trace 32.15's mention to it, rather than carrying a second copy.
- **32.16.1** (malware scanning) - check whether FDDs with their own upload flows (FDD 10 Staffing, EPP docs) restate this rather than pointing to it; if so, model it once under 32.16 and trace the others to it.

### Revise

- **32.11.2** (legacy data access) - this one blends two different things that should probably be pulled apart:
  - A genuine, durable **functional requirement**: a MiEdWorkforce user can reference historical/legacy information for their professional record through the application (regardless of where or how it's stored). That belongs in FDD 32/11 as a real requirement, but needs firmer wording - "an appropriate platform (e.g., cold storage)" isn't testable. State the actual mechanism once chosen, or at minimum a concrete acceptance bar (e.g., "retrievable within N business days").
  - **Migration-plan/execution content** that reads more like a cutover plan than a system requirement: "identified data must also be loaded, tested, and updated to the Solution as needed for production and lower environments throughout the project lifecycle," "wherever possible... replicate legacy data." This describes *how the migration project gets executed*, not an end-state system capability - it doesn't hold up as a requirement someone tests against the finished system, it holds up as a project plan someone executes once during migration. Recommend this piece move to (or already exist in) a Data Migration & Cutover Plan, separate from the FDD, and that 32.11.2 be trimmed down to just the end-state "user can access historical data" requirement.
- **32.14 Batch Processing** - stories (.1–.7) are legitimate, real actor and real choices; keep the structure but cut unfalsifiable filler ("reasonable customization... without new development"). Two open items:
  - *Build-vs-cloud-native:* the doc already hedges between a custom scheduler and Azure-native ("preferably incorporated... in case an external scheduler is used"; "if a native solution is implemented, e.g. azure logic apps"). As written it describes outcomes, not a UI mandate - cloud-native scheduling/monitoring behind a thin admin view is plausible. The catch: several acceptance criteria assume in-app job-level RBAC ("cannot see jobs for which they are not authorized"), which would need to map onto platform RBAC instead of an app permission table if scheduling lives natively in Azure - also changes where the audit trail lives (platform logs vs. app log), tying back to 32.3. **Confirm the approach with the state before committing.**
  - Check overlap with FDD 05's generic "batch jobs" mentions; settle one canonical owner.
- **32.15 Worklist** - .3/.8/.9 (privilege configuration+audit, dashboard access, role-based allocation) are the genuine framework; keep as the canonical feature, have the domain-specific duplicates reference it instead.
- **32.16 Electronic Document Management** - keep the majority (upload, versioning, retention/legal holds, bulk ops, permissioned viewing) as canonical shared infrastructure. Reword 32.16.1 out of story format into a plain security-control statement - no discretionary actor, same category as the 32.1/32.5–32.10 NFRs.

## Recommended next steps

All consolidation below lands either within FDD 32 itself or in an existing FDD that already owns the content - never a new FDD number.

### Consolidation / editing (within FDD 32 or existing FDDs)

1. Collapse 32.1, 32.2, 32.3, 32.4 (NFR portion), and 32.5–32.10 into a single NFR section within FDD 32 (or a standards appendix, still under the existing numbering) - no stories, no vendor/schema speculation, explicit retention schedules pulled from wherever they're currently scattered (32.11.2, 32.16.6).
2. Remove 32.12 from the functional/NFR content entirely; its material belongs in the SOW and a security sign-off checklist, not a new or existing FDD.
3. Trim 32.11.2 to the durable "user can access historical data" requirement; move the migration execution/loading/testing language to a Data Migration & Cutover Plan (confirm whether one already exists before creating it).
4. Within 32.15, keep the framework-level stories (.3, .8, .9) as the section's content; strike the domain-specific ones (.1, .2, .4–.7) and point to the FDDs that already own them (5.5, 13.3, 14.3, etc.).
5. Keep 32.16 as-is aside from rewording .1 out of story format; if other FDDs (10, 13, etc.) are found to restate upload/scan/versioning behavior, trim those and point back to 32.16 as the source.
6. Once FDD 02 is confirmed to cover SAS integration and legacy report parity, remove 32.13 from FDD 32 and rely on FDD 02 alone.

### Worth raising with the client (content ambiguity, not document structure)

7. 32.2 mobile-readiness: confirm there's no mobile-specific (behaviorally different, not just responsive) feature hiding behind "particular focus for educators and non-certified staff" before folding it fully into the NFR.
8. 32.14 batch processing: confirm build-vs-cloud-native approach before committing to an admin-screen design.
9. Whether they want "Epic 32" tracked as a single epic going forward, given how much of its content isn't a standalone feature - purely a question about their backlog/epic structure, not about FDD 32's content or numbering.
