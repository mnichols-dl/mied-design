# [Domain Name] - Permissions Catalog

**Domain:** [Full Name]  
**Version:** [X.Y]  
**Last Updated:** [YYYY-MM-DD]

This document defines all atomic permissions for the [Domain Name] domain.

---

## Permissions

<!--
PERMISSION AUTHORING RULES (remove this comment block before publishing):

NAMING PATTERN: `{domain}.{resource}.{action}`
- domain: kebab-case domain identifier (e.g., credentialing, iam, staffing)
- resource: kebab-case resource name (e.g., application, credential, endorsement-definition)
- action: lowercase verb from the approved list below

APPROVED ACTION VERBS (use consistently, only expand when necessary):
  view, search, create, edit, delete, approve, deny, export,
  configure, manage, import, submit, suspend, revoke, reinstate,
  nullify, on-hold, bypass-validation, impersonate, compare-versions,
  manual-submit, print, issue

SCOPE TYPES (use exactly as defined in Scope Definitions section):
  System-wide, Entity, District, ISD, Self-only, Individual, Worklist-specific

RULES:
- Include ALL permissions owned and enforced by THIS domain in a single table
- Use the Category column to organize; do NOT create separate tables per category
- Do NOT include permissions from other domains (those go in Cross-Domain section)
- Do NOT include role definitions or UI-specific permissions
- If a permission applies at multiple scopes, list all in the Applicable Scopes column
  (e.g., "System-wide, Entity, District, ISD (transitive)")
- Duplicate rows for the same permission ID indicate a copy-paste error; consolidate

ALIGNMENT CHECKLIST:
- Every aggregate state transition in the domain model should map to a permission
- Admin-only operations should note "System Admin only" or equivalent in Notes
- Audit-logged operations should note "audit logged" in Notes
- Every sequence diagram actor action should be traceable to at least one permission here
-->

| Category | Permission ID | Description | Applicable Scopes | Notes |
|----------|---------------|-------------|-------------------|-------|
| [Category] | `[domain].[resource].[action]` | [What this allows] | [Scope(s)] | [Constraints, role restrictions, or audit notes] |

---

## Scope Definitions

<!--
SCOPE AUTHORING RULES (remove this comment block before publishing):

- Define ONLY scope types that appear in the permissions table above
- Provide a transitive inheritance example if any scope is marked transitive
- Use consistent terminology across all domain permission docs
- "Individual" and "Self-only" are distinct:
    Self-only = user can only act on their OWN records (applies to any user)
    Individual = permission applies to a single named person (used for citizen-type scopes)
-->

**System-wide:** Permission applies across the entire system with no organizational scoping.

**Entity, District, ISD:** Permission is granted for a specific organizational scope.

**Transitive:** If granted at ISD level, automatically applies to all constituent districts and buildings within that ISD. If granted at District level, applies to all buildings within that district.

**Self-only:** Permission applies only to the user's own records (e.g., withdraw own application, print own certificate).

**Individual:** Permission applies only to a specific named person's data. Common for citizen-type user access.

**Worklist-specific:** Permission applies only within worklists explicitly assigned to the user.

**Example (transitive):**
- User has `credentialing.application.view` at **ISD 50** (transitive)
- User can view applications for ISD 50, all districts within ISD 50, and all buildings within those districts
