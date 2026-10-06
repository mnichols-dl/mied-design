# Organizations — Permissions Catalog

This document defines all atomic permissions for the Organizations capability.

---

## Permissions

| Category | Permission ID | Description | Applicable Scopes | Notes |
|----------|---------------|-------------|-------------------|-------|
| Organizations | `organizations.organization.view` | View organization details, grade band, and Lead Administrator | System-wide | Granted to all authenticated users; required to use org search and detail endpoints |
| Organizations | `organizations.lead-admin.view` | Retrieve Lead Administrator contact for an organization | System-wide | Internal service permission; consumed by IAM for authorization approval routing. Lead Admin email not exposed to non-admin users directly. |
| Organizations | `organizations.org-types.view` | List active organization types | System-wide | Granted to all authenticated users; used to populate type filter dropdowns |
| Organizations | `organizations.sync.trigger` | Manually trigger an organization sync run | System-wide | System Admin only; reserved for emergency re-seeding or post-failure recovery. Normal sync is pipeline-driven. |
| Organizations | `organizations.sync-log.view` | View sync execution history and status | System-wide | System Admin / Platform Operations only |

---

## Scope Definitions

**System-wide:** All organization permissions apply across the entire system without organizational scoping. The Organizations capability describes *institutions*, not user-scoped data — there is no concept of "view only your district's org record."

**Self-only / Individual:** Not applicable to this capability.

**Note on Internal Service Permissions:** `organizations.hierarchy.view` and `organizations.lead-admin.view` are marked as internal service permissions. They are enforced at the Istio authorization policy layer (only IAM's service account principal is permitted to call these endpoints) in addition to the standard IAM permission check. User-facing roles do not include these permissions.
