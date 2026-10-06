# Reporting - Technical Design

## Purpose

This document provides technical design details for the Reporting platform capability. It covers the Power BI embed token generation algorithm, parameter resolution strategy, the standard parameter contract, security considerations for token issuance, and the client-side rendering lifecycle.

---

## Power BI Embed Token Generation

### Overview

Embedding a Power BI report in the portal requires a short-lived token issued by the Power BI REST API. The token is scoped to a specific report, optionally carries RLS parameters that filter data within the report, and expires after a configured window (target: 60 minutes). The Reporting API acts as the secure server-side broker: it validates authorization, resolves parameters, calls the Power BI API with a service principal credential, and returns only the token and embed URL to the client — never the service principal credential itself.

### Token Generation Algorithm

```mermaid
flowchart TD
    A([Client: POST /reports/{id}/embed-token]) --> B[Load ReportDefinition from catalog]
    B --> C{Report exists\nand Active?}
    C -->|No| ERR1[404 Not Found]
    C -->|Yes| D[Check user permission via IAM API\npermissionKey declared on report]
    D --> E{Authorized?}
    E -->|No| ERR2[403 Forbidden\nAudit: Denied/InsufficientPermission]
    E -->|Yes| F[Retrieve user's org scope\nfrom IAM authorization context]
    F --> G[Fetch org hierarchy\nfrom Organizations API if OrgCode in contract]
    G --> H[Resolve all ParameterContract fields\nagainst user + org context]
    H --> I{All required params\nresolved?}
    I -->|No| ERR3[400 Bad Request\nAudit: Denied/ParameterResolutionFailure]
    I -->|Yes| J[Call Power BI REST API\nGenerateToken with resolved params]
    J --> K{Power BI API\nsucceeded?}
    K -->|No| ERR4[502 Bad Gateway\nLog error, surface user-friendly message]
    K -->|Yes| L[Write EmbedTokenRequest audit record\nNO token stored]
    L --> M([Return embedToken + embedUrl + expiresAt])
```

### Power BI API Call

The `GenerateToken` request is made using the **service principal** credential stored in Azure Key Vault. The call is server-to-server; the client never receives or uses the service principal credential.

```
POST https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/reports/{reportId}/GenerateToken

Authorization: Bearer {service_principal_access_token}

{
  "accessLevel": "View",
  "identities": [
    {
      "username": "{UserUniqueId}",
      "roles": ["{RlsRoleName}"],
      "datasets": ["{datasetId}"]
    }
  ]
}
```

**Notes on RLS Identity Block:**
- `username` maps to `UserUniqueId` from the resolved parameter set
- `roles` maps to the RLS role name configured in the Power BI report definition — this is agreed upon in the ParameterContract and is a static string per report
- Additional filter parameters (OrgCode, OrgType, OrgAncestorCodes) are passed as effective identity filters, not as separate API fields — the exact mechanism depends on the report's dataset configuration (Import vs. DirectQuery vs. Live Connection) and is detailed per-report in the ParameterContract

**Licensing Dependency:** The `GenerateToken` endpoint behavior and available features (RLS, filter parameters) depend on the Power BI licensing SKU. See Open Question #1.

---

## Standard Parameter Contract

### Purpose

The parameter contract establishes a shared language between the application and the Power BI report author. When a report is registered in the catalog, its `ParameterContract` declares which standard parameters it uses and how they map to RLS or filter behavior inside the report definition. The application resolves these parameters from the user's session context before calling the Power BI API.

This avoids per-report custom resolution logic in the application. All resolution is driven by parameter names, which map to fixed resolution sources.

### Standard Parameters and Resolution Sources

| Parameter Name | Type | Resolution Source | Resolution Logic |
|----------------|------|-------------------|-----------------|
| `UserUniqueId` | String | UserContext | Mi-Key Unique ID from the authenticated user's identity token |
| `OrgType` | String | OrgContext | The organizational type of the user's scoped authorization (`District`, `ISD`, `System`) |
| `OrgCode` | String | OrgContext | The primary org code from the user's scoped authorization |
| `OrgAncestorCodes` | String (CSV) | OrgContext + Organizations API | Ancestor org codes from the materialized hierarchy; fetched from Organizations API using `OrgCode` |
| `UserRole` | String | UserContext | The user's highest-ranking role within the report's domain, derived from IAM authorizations |

### Resolution Algorithm Detail

```mermaid
flowchart TD
    A([ParameterContract for this report]) --> B[For each declared parameter]
    B --> C{Resolution source?}
    C -->|UserContext| D[Read from JWT claims\nor IAM user session]
    C -->|OrgContext| E{OrgCode already\nresolved?}
    E -->|Yes| F[Use cached OrgCode]
    E -->|No| G[Read primary org scope\nfrom IAM authorization response]
    G --> F
    F --> H{Parameter is\nOrgAncestorCodes?}
    H -->|Yes| I[Call Organizations API\nGET /organizations/{OrgCode}/hierarchy\nFlatten ancestor chain to CSV]
    H -->|No| J[Use OrgCode/OrgType directly]
    D --> K{Parameter\nis_required?}
    I --> K
    J --> K
    K -->|Yes and value is null| ERR[Fail: ParameterResolutionFailure]
    K -->|No or value resolved| L[Add to resolved parameter set]
    L --> M{More parameters?}
    M -->|Yes| B
    M -->|No| N([Return resolved parameter set])
```

### Example: District-Scoped Staffing Report

ParameterContract declaration:
```json
[
  { "parameter_name": "UserUniqueId",      "is_required": true,  "resolution_source": "UserContext" },
  { "parameter_name": "OrgType",           "is_required": true,  "resolution_source": "OrgContext"  },
  { "parameter_name": "OrgCode",           "is_required": true,  "resolution_source": "OrgContext"  },
  { "parameter_name": "OrgAncestorCodes",  "is_required": false, "resolution_source": "OrgContext"  }
]
```

