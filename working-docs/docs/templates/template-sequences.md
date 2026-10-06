# [Domain Name] - Workflows & Sequences

**Domain:** [Full Name]  
**Version:** [X.Y]  
**Last Updated:** [YYYY-MM-DD]

This document contains sequence diagrams for all workflows in the [Domain Name] domain.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

<!--
SEQUENCES AUTHORING RULES (remove this comment block before publishing):

DIAGRAM TITLES: Always use the pattern "[Domain] - [Flow Name]"
  For variants: "[Domain] - [Flow Name] - [Variant]" (e.g., "Happy Path", "Error Case")

STATE CHANGES IN DIAGRAMS: Always annotate with:
  Note over ServiceName: State: OldState > NewState

EVENT PUBLISHING: Always use fire-and-forget (--)) for async events:
  ServiceName--)EventBus: EventName

ACTOR VS PARTICIPANT:
  actor = human user, external system, or external organization
  participant = internal service or component within this system

PARTICIPANT NAMING: Use consistent short aliases with descriptive labels:
  participant AppService as Application Service
  participant EventBus as Event Bus
  participant IAM as Identity & Access API

ALT/ELSE BLOCKS: Use for meaningful branch points (PPR status, payment state, etc.)
  Do not use for trivial variations

NOTES: Use sparingly; only for context not obvious from the diagram itself

COVERAGE REQUIREMENT:
  Every workflow listed in the domain doc's Workflows section must have a sequence here.
  Every domain event listed in the domain doc must appear as a publish in at least one sequence.
  Every permission in the permissions catalog should be traceable to at least one sequence.

METADATA SECTIONS:
  Each sequence must include:
  - Key Decisions: branch points and what drives each path
  - State Changes: explicit before/after states for each aggregate affected
  - Events Published: each event and when it fires
  - Error Scenarios: at least the most likely failure conditions

SPLIT COMPLEX FLOWS:
  If a flow has more than ~4 major branches, split into Happy Path + separate variant diagrams
  rather than nesting deeply in a single diagram.
-->

---

## [Flow Name]

**What:** [One sentence: what this workflow accomplishes]  
**When:** [Business trigger that starts this flow]  
**Who:** [Primary actor(s) - reference permission required if relevant]

```mermaid
---
title: [Domain] - [Flow Name]
---
sequenceDiagram
    actor User
    participant UI as [Domain] UI
    participant AppService as Application Service
    participant ExternalAPI as External API
    participant EventBus as Event Bus

    User->>UI: [Initiating action]
    UI->>AppService: [Request]
    AppService->>ExternalAPI: [Downstream call]
    ExternalAPI-->>AppService: [Response]

    alt [Branch condition - e.g., validation passes]
        AppService->>AppService: [Create/update aggregate]
        Note over AppService: State: OldState > NewState
        AppService--)EventBus: EventName
        AppService-->>UI: [Success response]
        UI-->>User: [Confirmation]
    else [Alternate condition]
        AppService-->>UI: [Error/alternate response]
        UI-->>User: [Message to user]
    end
```

**Key Decisions:**
- **[Branch point]:** [What determines the path taken and why it matters]
- **[Another decision]:** [Explanation]

**State Changes:**
- [Aggregate] status: `OldState` > `NewState`

**Events Published:**
- `EventName` - [When this fires and why downstream consumers care]

**Error Scenarios:**
- [Error condition] > [What happens - state, user message, recovery path]

---

## [Another Flow]

**What:** [Description]  
**When:** [Trigger]  
**Who:** [Actor]

```mermaid
---
title: [Domain] - [Another Flow]
---
sequenceDiagram
    [diagram]
```

**Key Decisions:**
- [Decision point and reasoning]

**State Changes:**
- [Entity]: `OldState` > `NewState`

**Events Published:**
- `EventName` - [Description]

**Error Scenarios:**
- [Error] > [Outcome]

---

<!--
COMPLEX FLOW PATTERN:
When a flow has significant variation (happy path vs. error/edge cases),
split into multiple diagrams under the same logical flow heading.
-->

## [Complex Flow Name]

**What:** [Description]  
**When:** [Trigger]  
**Who:** [Actor]

### Happy Path

**Scenario:** [What conditions make this the happy path]

```mermaid
---
title: [Domain] - [Complex Flow Name] - Happy Path
---
sequenceDiagram
    [diagram]
```

**State Changes:**
- [Entity]: `OldState` > `NewState`

**Events Published:**
- `EventName` - [Description]

---

### [Variant Name - e.g., "PPR Hold" or "Manual Review Required"]

**Scenario:** [What conditions lead to this variant]

```mermaid
---
title: [Domain] - [Complex Flow Name] - [Variant Name]
---
sequenceDiagram
    [diagram]
```

**Recovery:** [How the system or user recovers from this path]

---

<!--
ADMIN/SYSTEM FLOW PATTERN:
For automated or system-initiated flows, omit the actor and use participants only.
Note the trigger clearly in the What/When fields.
-->

## [System-Initiated Flow Name]

**What:** [What the system does automatically]  
**When:** [Event or condition that triggers this - often an event subscription]  
**Who:** System process (triggered by `EventName` from [source domain])

```mermaid
---
title: [Domain] - [System-Initiated Flow Name]
---
sequenceDiagram
    participant EventBus as Event Bus
    participant Service as [Domain] Service
    participant OtherService as Other Service

    EventBus->>Service: TriggerEventName
    Service->>Service: Load aggregate
    [...]
    Service--)EventBus: ResultEventName
```

**Key Decisions:**
- [Decision and reasoning]

**State Changes:**
- [Entity]: `OldState` > `NewState`

**Events Published:**
- `ResultEventName` - [Description]

**Error Scenarios:**
- [Error] > [Outcome]

---

<!--
INTEGRATION FLOW PATTERN:
For flows centered on external system interaction, use this structure.
Document auth, timeout, and retry behavior inline or in the metadata below.
-->

## [External System] Integration

**Purpose:** [Why this integration exists and what business need it serves]  
**Trigger:** [What initiates the call - user action, event, schedule]

```mermaid
---
title: [Domain] - [External System] Integration
---
sequenceDiagram
    participant Service as [Domain] Service
    participant Gateway as API Gateway / Adapter
    participant External as [External System Name]

    Service->>Gateway: [Internal request]
    Gateway->>External: [Transformed/authenticated request]
    External-->>Gateway: [Response]
    Gateway-->>Service: [Normalized response]

    alt Success
        Service->>Service: [Process result]
        Note over Service: State: OldState > NewState
    else Failure / Timeout
        Service->>Service: [Error handling]
        Note over Service: [Fallback behavior or retry queued]
    end
```

**Auth:** [Authentication method - e.g., OAuth2 client credentials, API key via Key Vault]  
**Timeout:** [Request timeout and what happens on timeout]  
**Retry:** [Retry strategy - e.g., exponential backoff, max 3 attempts, dead-letter queue]  
**Error Handling:** [Circuit breaker, fallback, or manual intervention path]