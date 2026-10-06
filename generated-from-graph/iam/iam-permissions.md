# Identity & Access Management (IAM) - Permissions Catalog

This document defines all atomic permissions for the IAM domain.

**Design decision (2026-08-14):** This catalog previously included four `iam.group.*`
permissions (view/create/edit/delete) implying "User Group" was a first-class,
membership-bearing entity. No `UserGroup` aggregate, sequence, or API endpoint was ever
built to back them — they were orphaned scaffolding. The confirmed direction going
forward is **Role + Scope only**: atomic permissions bundled into Roles, Roles assigned at
an organizational scope (with transitive inheritance down the org hierarchy where
applicable). "Group" is not modeled as a separate governance layer unless a concrete need
surfaces that Role + Scope can't express. The one behavior "Group" was informally
expected to enforce — that a user's access stays within one MiLogin identity type
(Citizen/Business/Worker) — is now a `RoleDefinition` attribute instead (see
`ApplicableMiLoginTypes` under `RoleDefinition` in `iam-domain.md`). The four permissions
above were removed rather than left in place unenforced. Pending final client
concurrence — see `fdd-sdd-review/client-questions.md` #2.

---

## Permissions

Permissions follow the pattern: `{domain}.{resource}.{action}`

| Category               | Permission ID                              | Description                                                                                          | Applicable Scopes                                   | Notes                                                                                                |
| ---------------------- | ------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| User Accounts          | `iam.user.view`                            | View user account details and list                                                                   | System-wide                                         | Read-only access to user directory                                                                   |
| User Accounts          | `iam.user.edit`                            | Edit user account details                                                                            | System-wide                                         | System Admin only; modify contact info, flags                                                        |
| User Accounts          | `iam.user.impersonate`                     | View system as another user (read-only)                                                              | System-wide                                         | All actions audit-logged; no transactions allowed                                                    |
| Authorization Requests | `iam.authorization.view`                   | View authorization requests                                                                          | Building, District, ISD (transitive) or System-wide | Scope-aware: Organization admins see their Organization only                                         |
| Authorization Requests | `iam.authorization.approve`                | Approve user authorization requests                                                                  | Building, District, ISD (transitive)                | Must be Lead Admin for Organization                                                                  |
| Authorization Requests | `iam.authorization.deny`                   | Deny user authorization requests                                                                     | Building, District, ISD (transitive)                | Must be Lead Admin for Organization; can add denial reason                                           |
| Authorization Requests | `iam.authorization.grant`                  | Manually grant authorization without approval                                                        | System-wide                                         | System Admin only; bypasses approval workflow                                                        |
| Authorization Requests | `iam.authorization.revoke`                 | Revoke/remove user authorization                                                                     | Building, District, ISD (transitive) or System-wide | Organization admin for scope or System Admin                                                         |
| Authorization Removal  | `iam.removal-request.review`               | Review public authorization removal requests                                                         | System-wide                                         | System Admin only; approve/deny external requests                                                    |
| Roles                  | `iam.role.view`                            | View role definitions                                                                                | System-wide                                         | See role names, descriptions, assigned permissions                                                   |
| Permissions            | `iam.permission.view`                      | View permission catalog                                                                              | System-wide                                         | See all defined permissions                                                                          |
| Inactivity Policy      | `iam.inactivity.configure`                 | Set auto-deactivation rules                                                                          | System-wide                                         | Configure threshold (days) and schedule                                                              |
| Inactivity Policy      | `iam.inactivity.run`                       | Manually trigger inactivity deactivation job                                                         | System-wide                                         | System Admin only; for testing/emergency runs                                                        |
| Approval Links         | `iam.approval-link.configure`              | Configure approval link expiration                                                                   | System-wide                                         | Set default expiration timeframe (days)                                                              |
| Reports                | `iam.reports.view`                         | View User Management Report                                                                          | System-wide                                         | System Admin only                                                                                    |
| Audit Logs             | `iam.audit.view`                           | View authorization audit logs                                                                        | System-wide or Organization                         | Scope-aware: Organization admins see their Organization only                                         |
| Identity Administrator | `iam.identity-admin.view-pending-requests` | View identity requests awaiting administrator review, of any origin or type                          | System-wide                                         | Identity Administrator role; a distinct role from Staffing Data Admin or System Admin — reviewing and resolving identity requests is not a function of either |
| Identity Administrator | `iam.identity-admin.resolve-request`       | Resolve an identity request awaiting administrator review (match, create new, approve/deny link, cancel) | System-wide                                         | Identity Administrator role only                                                                     |
| Identity Administrator | `iam.identity-admin.update-person-record`  | Directly update a person's demographic record via Mi-Key, bypassing matching                         | System-wide                                         | Identity Administrator role only; rare-use path distinct from citizen/business-user self-service demographic updates |
| Identity Administrator | `iam.identity-admin.manage-configuration`  | Manage identity-specific system configuration: validation rules, data element definitions, training materials, and processing queue monitoring | System-wide                                         | Identity Administrator role; narrower than the general `iam.*` system-configuration permissions, which do not cover identity processing specifically |
|                        | `iam.authorization.remove-own`             | Remove own active authorization (self-service)                                                       | Self-only                                           | Users can revoke their own access to organizations without approval. Immediate effect.               |
|                        | `iam.authorization.request`                | Submit authorization request for organizational access                                               | Self-only                                           | Business/Worker users without existing authorizations can request access to organizations. Citizen users receive automatic Individual scope authorization instead. |
|                        | `iam.authorization.view-own`               | View own authorization requests and active authorizations                                            | Self-only                                           | Users can see their pending requests, approval status, and current active authorizations.            |
|                        | `iam.authorization.withdraw`               | Withdraw own pending authorization request                                                           | Self-only                                           | Users can cancel requests before Lead Admin approval.                                                |
|                        | `iam.citizen-identity.cancel-own-request`  | Cancel own pending Account Update request (Near Match only)                                          | Self-only                                           | Citizen users only; Account Creation requests cannot be self-cancelled once RequiresResolution (Identity Admin only). |
|                        | `iam.citizen-identity.submit-request`      | Submit an Account Creation or Account Update identity request                                        | Self-only                                           | Citizen users only.                                                                                  |
|                        | `iam.citizen-identity.view-own-request`    | View own pending/historical identity request status                                                  | Self-only                                           | Citizen users only; never exposes candidate match data (see iam-domain.md Near Match Resolution rule). |
|                        | `iam.user.view-own`                        | View own user profile                                                                                | Self-only                                           | Read-only access to own account details (name, email, MiLogin type, contact info).                   |

