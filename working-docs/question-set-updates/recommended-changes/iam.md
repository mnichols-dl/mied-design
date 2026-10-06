# Recommended Changes: IAM

**Impact: Medium.** No IAM domain model change. Permission registration, scope handling, and a few role questions raised by the POC. Files under `design/working-docs/solution-areas/iam/`.

## Changes

1. **Register the new permissions** from `questionsets-permissions.md` in the solution-wide permission catalog flow that IAM's role-and-scope model consumes. Roles remain Role plus Scope only (the 2026-08-14 decision, pending client concurrence). No UserGroup entity is added.
2. **Scope for functional areas.** The authoring permissions repeat per functional area (`credentialing`, `profpractice`, `proflearning`, `epp`). Confirm that role definitions can bundle the per-area permissions (for example "Credentialing Admin" gets the `credentialing` set) without a new scope type. If a single role needs several functional areas, that is a normal multi-permission grant.
3. **Self-only scope on a capability permission.** `questionsets.response.complete` is Self-only. Confirm IAM's scope enforcement handles Self-only where the subject is bound to a response rather than to a resource owned by the caller's profile.
4. **On-behalf entry and impersonation.** Staff entering answers for an educator (admin-issued permits) must record actor and subject separately. Confirm that impersonation sessions surface the actor id in claims so Question Sets can store both. If impersonation is used for this, document it in the IAM impersonation section (`iam-technical-design.md`).
5. **Reviewer types: permission versus role.** The 9/29 POC note asks whether a reviewer type (for example PPR specialist) comes from permissions or roles, and whether admins can preview roles with permissions. Recommendation: reviewer type is derived from a permission, so "can review this kind of case" is not coupled to a role name. IAM should offer (or confirm) a way for an admin UI to list the permissions bundled by a role and preview "what would this role see". Related existing discrepancies: REV 05 (Credential Admin versus System Administrator Credentialing as synonyms), REV 09 (who manages PPR questions, markers, deletes).
6. **Prefill sources from IAM.** Full name, email and unique id are IAM or MiLogin values. Register them as prefill sources with resolution mode Provider (identity) or ConsumerSupplied. Confirm whether IAM exposes a read endpoint for the current subject's profile that is safe to call from Question Sets, or whether the consumer passes the values.
7. **Audit.** IAM's audit expectations for reading sensitive data apply to response reads. Reuse the same audit event shape if one exists.

## No change needed

- `iam-domain.md` aggregates. IAM's identity, authorization and public removal request forms are fixed forms with no dynamic-question need, so they do not become Question Sets consumers.
