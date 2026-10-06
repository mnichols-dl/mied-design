# Reporting - Permissions Catalog

This document defines all atomic permissions for the Reporting platform capability.

---

## Permissions

| Category           | Permission ID                    | Description                                                     | Applicable Scopes | Notes                                                                                                                                                                                                                 |
| ------------------ | -------------------------------- | --------------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Catalog Management | `reporting.catalog.view`         | View the report catalog and browse available reports            | System-wide       | All authenticated users; individual reports are further gated by their declared domain permission                                                                                                                     |
| Catalog Management | `reporting.catalog.manage`       | Register, edit, or deactivate report definitions in the catalog | System-wide       | System Admin only; does not affect the underlying Power BI report definition                                                                                                                                          |
| Embed Token        | `reporting.embed-token.generate` | Trigger embed token generation for an eligible report           | System-wide       | Not directly role-assignable; granted implicitly when the user holds the report's declared domain permission at an appropriate scope. The Reporting API enforces this — it is not a standalone assignable permission. |
| Audit              | `reporting.embed-audit.view`     | View embed token request audit log                              | System-wide       | System Admin only                                                                                                                                                                                                     |

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