For a district user (org scope: `District / 99123`), the resolved parameters would be:
```json
{
  "UserUniqueId":     "87654321",
  "OrgType":          "District",
  "OrgCode":          "99123",
  "OrgAncestorCodes": "50001,00000"
}
```

The RLS configuration in the Power BI report uses these values to filter the dataset to rows matching `OrgCode = '99123'` or ancestors where transitive filtering is needed.

### Adding New Parameters

If a report requires a parameter not in the standard set, a catalog governance review is required before registration. New parameters must have a defined, general-purpose resolution source so they can serve multiple reports — one-off parameters that require per-report custom resolution logic are not permitted.

---

## Client-Side Rendering and Token Refresh

### Embed Rendering

The portal renders embedded reports using the [Power BI JavaScript SDK](https://github.com/microsoft/PowerBI-client-react) (or equivalent). The embed flow is:

1. Portal calls `POST /reports/{reportId}/embed-token` on page load (or report selection)
2. Reporting API returns `{ embedToken, embedUrl, expiresAt }`
3. Portal initializes the Power BI embed component with the token and URL
4. Power BI renders the report in an iframe; RLS filtering is applied server-side by Power BI using the token's embedded identity

### Token Expiry and Refresh

Embed tokens have a finite lifespan (target: 60 minutes). To prevent the report going stale mid-session, the portal should implement proactive token refresh:

```
refreshThresholdMinutes = 10  // Refresh when ≤ 10 minutes remain

On page load:
  Schedule refresh at: expiresAt - refreshThresholdMinutes

On refresh:
  Call POST /reports/{reportId}/embed-token
  Apply new token to embedded report via PowerBI SDK updateSettings / setAccessToken
  Reschedule next refresh
```

This avoids a full page reload for token refresh. The Power BI JavaScript SDK supports applying a new token to an already-rendered report without re-rendering from scratch.

### Ad Hoc Exploration vs. Report Editing

Embedded reports support user-driven exploration: filtering, slicing, drilling through pages, and cross-highlighting visuals. This is facilitated by the embed token's `accessLevel: "View"` combined with report page navigation and filter pane visibility configured at the report definition level.

**Report definitions are not editable through the portal.** The `accessLevel` in the token is always `"View"`. Editing report definitions requires a Power BI Pro or Premium Per User license, Power BI Desktop, and access to the Power BI Service workspace — all out of scope for the portal.

---

## Security Considerations

### Service Principal Credential Management

The service principal used to call the Power BI API is stored in Azure Key Vault. The Reporting API accesses it via managed identity — the credential string is never in application configuration files, environment variables, or logs.

Rotation schedule and process: **TBD** (See Open Question #6).

### Token Scope Enforcement

The embed token is scoped by the Power BI API to a specific `workspaceId` and `reportId`. Even if a client attempts to use a token against a different report ID, the Power BI Service will reject it. This means a token cannot be reused or redirected.

### Preventing Parameter Spoofing

All parameters in the resolved parameter set are derived server-side from the user's authenticated session and IAM authorization context. The client request body for `POST /reports/{reportId}/embed-token` does **not** accept raw parameter values from the caller. The only client input is the report ID (from the URL path). This ensures:

- A district user cannot pass a different `OrgCode` to see another district's data
- A user cannot elevate their `UserRole` parameter beyond what IAM resolves

### Audit Log

Every token request (granted or denied) is written to `EMBED_TOKEN_REQUESTS`. This provides:
- A full record of who accessed which report and when
- Denial patterns that may indicate unauthorized access attempts
- Parameter snapshots for forensic review if a data access concern is raised

The token string itself is never written to the audit log.

---

## Dataset Configuration Considerations

Power BI supports multiple dataset connectivity modes. The choice affects how RLS and parameter filtering work at the Power BI layer. Report authors and the platform team should agree on dataset mode before registering a report.

| Mode | RLS Support | Notes |
|------|-------------|-------|
| Import | Full RLS via roles | Data refreshed on schedule; not real-time. Suitable for most reporting use cases. |
| DirectQuery | Full RLS via roles | Real-time data; query performance depends on source DB. Higher load on Azure SQL. |
| Live Connection (SSAS/AAS) | RLS via identity passthrough | Not typical for this architecture. |
| Composite | Partial; depends on table mode | Complex to configure; avoid unless necessary. |

For MiEdWorkforce, **Import mode** is the expected default given data freshness requirements for most reports. DirectQuery may be considered for reports requiring near-real-time data if performance testing supports it.

---

## Open Technical Questions

| # | Question | Impact | Owner | Target Date |
|---|----------|--------|-------|-------------|
| 1 | Power BI licensing SKU (Embedded A-SKU vs. Premium P-SKU vs. Per-User Pro): determines whether `GenerateToken` supports service-principal-based multi-user embedding without per-user licenses. A-SKU or P-SKU is required for anonymous/service-principal embedding at scale. | H | Platform / Procurement | TBD |
| 2 | Should `OrgAncestorCodes` be pre-computed and cached (reducing Organizations API calls per token request), or fetched fresh on each token generation? Cache TTL would need to align with the org sync pipeline cadence (nightly). | M | Platform | TBD |
| 3 | How does the Power BI JavaScript SDK handle token refresh for reports already rendered? Confirm `setAccessToken` API is available in the chosen SDK version before committing to the no-reload refresh approach. | M | Frontend | TBD |
| 4 | What is the datasetId discovery strategy? The `GenerateToken` call requires the dataset ID associated with the report. Should this be stored in the ParameterContract at registration time, or fetched dynamically from the Power BI API at token generation time? | M | Platform | TBD |