### All Authenticated Users (Any MiLogin Type)

The following permissions are implicitly granted to an individual as a baseline. These are documented here for completeness.

| Permission ID                  | Description                                               | Notes                                                                                                                                                      |
| ------------------------------ | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `iam.authorization.request`    | Submit authorization request for organizational access    | Business/Worker users without existing authorizations can request access to organizations. Citizen users receive automatic Individual scope authorization. |
| `iam.authorization.view-own`   | View own authorization requests and active authorizations | Users can see their pending requests, approval status, and current active authorizations.                                                                  |
| `iam.authorization.withdraw`   | Withdraw own pending authorization request                | Users can cancel requests before Lead Admin approval. Already cataloged above as explicit permission for clarity.                                          |
| `iam.authorization.remove-own` | Remove own active authorization (self-service)            | Users can revoke their own access to organizations without approval. Immediate effect.                                                                     |
| `iam.user.view-own`            | View own user profile                                     | Read-only access to own account details (name, email, MiLogin type, contact info).                                                                         |
| `iam.citizen-identity.submit-request` | Submit an Account Creation or Account Update identity request | Citizen users only.                                                                                       |
| `iam.citizen-identity.view-own-request` | View own pending/historical identity request status  | Citizen users only; never exposes candidate match data (see `iam-domain.md` Near Match Resolution rule).                                                    |
| `iam.citizen-identity.cancel-own-request` | Cancel own pending Account Update request (Near Match only) | Citizen users only; Account Creation requests cannot be self-cancelled once `RequiresResolution` (Identity Admin only).                               |

---

## Scope Definitions

**System-wide:** Permission applies across entire system with no scoping restrictions. User can perform action for any Organization/user.

**Building:** Permission applies to a building.

**District:** Permission applies to a district and all its constituent buildings.

**ISD:** Permission applies to an ISD and all constituent districts and their buildings.

**Individual:** Permission applies only to the user's own data.

**Self-only:** Permission applies only to the user's own records (e.g., withdraw own authorization request).

**Transitive:** If granted at ISD level, permission automatically applies to all constituent districts and buildings within that ISD. If granted at District level, applies to all buildings within that district.

**Example:**
- User has `iam.authorization.approve` at **ISD 50** (transitive)
- User can approve requests for ISD 50, all districts within ISD 50, and all buildings within those districts
