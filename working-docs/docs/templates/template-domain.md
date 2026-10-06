# [Domain Name]

- **Type:** [Core Domain | Supporting Domain | Platform Capability]  
- **Identifier:** [briefandlowercase]  
- **Primary Sources:** [BRD numbers or input docs]

---

## Purpose

[2-3 sentences: What business problem does this solve? What value does it enable?]

---

## Classification Rationale

[ONLY if NOT a Core Domain - explain why this is supporting/capability and what depends on it]

---

## Scope

**This domain owns:**
- [Core responsibility 1]
- [Core responsibility 2]
- [Core responsibility 3]

**This domain does NOT own:**
- [Common confusion] > owned by `[other-domain]`
- [Another exclusion] > owned by `[other-domain]`

---

## Ubiquitous Language

| Term | Definition |
|------|------------|
| **[Term]** | [How this domain understands this concept - business meaning, not implementation] |
| **[Term]** | [Definition] |

---

## Domain Model

### Core Aggregates

#### [AggregateName]

**Root Entity:** [EntityName]

**Purpose:** [What business invariant does this aggregate protect?]

**Entities & Value Objects:**
- **[Entity]** - [Business purpose]
- **[ValueObject]** - [Business purpose]

**Key Invariants:**
- [Business rule that MUST always be true]
- [Another invariant]

**Key States:** [InitialState], [State2], [FinalState]

**Referenced In:**
- Sequence: [Flow Name]
- Sequence: [Another Flow]

---

[Repeat for each aggregate]

---

### Entity Relationship Diagram

<!--
ERD AUTHORING RULES (remove this comment block before publishing):

NAMING:
- Table names: SCREAMING_SNAKE_CASE, pluralized for aggregate roots
- Column names: snake_case
- PK: `uuid id PK "description"`
- FK: `uuid parent_id FK "description"`
- UK: `string code UK "description"`

DATA TYPES: uuid, string, int, decimal, boolean, date, datetime, json, binary

