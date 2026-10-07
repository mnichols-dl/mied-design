# Reporting - Workflows & Sequences

**Domain:** Reporting (Platform Capability)
**Version:** 1.0
**Last Updated:** TBD

This document contains sequence diagrams for all workflows in the Reporting platform capability.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human only
- **participant** = Internal service/component or external system (UI, services, Event Bus and external systems are all participants)
- Participants are grouped with `box`: Browser (UI), MiEdWorkforce (AKS) (services), External (external systems)
- Every request arrow starts with an API kind tag, then the verb and path from the API catalog: `APP` = Application API (UI to owning API, user delegated token), `SVC` = Service API (API to API, in-cluster mTLS), `EXT` = External API (inbound from an external system), `OUT` = outbound call to an external system. Responses carry no tag.
- Every application API call is authorized by the owning service through the cached IAM permission check (Service API). It is not drawn unless noted.

---

## Browse Report Catalog

**What:** A user views the list of reports available to them, filtered to only those they are authorized to access.  
**When:** User navigates to the Reports section of the MiEdWorkforce Portal.  
**Who:** Any authenticated portal user. Permission: reporting.catalog.view (system-wide; each report is further gated by its own permission key).

```mermaid
---
title: Reporting - Browse Report Catalog
---
sequenceDiagram
    actor User
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    participant IamApi as IAM API
    end

    User->>UI: Navigate to Reports
    UI->>ReportingApi: APP GET /reports
    ReportingApi->>IamApi: SVC POST /permissions/check (batch: all report permission keys)
    IamApi-->>ReportingApi: Permission results per report
    ReportingApi-->>UI: Filtered report catalog (only reports user is authorized for)
    UI-->>User: Report list grouped by domain
```

**Key Decisions:**
- **Catalog filtering at API layer:** The Reporting API filters the catalog server-side based on permission checks rather than returning the full catalog and relying on the client to hide unauthorized entries. This prevents information leakage about the existence of reports the user cannot access.
- **Batch permission check:** To avoid N+1 IAM calls (one per report), the Reporting API batches all permission key checks in a single IAM request.

**State Changes:** None — read-only operation.

**Events Published:** None.

**Error Scenarios:**
- IAM unavailable > Return 503; do not return partial catalog. Surface error to user.
- User holds no applicable report permissions > Return empty catalog; do not return 403 (empty state is a valid response, not an error).

---

## Generate Embed Token and View Report

**What:** A user selects a report and the system validates their authorization, resolves organizational context parameters, generates a Power BI embed token, and returns it to the portal for rendering.  
**When:** User selects a specific report from the catalog.  
**Who:** Any authenticated portal user who holds the report's declared permission. Permission: reporting.embed-token.generate (not directly assignable; decided by IAM from the report's declared permission key at the caller's organization scope).

### Happy Path

**Scenario:** User holds required permission at an appropriate scope; all required parameters can be resolved.

```mermaid
---
title: Reporting - Generate Embed Token and View Report - Happy Path
---
sequenceDiagram
    actor User
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    participant IamApi as IAM API
    participant OrgApi as Organizations API
    end
    box External
    participant PowerBi as Power BI
    end

    User->>UI: Select report
    UI->>ReportingApi: APP POST /reports/{reportDefinitionId}/embed-token

    ReportingApi->>IamApi: SVC POST /permissions/check (report's permissionKey, user, scope)
    IamApi-->>ReportingApi: Authorized: true, orgScope: { type: District, code: 12345 }

    ReportingApi->>OrgApi: SVC GET /organizations/{organizationCode}/hierarchy
    OrgApi-->>ReportingApi: Ancestor chain

    ReportingApi->>ReportingApi: Resolve embed parameters against ParameterContract
    Note over ReportingApi: UserUniqueId, OrgType, OrgCode, OrgAncestorCodes resolved

    ReportingApi->>PowerBi: OUT POST /GenerateToken (workspaceId, reportId, resolved params)
    PowerBi-->>ReportingApi: Embed token + expiry

    ReportingApi->>ReportingApi: Persist EmbedTokenRequest audit record (no token stored)
    Note over ReportingApi: outcome: Granted, resolvedParameters snapshot, expiresAt

    ReportingApi-->>UI: { embedToken, embedUrl, expiresAt }
    UI-->>User: Render embedded Power BI report
```

**Key Decisions:**
- **Parameter resolution before Power BI call:** All ParameterContract requirements are validated locally before calling the Power BI API. A failed resolution causes a 400/403 response without consuming a Power BI API call.
- **Org hierarchy for transitive scoping:** `OrgAncestorCodes` is resolved from the Organizations API to allow reports to support transitive org filtering (e.g., an ISD user sees data for all constituent districts).
- **Token not persisted:** Only audit metadata is stored; the token string itself is never written to the database.

**State Changes:**
- EmbedTokenRequest created with `outcome: Granted`

**Events Published:** None.

**Error Scenarios:**
- Power BI API timeout or error > Return 502 to portal; log audit record with failure reason; surface user-friendly error ("Report temporarily unavailable").

---

### Access Denied — Permission Check Fails

**Scenario:** User does not hold the report's declared permission, or the report is not in the catalog.

