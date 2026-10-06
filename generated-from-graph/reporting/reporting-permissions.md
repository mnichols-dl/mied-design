# Reporting - Permissions Catalog

This document defines all atomic permissions for the Reporting platform capability.

---

## Permissions

| Category                       | Permission ID                          | Description                                                                                | Applicable Scopes                  | Notes                                                                                                |
| ------------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Catalog Management             | `reporting.catalog.view`               | View the report catalog and browse available reports                                       | System-wide                        | All authenticated users; individual reports are further gated by their declared domain permission    |
| Catalog Management             | `reporting.catalog.manage`             | Register, edit, or deactivate report definitions in the catalog                            | System-wide                        | System Admin only; does not affect the underlying Power BI report definition                         |
| Embed Token                    | `reporting.embed-token.generate`       | Trigger embed token generation for an eligible report                                      | System-wide                        | Not directly role-assignable; granted implicitly when the user holds the report's declared domain permission at an appropriate scope. The Reporting API enforces this — it is not a standalone assignable permission. |
| Audit                          | `reporting.embed-audit.view`           | View embed token request audit log                                                         | System-wide                        | System Admin only                                                                                    |
| Domain-Specific: credentialing | `reporting.credentialing-reports.view` | View application status and volume reports                                                 | System-wide, Entity, District, ISD | Scope-aware reporting. Migrated from credentialing.reports.view (credentialing-permissions.ttl, formerly cred:Perm_ReportsView) — see this file's header. Not required by any sd:ApiOperation via a static PermissionRequirement; it's consumed dynamically as a Power BI ReportDefinition's declared PermissionKey string (see rpt:PermissionKeyVO), the same as before the rename. |
| Domain-Specific: documents     | `reporting.documents-reports.view`     | View document lifecycle and compliance reports                                             | System-wide                        | Migrated from documents.reports.view (documents-permissions.ttl, formerly docs:Perm_ReportsView) — see this file's header. Documents' own permissions file already noted this one had no matching PermissionRequirement (no dedicated reports endpoint in documents-api.ttl); unchanged by the move — still consumed dynamically as a Power BI ReportDefinition PermissionKey, not a static API gate. |
| Domain-Specific: epp           | `reporting.epp-reports.view`           | View EPP-specific reports (enrollment metrics, application statistics)                     | System-wide, Entity                | Migrated from epp.reports.view (epp-permissions.ttl, formerly epp:Perm_ReportsView) — see this file's header. epp-api.ttl has no /reports endpoint; same documented gap as before the move, now living alongside Reporting's other per-domain view permissions instead of in epp-permissions.ttl. |
| Domain-Specific: payments      | `reporting.payments-reports.view`      | View all payment reports (transactions, refunds, reconciliation, revenue, failed payments) | System-wide                        | Includes export. Migrated from payments.report.view (payments-permissions.ttl, formerly pay:Perm_ReportView) — see this file's header; found on the second migration sweep (singular 'report', not 'reports'). No API operation currently models this permission — payments-api.yml has no export endpoint; same documented gap as before the move. |
| Domain-Specific: proflearning  | `reporting.proflearning-reports.view`  | View and export all professional learning reports and analytics                            | System-wide                        | Migrated from proflearning.report.view (proflearning-permissions.ttl, formerly plrn:Perm_ReportView) — see this file's header; found on the second migration sweep (singular 'report', not 'reports'). proflearning-api.ttl defines no reporting/analytics endpoints; same documented gap as before the move. |
| Domain-Specific: staffing      | `reporting.staffing-reports.manage`    | Define and manage report availability and access for Staffing                              | System-wide                        | Migrated from staffing.admin.manage-reports (staffing-permissions.ttl, formerly staff:Perm_AdminManageReports) — see this file's header for an important caveat: fdd-review/10-staffing-admin's own review already links this exact story to the existing GLOBAL rpt:Perm_CatalogManage as a 'good match,' which may be too loose since that permission is System-Admin-only and covers every domain's reports, not a Staffing-scoped delegation. No sd:ApiOperation implements per-domain-scoped catalog management yet (only the global reporting.catalog.manage exists) — this permission is a placeholder anchor for that still-missing capability, tracked in DESIGN-SPIKES.md item 3, not a currently-enforced permission. |

---

## Notes on Permission Model

### Report-Level Access Is Governed by Domain Permissions

The Reporting capability itself does not define per-report permissions. Instead, each registered report in the catalog declares a `PermissionKey` that maps to a permission owned by another domain (e.g., `profpractice.reports.view`, `staffing.reports.view`). The Reporting API enforces this check at token generation time.

This means:
- Domain teams own and define the permissions that gate their reports
- The Reporting capability acts as the enforcement point, but not the definition point
- A user who holds `profpractice.reports.view` at `ISD` scope will be able to view PPR reports scoped to that ISD — the scope is evaluated by IAM and passed as resolved parameters

### `reporting.embed-token.generate` Is Not Directly Assignable

This permission is a technical gate, not a role-assignable permission. It exists to document the authorization behavior of the embed token endpoint. The actual authorization decision is: "Does this user hold the `PermissionKey` declared on this specific report?" That check is delegated to the IAM domain.

---

## Cross-Domain Permissions

Permissions from other domains that Reporting commonly checks:

| Permission                   | Domain        | When Reporting Checks It                                                                     |
| ---------------------------- | ------------- | -------------------------------------------------------------------------------------------- |
| `iam.authorization.view`     | iam           | Resolving the user's active authorizations to determine org context for parameter resolution |
| `profpractice.reports.view`  | profpractice  | Before generating embed token for any PPR-associated report                                  |
| `credentialing.reports.view` | credentialing | Before generating embed token for any Credentialing-associated report                        |
| `staffing.reports.view`      | staffing      | Before generating embed token for any Staffing-associated report                             |
| `proflearning.reports.view`  | proflearning  | Before generating embed token for any Professional Learning-associated report                |
| `epp.reports.view`           | epp           | Before generating embed token for any EPP-associated report                                  |

> **Note for domain teams:** If your domain has reports that should be accessible in the portal, ensure a `{domain}.reports.view` permission is defined in your domain's permissions catalog. The Reporting catalog will reference it as the `PermissionKey` for your reports.