RELATIONSHIPS:
- ||--o{ = one-to-many
- ||--|| = one-to-one
- }o--o{ = many-to-many (requires junction table)
- Always include a relationship label

DOMAIN EVENT TABLES:
Every event-sourced aggregate gets an events table with:
  uuid event_id PK
  uuid aggregate_id FK "Reference to parent aggregate"
  string event_type "EventName1|EventName2|..."
  datetime event_timestamp
  int event_version "Optimistic concurrency"
  json event_payload
  uuid caused_by_user_id FK
  uuid correlation_id "Distributed tracing"
  binary event_hash "SHA256 for tamper detection"
  binary previous_event_hash "Hash chain"

VERSIONED DEFINITIONS:
Aggregates that version (e.g., definitions) need:
  date effective_from
  date effective_to "NULL for current version"
  uuid superseded_by_version_id FK "Links to newer version"

STATE MACHINE COLUMNS:
Document all enum values inline: `string status "Draft|Active|Deprecated"`

ALIGNMENT CHECKLIST:
- Every aggregate root → main table
- Every child entity → table with FK to parent
- Every value object → embedded columns or separate table
- Every domain event → entries in the events table
- Every versioned aggregate → version history columns or table
-->

```mermaid
---
title: [Domain Name] ERD
---
erDiagram
    PARENT_TABLE ||--o{ CHILD_TABLE : "relationship"
    PARENT_TABLE ||--o{ PARENT_EVENTS : "generates"

    PARENT_TABLE {
        uuid id PK "Primary key"
        string status "State1|State2|State3"
        datetime created_at
    }

    CHILD_TABLE {
        uuid id PK
        uuid parent_id FK "Reference to parent"
        string field_name "Description"
    }

    PARENT_EVENTS {
        uuid event_id PK
        uuid aggregate_id FK "id from PARENT_TABLE"
        string event_type "EventName1|EventName2"
        datetime event_timestamp
        int event_version "Optimistic concurrency"
        json event_payload
        uuid caused_by_user_id FK
        uuid correlation_id "Distributed tracing"
        binary event_hash "SHA256 for tamper detection"
        binary previous_event_hash "Hash chain"
    }
```

---

## Domain Events

Events published by this domain that other domains may subscribe to:

| Event | Aggregate | Trigger | Payload Highlights | Consumers |
|-------|-----------|---------|-------------------|-----------|
| `EventName` | [Aggregate] | [Business condition] | `{ field, field, field }` | [domain], [domain] |
| `AnotherEvent` | [Aggregate] | [Condition] | `{ field, field }` | [domain] |

**Event Naming Convention:** [PastTense][Noun][Action] (e.g., `AuthorizationApproved`, `UserAccountDeactivated`)

**Published To:** [Event bus/queue technology - e.g., "Azure Service Bus topic: domain-events"]

---

## Dependencies

### Upstream (We Consume From)

| Source | What We Need | How We Get It | Notes |
|--------|--------------|---------------|-------|
| **[external-system]** | [Data/capability] | [REST API / Event / Batch] | [Any quirks or constraints] |
| **[other-domain]** | [Data/capability] | [Pattern] | [Notes] |

### Downstream (Others Consume From Us)

| Consumer | What They Need | How They Get It | Notes |
|----------|----------------|-----------------|-------|
| **[other-domain]** | [Our data/capability] | [Event subscription / API call] | [Notes] |
| **[another-domain]** | [Data/capability] | [Pattern] | [Notes] |

---

## Business Rules

[ONLY include rules that are unique to this domain and impact design decisions]

### [Rule Category]

**Rule:** [Statement of the rule]

**Rationale:** [Why this rule exists - business/compliance/technical reason]

**Enforced By:** [Which aggregate/service enforces this]

**Example:** [Concrete example of rule in action]

---

[If no unique business rules beyond what's captured in aggregate invariants, OMIT this section]

---

## Integration Patterns

[ONLY include if there are domain-specific integration quirks worth documenting]

### [External System Name]

**Purpose:** [Why we integrate]  
**Pattern:** [Sync API / Async Event / Batch]  
**Frequency:** [Real-time / Scheduled]  
**Authentication:** [How we auth]  
**Error Handling:** [Retry strategy / Circuit breaker / etc.]  
**Constraint:** [Any specific limitation - e.g., "Rate limited to 100 req/min"]

---

[If integrations are straightforward REST/Event patterns, OMIT this section]

---

## Technical Considerations

[ONLY document domain-unique technical constraints that impact design]

**Performance:**
- [SLA or constraint if it drives architectural decisions]

**Security:**
- [Domain-specific security rules - not generic "use HTTPS"]

**Compliance:**
- [Regulations that constrain this domain - e.g., "FERPA governs identity matching"]

**Data Retention:**
- [Domain-specific retention rules]

---

[If all standard enterprise practices apply, OMIT this section]

---

## Workflows

This domain implements the following business workflows:

1. **[Flow Name]** - [One sentence: what this enables] > See: [Sequences Doc](./[domain]-sequences.md#flow-name)
2. **[Another Flow]** - [Description] > See: [Sequences Doc](./[domain]-sequences.md#another-flow)

---

## Open Questions

| # | Question | Impact | Owner | Target Date |
|---|----------|--------|-------|-------------|
| 1 | [Specific question needing decision] | [H/M/L] | [Name] | [Date] |
| 2 | [Another question] | [H/M/L] | [Name] | [Date] |

---

## References

**Related Documents:**
- [Sequences & Workflows](./[domain]-sequences.md)
- [Permissions Catalog](./[domain]-permissions.md) *(if applicable)*
- [API Contracts](./[domain]-api.yml) *(if applicable)*
- [Technical Architecture](./[domain]-technical-arch.md) *(if applicable)*

**External References:**
- [BRD Document Name] - [Link or doc ID]
- [Another source document]