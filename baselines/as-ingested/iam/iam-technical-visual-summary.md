# IAM Authorization Model - Visual Summary

## 1. Authorization Model Components

```mermaid
---
config:
  theme: neo-dark
---
graph TB
    subgraph "How Authorization Works"
        P[Permissions<br/>Atomic Actions<br/>staffing.employee.view]
        R[Roles<br/>Permission Bundles<br/>District HR Admin]
        S[Scopes<br/>Org Boundaries<br/>District-42]
        U[Users<br/>MiLogin Identities<br/>jane@example.com]
    end
    
    P -->|bundled into| R
    R -->|assigned at| S
    U -->|granted| R
    S -->|may have| H[Hierarchical<br/>Inheritance<br/>ISD → District → Building]
```

---

## 2. Organization Hierarchy & Transitive Permissions

```mermaid
---
config:
  theme: neo-dark
---
graph TD
    ISD50[ISD-50<br/>Intermediate School District]
    ISD50 --> D10[District-10]
    ISD50 --> B201[Building-201]
    
    D10 --> B101[Building-101]
    D10 --> B102[Building-102]
    
    EPP1[EPP-501<br/>Standalone]
    
    Note1[Authorization at ISD-50<br/>grants access to all<br/>Districts and Buildings]
```

---

## 3. Authorization Evaluation Flow

```mermaid
---
config:
  theme: neo-dark
---
flowchart TD
    Start([API Request]) --> Auth[Get User's<br/>Authorizations]
    Auth --> CheckScope{Permission<br/>Scope-Sensitive?}
    
    CheckScope -->|No| AnyAuth{Has Permission<br/>in ANY Role?}
    AnyAuth -->|Yes| Grant[✓ Grant Access]
    AnyAuth -->|No| Deny[✗ Deny Access]
    
    CheckScope -->|Yes| GetOrg[Get Organization<br/>from Request]
    GetOrg --> DirectMatch{Direct<br/>Match?}
    
    DirectMatch -->|Yes| CheckPerm[Role Has<br/>Permission?]
    DirectMatch -->|No| TransMatch{Parent Org<br/>Match?}
    
    TransMatch -->|Yes| CheckPerm
    TransMatch -->|No| Deny
    
    CheckPerm -->|Yes| Grant
    CheckPerm -->|No| Deny
```

---

## 4. Permission Design: Endpoint-Focused

```mermaid
---
config:
  theme: neo-dark
---
graph LR
    subgraph "UI Layer"
        U1[Employee Roster Screen]
        U2[Add Employee Button]
        U3[Edit Employee Form]
    end
    
    subgraph "Permission Layer"
        P1[staffing.employee.view]
        P2[staffing.employee.create]
        P3[staffing.employee.edit]
    end
    
    subgraph "API Layer"
        E1[GET /api/employees]
        E2[POST /api/employees]
        E3[PUT /api/employees/:id]
    end
    
    U1 -.requires.-> P1
    U2 -.requires.-> P2
    U3 -.requires.-> P3
    
    P1 --> E1
    P2 --> E2
    P3 --> E3
```

**Key Principle:** Permissions map to technical endpoints, not business processes. Backend checks permissions, not roles.

---

## 5. Scope Types

```mermaid
---
config:
  theme: neo-dark
---
graph TD
    subgraph "Scope-Sensitive Permissions"
        SS1[staffing.employee.view<br/>Requires: District-42]
        SS2[credential.application.approve<br/>Requires: ISD-50]
        SS3[iam.authorization.approve<br/>Requires: Building-101]
    end
    
    subgraph "Scope-Agnostic Permissions"
        SA1[iam.role.view<br/>System-Wide]
        SA2[system.report.export<br/>System-Wide]
        SA3[iam.audit.view<br/>System-Wide]
    end
```

---

## 6. Impersonation: Read-Only Mode

```mermaid
---
config:
  theme: neo-dark
---
graph TB
    subgraph "Admin Impersonating User"
        IMP[System Admin<br/>Viewing as Jane]
    end
    
    subgraph "Allowed<br/>(Read-Only)"
        A1[*.view]
        A2[*.search]
        A3[*.export]
    end
    
    subgraph "BLOCKED<br/>(Mutations)"
        B1[*.create]
        B2[*.edit]
        B3[*.delete]
        B4[*.approve]
        B5[*.revoke]
    end
    
    IMP -->|✓| A1
    IMP -->|✓| A2
    IMP -->|✓| A3
    
    IMP -->|✗| B1
    IMP -->|✗| B2
    IMP -->|✗| B3
    IMP -->|✗| B4
    IMP -->|✗| B5
```

**Rule:** ALL mutations blocked during impersonation, regardless of target user's permissions.

---

## 7. Three-Layer Cache Architecture

```mermaid
---
config:
  theme: neo-dark
---
graph TB
    Request[API Request] --> L1
    
    subgraph "Layer 1: Results Cache"
        L1[auth:userId:permission:org<br/>→ boolean<br/>TTL: 5 min]
    end
    
    L1 -->|Miss| L2
    
    subgraph "Layer 2: User Auth Cache"
        L2["user-auth:userId:miLoginType<br/>→ Authorization[]<br/>TTL: 5 min"]
    end
    
    L2 -->|Miss| DB[(Database)]
    
    L1 -->|Needs Hierarchy| L3
    
    subgraph "Layer 3: Org Hierarchy Cache"
        L3["hierarchy:orgCode<br/>→ Parent Codes[]<br/>TTL: 24 hours"]
    end
    
    L3 -->|Miss| EEM[EEM API]
```

**Performance Target:** < 5ms for cache hits, < 50ms for cache misses

---

## 8. Transitive Permission Example

```mermaid
---
config:
  theme: neo-dark
---
graph TD
    subgraph "Scenario"
        S1[User: Jane Smith<br/>Role: Staffing Admin<br/>Scope: ISD-50]
        S2[Request: View Employees<br/>Target: Building-101<br/>Permission: staffing.employee.view]
    end
    
    subgraph "Evaluation"
        E1[Building-101 hierarchy:<br/>Building-101 → District-10 → ISD-50]
        E2[Jane has Staffing Admin at ISD-50 ✓]
        E3[ISD-50 in hierarchy ✓]
        E4[Role contains staffing.employee.view ✓]
        E5[GRANT ACCESS ✓]
    end
    
    S1 --> E1
    S2 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> E4
    E4 --> E5
```

---

## Quick Reference

### Authorization Decision Factors
1. **User's Roles** - What roles are assigned?
2. **Role Permissions** - What can those roles do?
3. **Scope Assignment** - Where was role granted?
4. **Organization Hierarchy** - Is target org in scope?
5. **Request Context** - What org is being accessed?

### Key Rules
- **Transitive:** ISD authorization → all Districts + Buildings
- **Endpoint-Focused:** Check permissions, not roles
- **Impersonation:** Read-only, mutations always blocked
- **Cache-First:** Sub-5ms performance for common checks
- **Three Segments:** `domain.resource.action` max