```mermaid
---
title: Reporting - Generate Embed Token and View Report - Access Denied
---
sequenceDiagram
    actor User
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    participant IamApi as IAM API
    end

    User->>UI: Select report (or direct URL access)
    UI->>ReportingApi: APP POST /reports/{reportDefinitionId}/embed-token

    ReportingApi->>IamApi: SVC POST /permissions/check (report's permissionKey, user, scope)
    IamApi-->>ReportingApi: Authorized: false

    ReportingApi->>ReportingApi: Persist EmbedTokenRequest audit record
    Note over ReportingApi: outcome: Denied, denialReason: InsufficientPermission

    ReportingApi-->>UI: 403 Forbidden
    UI-->>User: Access denied message
```

**State Changes:**
- EmbedTokenRequest created with `outcome: Denied`

---

### Access Denied — Required Parameter Cannot Be Resolved

**Scenario:** User holds the permission but cannot be scoped to the required organizational context (e.g., a system-level user requesting a district-scoped report without a district in context).

```mermaid
---
title: Reporting - Generate Embed Token and View Report - Parameter Resolution Failure
---
sequenceDiagram
    actor User
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    participant IamApi as IAM API
    end

    User->>UI: Select report
    UI->>ReportingApi: APP POST /reports/{reportDefinitionId}/embed-token

    ReportingApi->>IamApi: SVC POST /permissions/check (permissionKey, user, scope)
    IamApi-->>ReportingApi: Authorized: true, orgScope: { type: System }

    ReportingApi->>ReportingApi: Attempt to resolve ParameterContract
    Note over ReportingApi: OrgCode required; cannot resolve from System-level scope

    ReportingApi->>ReportingApi: Persist EmbedTokenRequest audit record
    Note over ReportingApi: outcome: Denied, denialReason: ParameterResolutionFailure

    ReportingApi-->>UI: 400 Bad Request (required parameter unresolvable)
    UI-->>User: "This report requires an organizational context. Select a district or ISD to continue."
```

**Recovery:** The portal should prompt the user to select an organizational context (e.g., via an org picker) and retry the token request with the selected scope included in the request body.

---

## Register Report in Catalog

**What:** An administrator registers a new Power BI report in the application catalog, making it available for embedding once activated.  
**When:** A new report has been published to the Power BI Service and is ready to be surfaced in the portal.  
**Who:** System Admin with `reporting.catalog.manage` permission. Permission: reporting.catalog.manage (system-wide).

```mermaid
---
title: Reporting - Register Report in Catalog
---
sequenceDiagram
    actor Admin
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    end
    box External
    participant PowerBi as Power BI
    end

    Admin->>UI: Open Report Catalog admin screen
    Admin->>UI: Fill in report metadata (name, domain, permissionKey, workspaceId, reportId, parameterContract)
    UI->>ReportingApi: APP POST /admin/reports

    ReportingApi->>PowerBi: OUT GET /reports/{reportId} (validate report exists in workspace)

    ReportingApi->>ReportingApi: Validate ParameterContract (all parameter names are standard)
    ReportingApi->>ReportingApi: Create ReportDefinition with status: Inactive

    ReportingApi-->>UI: { reportDefinitionId, status: Inactive }
    UI-->>Admin: "Report registered. Set status to Active to make it visible in the portal."

    Note over Admin,UI: Admin reviews, then activates separately
    Admin->>UI: Activate report
    UI->>ReportingApi: APP PATCH /admin/reports/{reportDefinitionId}/status (Active)
    ReportingApi->>ReportingApi: Update status: Inactive > Active
    UI-->>Admin: Report is now live in the catalog
```

**Key Decisions:**
- **Register then activate pattern:** New reports default to `Inactive` so admins can verify catalog metadata before making the report visible to users. This decouples registration from publication.
- **Power BI validation at registration:** Confirming the report exists in the workspace at registration time (not at token generation time) surfaces configuration errors early.

**State Changes:**
- ReportDefinition created with `status: Inactive`
- ReportDefinition status: `Inactive` > `Active` on explicit activation

**Events Published:** None.

**Error Scenarios:**
- Power BI workspace/report ID not found > Return 422 with message "Power BI report not found in specified workspace. Verify workspaceId and reportId."
- Non-standard parameter name in contract > Return 400; list disallowed parameter names.

---

## Deactivate Report

**What:** An admin removes a report from portal visibility. The Power BI report definition is unaffected.  
**When:** A report is retired, replaced, or temporarily pulled from the portal.  
**Who:** System Admin with `reporting.catalog.manage` permission. Permission: reporting.catalog.manage (system-wide).

```mermaid
---
title: Reporting - Deactivate Report
---
sequenceDiagram
    actor Admin
    box Browser
    participant UI as UI
    end
    box MiEdWorkforce (AKS)
    participant ReportingApi as Reporting API
    end

    Admin->>UI: Select report in catalog admin
    Admin->>UI: Deactivate report
    UI->>ReportingApi: APP PATCH /admin/reports/{reportDefinitionId}/status (Inactive)

    ReportingApi->>ReportingApi: Update status: Active > Inactive
    Note over ReportingApi: Existing embed tokens issued before deactivation remain valid until natural expiry

    UI-->>Admin: "Report deactivated. No longer visible in portal."
```

**Key Decisions:**
- **Deactivation does not invalidate outstanding tokens:** Tokens already in flight (issued but not yet expired) remain valid since the Power BI Service is not notified. This is acceptable given the short token expiry window. Revoking active tokens would require a Power BI API call per active session — disproportionate cost for a non-emergency deactivation.

**State Changes:**
- ReportDefinition status: `Active` > `Inactive`

**Events Published:** None.

**Error Scenarios:**
- Report already inactive > Return 409 Conflict with message.