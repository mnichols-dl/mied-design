# Credentialing Survey: Baseline Docs vs Graph

Read-only survey, run 2026-10-04, before any exporter code. It lines up the five credentialing baseline docs (`original-solutioning/credentialing/`) against what the graph holds for credentialing (`solutions/MiEdWorkforce/solution/MiEdWorkforce.ttl`, nodes under `urn:solutiondesign:instance:credentialing:`).

This is a mapping, not a verdict. Where it says "no home" it means no class or property holds that content today; whether to add one, put it in a template, or leave it out of the export is a separate decision.

"Verified" means checked by query or count. "Unverified" means inferred from structure and still to confirm.

## What the graph holds for credentialing

| Node type | Count | Notes |
|---|---|---|
| Requirement | 141 | FDD review data, not in the docs |
| SourceDocument | 75 | Mostly FDD files and wireframes; 5 are the domain doc homes |
| ValueObject | 39 | |
| ApiOperation | 36 | 34 are in the OpenAPI spec, 2 are placeholders for confirmed gaps |
| Permission | 30 | |
| StateDefinition | 29 | |
| DomainEvent | 22 | |
| Sequence | 18 | Plus 2 BatchJob |
| Annotation | 18 | |
| PermissionCategory | 15 | |
| BusinessRule | 14 | |
| UserStory | 10 | FDD review data |
| Aggregate | 7 | |
| DomainEntity | 7 | |
| UbiquitousTerm | 6 | |
| TechnicalProcess | 2 | |
| ApiSpec / BoundedContext | 1 each | |

Each doc-backed node carries `sd:definedIn` pointing at a `SourceDocument` node for its home doc (`DomainDoc`, `SequencesDoc`, `ApiDoc`, `PermissionsDoc`, `TechnicalDesignDoc`). Other nodes point at FDD source documents. That gives a starting boundary for what to export, though it is not a clean one (see "Graph ahead of the docs" below).

## ID-level match (verified)

| Item | In doc | In graph | Matching |
|---|---|---|---|
| Aggregates | 7 | 7 | 7 |
| Sequences (by title) | 18 | 18 | 18 |
| API operations (by operationId) | 34 | 34 | 34 |
| Business rules | 14 | 14 | 14 (by section vs node count) |
| Permissions (by ID) | 31 | 30 | 30 |
| Domain events | 20 | 22 | 20 |
| Ubiquitous terms | 25 | 6 | 6 |
| Open questions | 20 | 3 attached to the domain doc | to check |

The structured skeleton is well covered. The gaps are in prose and detail.

Differences worth knowing:
- Permission only in the doc: `credentialing.reports.view`. The graph has a Reporting-owned `Perm_CredentialingReportsView`, so it was probably relocated deliberately. Unverified.
- Events only in the graph: `CredentialExpired` (from the technical design batch job) and `AssessmentResultImported`.
- 19 ubiquitous terms in the doc have no node, among them Nullification, Revocation, Suspension, Grade Band, IDP, Print Name, Is Permanent, Manual Review Required.

## Mapping per doc

### credentialing-domain.md

| Section | Graph home | Status |
|---|---|---|
| Header: Type, Identifier, Display Name | `BoundedContext` `kind`, `identifier`, `displayName` | Covered |
| Header: Primary Sources (BRD refs) | None | No home. These point at the BRD, which is now archived |
| Purpose | `BoundedContext.purpose` | Covered |
| Scope: "This domain owns" list | None | No home |
| Scope: "This domain does NOT own" | `BoundedContext.notes`, one prose blob | Covered as text, not structured |
| Ubiquitous Language | `UbiquitousTerm` (`prefLabel`, `definition`, `describesConcept`) | 6 of 25 terms |
| Aggregates (7): Root, Purpose, Entities and Value Objects, Invariants, States | `Aggregate` (`rootEntity`, `description`, `hasEntity`, `hasValueObject`, `invariants`, `hasState`) | Covered, but order is lost (see below) |
| Aggregate "Referenced In" | Probably derived from `SequenceReference` | Unverified |
| Nested properties under a value object | `sd:fields` on 3 value objects | Partly covered |
| Entity Relationship Diagram | None (no `erDiagram` text anywhere in the graph) | No home |
| Domain Events | `DomainEvent` (`trigger`, `payloadHighlights`, `publishedToTopic`) | Covered, plus 2 extra |
| Dependencies, Upstream and Downstream | `Dependency` relationship nodes (22 touching credentialing) with `mechanism`, `viaOperation`, `viaEvent`, `notes` | Coarse. Grain differs (doc has one row per counterpart, graph one node per mechanism). Unverified |
| Business Rules (14) | `BusinessRule` (`statement`, `rationale`, `example`, `enforcedBy`, `appliesTo`, `notes`) | Covered. No title property, so a section heading would have to be derived from the IRI |
| Open Questions (20, numbered, with gaps in numbering and some struck through as resolved) | `Annotation` | Only 3 annotations attach to the domain doc, and numbering is not preserved. Needs a check |

### credentialing-sequences.md

| Section | Graph home | Status |
|---|---|---|
| Title and conventions block | Static boilerplate | Template, not graph |
| Per sequence: heading | `Sequence.title` | Covered |
| What / When / Who | `whatWhenWho`, one blob | Covered as text |
| Mermaid diagram | `content` (stored without the `title:` front matter, which is derivable) | Covered |
| Key Decisions | `keyDecisions` | Covered |
| State Changes | None | **No home** |
| Events Published | `eventsPublished` | Covered as text |
| Error Scenarios | `errorScenarios` | Covered as text |
| Order of the 18 sequences | None | Lost |

Any extra per-sequence sections beyond these were not checked.

### credentialing-permissions.md

| Section | Graph home | Status |
|---|---|---|
| Header: Domain, Version | Context identifier; version unknown | Likely template plus context |
| Table: Category | `Permission.hasCategory` to `PermissionCategory.label` | Covered |
| Table: Permission ID | `permissionId` | Covered |
| Table: Description | `description` | Covered |
| Table: Applicable Scopes | `hasScope` to shared scope nodes | Covered |
| Table: Notes | `notes` | Covered |
| Row order and category grouping | None | Lost |

This is the most mechanical doc. Verified that all 30 graph permissions have a row.

### credentialing-api.yml

Of the five, this has the widest gap between graph and doc. The doc is about 1,700 lines of OpenAPI detail. The graph's `ApiSpec` node holds only a version, and no property holds the spec text.

| OpenAPI element | Graph home | Status |
|---|---|---|
| `info` and `servers` | None | No home |
| `summary`, `operationId`, `tags` | `ApiOperation` properties | Covered |
| `path` and method | `path`, `method` | Covered |
| `x-access` | `accessType` | Covered |
| `x-permissions-required` | `PermissionRequirement` nodes (`requiresPermission`, `onOperation`, `conditional`, `conditionNote`) | Covered |
| `x-scope-sensitive` | `scopeSensitive` | Covered |
| Operation `description` | Only `notes` on 6 operations | Mostly no home |
| `parameters` | None | No home |
| `requestBody` | `consumesShape` on 11 operations, `requestBodySchema` on 2 | Coarse |
| `responses` (status codes, bodies) | `producesShape` and a `projection` text per operation | Coarse |
| `components`: 23 schemas, parameters, responses | Value objects and aggregates, with names and descriptions only | Mostly no home |

The ontology's own convention is to link request and response shapes to the domain model instead of restating them. That convention is why the yml cannot be rebuilt as it stands.

### credentialing-technical-design.md

| Section | Graph home | Status |
|---|---|---|
| Purpose | None found | Unverified |
| Configuration Parameters (overview, open design question, what is modeled) | None found | Probably no home. Unverified |
| Scheduled / Batch Processing: two jobs | `BatchJob` (`schedule`, `steps`, `whatWhenWho`, `errorScenarios`, `appliesTo`) | Covered |
| Certificate PDF Generation, and the cover letter note | `TechnicalProcess` (`steps`, `implementationNotes`, `notes`) | Covered |
| Electronic Signature / Attestation Capture | None found | Probably no home. Unverified |
| Open Technical Questions | 5 `Annotation` nodes on this doc | Likely covered |

Only 9 graph nodes (2 batch jobs, 2 technical processes, 5 annotations) are defined in this doc.

## Findings across all five

1. **IDs line up, detail does not.** Every identifier-bearing structured item matches. The gaps are in prose sections and in fine detail.
2. **Ordering is lost everywhere.** For example, an aggregate's states come back as Deprecated, Active, Draft in the graph and Draft, Active, Deprecated in the doc. Aggregate entities and value objects are one interleaved list in the doc but two separate properties in the graph. The order of sequences, permission rows, and sections has no home. This confirms `sd:order` is needed.
3. **Where the graph is ahead of the docs.**
   - Two extra domain events.
   - Two placeholder operations (`CredentialTypeOperation`, `LastIssuanceOperation`) with notes marking confirmed gaps.
   - Notes on some nodes that mention graph history ("converted in abox/mied-v3/...") rather than design content.
   - FDD-derived requirements, user stories, source documents, and annotations.
   The exporter needs to exclude or route these. The `definedIn` property helps, but some doc-relevant nodes are defined in FDD source documents.
4. **No title properties on some nodes.** Business rules and some other nodes have no human heading, so headings would be derived from IRIs, which is fragile.
5. **Several fields are single prose blobs holding list content** (`invariants`, `eventsPublished`, `errorScenarios`, `whatWhenWho`). Reproducing exact list formatting depends on how the text was authored, not on graph structure.
