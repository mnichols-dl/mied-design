# Credentialing - Technical Design

## Purpose

This document provides technical design details for Credentialing domain concepts that are
implementation-specific and don't belong in the domain model documentation. It covers
algorithmic and operational content flagged during FDD review as needing a home outside
`credentialing-domain.md`: admin-configurable parameters, scheduled/batch processing, and
document/attestation generation mechanics.

---

## Configuration Parameters

### Overview

FDD 05 ("System Administration - Credentialing," Feature 5.3, "Manage Global Parameters" /
"Credentialing Controls") describes an admin screen for editing named system-wide values —
its own examples span "credentialing landing page text" and "specific fee amounts." That's
a wide range: some of these are pure UI copy, some are values that already exist as
per-type fields elsewhere in the domain model (`CredentialDefinition.FeeSchedule` is
already per-credential-type, not shared), and some may be genuinely global (a single value
used the same way across every credential type).

### Open Design Question: Configurability Model

**Not yet resolved — needs a decision before this can be modeled.** There are at least
three shapes this could take, and the FDD doesn't distinguish between them:

1. **Fully independent per credential type.** Every `CredentialDefinition` carries its own
   value for everything (its own fee, its own landing-page text, etc.), with no sharing or
   default. This is already how `FeeSchedule` works today — but extending it to every
   configurable value means no "change it once, applies everywhere" lever, and N values to
   maintain for N credential types even when they're usually the same.
2. **Shared default + per-type override.** A named parameter has one global/default value;
   a `CredentialDefinition` may optionally override it. This is the most flexible model and
   matches the shape `CredentialProperties.CanApply`/`CanRenew` already hints at (per-type
   flags that could plausibly fall back to a system default), but it's more to build and
   needs a clear precedence rule (does an explicit `null`/unset override always mean
   "inherit," or does an admin need to actively "revert to default"?).
3. **Fully centralized, referenced everywhere.** One `GlobalParameter`/`CredentialingControl`
   key-value store; every credential type or business rule that needs a value reads it from
   there, with no per-type variation possible at all. Simplest to build, but doesn't
   accommodate the FDD's own permit-side evidence that per-type variation already exists in
   practice (e.g., permit duration/renewal limits are explicitly per-permit-type in
   `credentialing-domain.md`'s "Permit Duration and Renewal Limits" business rule — a fully
   centralized model would need to special-case those anyway).

None of these can be ruled out from the FDD text alone (see Open Technical Questions
below). Whichever is chosen, note that FDD 05 also raised whether "Global Parameters" is
credentialing-owned at all, or a shared cross-cutting capability other domains would also
use (see `patterns-and-principles/configuration.md`) — that's a separate axis from the
per-type-vs-shared question above, and both need an answer before drafting a
`GlobalParameter`/`CredentialingControl` aggregate in `credentialing-domain.md`.

### What's Already Modeled vs. What This Covers

Business-rule-relevant configurable values that already have a home should stay where they
are (e.g., `InactivityPolicy`-style fields directly on the relevant aggregate) — this
section is specifically about the *generic, admin-editable-by-name* pattern FDD 05
describes (arbitrary key/value pairs, edited through one screen, spanning fees and free
text), not a mechanism for every configurable value in the domain.

---

## Scheduled / Batch Processing

### Overview

FDD 05 (Feature 5.4, "Run & View Batch Jobs") describes an admin console for viewing batch
job run history, manually triggering predefined jobs, and scheduling jobs. Two existing
`credentialing-domain.md` business rules imply an underlying scheduled mechanism without
naming one; both are described below as if implemented as nightly jobs, consistent with
patterns used elsewhere in this repo (e.g., `documents-technical-design.md`'s "Hard Delete
Background Job," `organizations-capability.md`'s "Sync Cycle"). **Whether these should be
admin-visible/monitorable through FDD 05's job console, or run as purely internal
event-driven/on-access logic with no admin-facing run history, is an open question** — see
Open Technical Questions. The generic multi-job admin console itself (arbitrary job
search/trigger/schedule) may be broader platform tooling rather than a credentialing-owned
capability; this section only covers the two jobs credentialing's own business rules
already imply.

### Definition Version Activation ("Cleanup of Future-Dated Changes")

**Schedule:** Nightly (exact time TBD — see Open Technical Questions)

**Process:**
1. Query `CredentialDefinition`/`EndorsementDefinition` versions where `EffectiveFrom` is
   today's date or earlier and the version is not yet `Active`
2. For each: deactivate the currently `Active` version (transition to `Deprecated`),
   activate the future-dated version
3. Publish a definition-changed event per activated version (consumed by any service
   caching definition data)
4. Generate a summary: versions activated, versions still pending future activation

**Error Handling:**
- If an activation fails (e.g., referential integrity issue), skip that version, continue
  processing the rest, and alert — do not let one bad version block the whole batch
- Never activate a version whose `EffectiveFrom` is still in the future, regardless of
  processing delays

**Rationale:** Lets a Credential Admin schedule a definition change in advance (e.g., a fee
change effective at the start of a school year) without needing to be online at the moment
it should take effect — see `credentialing-domain.md`'s "Credential Definition Versioning"
business rule.

### Credential / Permit Expiration Processing

**Schedule:** Nightly (exact time TBD)

**Process:**
1. Query `ISSUED_CREDENTIALS` where `expires_at` is today's date or earlier and
   `status = Valid`
2. Transition each to `Expired`
3. Publish `CredentialExpired` per record (consumers: communications for expiration
   notices, staffing for roster-eligibility re-evaluation)
4. Generate a summary: credentials expired, notification events published

**Error Handling:**
- Retry failed transitions the next night (bounded retry count, consistent with
  `documents-technical-design.md`'s "Hard Delete Background Job" pattern)
- Never expire a credential whose `expires_at` is still in the future

**Rationale:** Permits and non-permanent credentials have hard expiration dates (see
"Permit Duration and Renewal Limits"); this keeps `IssuedCredential.status` accurate
without requiring an on-access check on every read.

---

## Certificate PDF Generation

*(Migrated from `credentialing-domain.md`'s "Certificate Document Generation" business
rule — the "Technical Implementation" detail belongs here, not embedded in a business rule
entry; that rule now points here instead.)*

**Trigger:** `CredentialIssued` event

**Process:**
1. PDF generation service consumes `CredentialIssued`
2. Retrieves credential details via the Credentialing API
3. Renders PDF using a template stored in the Documents capability
4. Stores the rendered PDF in Documents, referenced by credential ID

**Implementation Notes:**
- PDF generation via a third-party library or service (e.g., Puppeteer, wkhtmltopdf, or an
  Azure Functions–hosted PDF generation function) — specific choice not yet made, see Open
  Technical Questions
- Template stored as HTML/CSS in the Documents capability, not hardcoded in the
  Credentialing service, so template edits don't require a deployment
- Variable substitution into the template: `{{educator_name}}`, `{{credential_type}}`,
  `{{issuance_date}}`, `{{expiration_date}}`, `{{endorsements}}`, `{{credential_id}}`, etc.
  — see `credentialing-domain.md`'s "PDF Contents" list for the full field set
- **Blank dates for Revoked/Nullified/Suspended:** per FDD 29 (Feature 29.1.6, 29.1.9),
  regenerating or displaying a certificate for a credential in one of these three statuses
  renders `{{issuance_date}}`/`{{expiration_date}}` blank rather than the credential's
  actual historical dates — see `credentialing-domain.md`'s "Certificate Document
  Generation" business rule.

### Related, Not Yet Modeled: Cover Letter Generation

FDD 29 ("29 - Educator Credentialing - Certificates," Feature 29.1/29.4.2) describes a
distinct document-generation capability — a downloadable cover letter generated per
*pending application* (not per issued credential), using business rules keyed to the
certificate type applied for, with a "No Pending Applications" empty state and its own
download/logging behavior. This is not the same mechanism as Certificate PDF Generation
above (different trigger — pending application vs. `CredentialIssued` — and different
content). No aggregate, business rule, or sequence in `credentialing-domain.md` or
`credentialing-sequences.md` covers it today. Not drafted here since the underlying
per-certificate-type cover letter content/business rules aren't specified in the FDD
beyond "generated based on the business rule definition" — see the review's Coverage Gaps.

---

## Electronic Signature / Attestation Capture

### Overview

Every FDD-17 permit document and the FDD 11,16 certificate application both require the
submitter to attest before submission (e.g., "Electronic Signature: (free-form text)... 
Please type in your full name"). The FDD-level altitude review flagged typed-name entry as
an implementation choice, not a business rule — the actual business requirement is "the
submitter must provide a legally sufficient electronic signature/attestation." This section
exists so that requirement has a concrete technical home instead of being silently implied
by "there's a signature field somewhere in the UI."

### Current Baseline (Minimum Viable)

- Capture: submitter types their full legal name into a dedicated attestation field,
  distinct from any other free-text field on the form
- Capture also records: timestamp, submitter's authenticated identity (MiLogin ID),
  IP address, and the exact attestation text presented at signing time (so a later change
  to the attestation copy doesn't retroactively change what a past submitter agreed to)
- Stored as an immutable record attached to the `CredentialApplication` — not just a
  boolean "signed" flag

### Not Yet Decided

Whether typed-name entry is *legally sufficient* for Michigan state credentialing
purposes, or whether a stronger mechanism (drawn signature capture, a third-party e-sign
integration such as DocuSign/Adobe Sign, or a click-to-attest pattern with additional
identity verification) is required, is a legal/compliance question this document can't
answer on its own — see Open Technical Questions.

---

## Open Technical Questions

| #   | Question                                                                                                                                                                                                                                                                     | Impact | Owner                | Target Date |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | -------------------- | ----------- |
| 1   | Which configurability model does the client actually want for admin-editable parameters: (a) fully independent per credential type, (b) a shared default with per-type override, or (c) fully centralized with no per-type variation? See "Configuration Parameters" above — this determines the shape of any `GlobalParameter`/`CredentialingControl` aggregate. | H      | Product               | TBD         |
| 2   | Is "Global Parameters"/"Credentialing Controls" a credentialing-owned concept, or a shared cross-domain configuration capability other domains would also consume?                                                                                                          | M      | Product / Platform    | TBD         |
| 3   | Should Definition Version Activation and Credential/Permit Expiration Processing be admin-visible, schedulable jobs through FDD 05's "Run & View Batch Jobs" console, or purely internal scheduled logic with no admin-facing run history?                                 | M      | Product               | TBD         |
| 4   | Exact nightly run time for the two batch jobs above (coordinate with other domains' nightly jobs, e.g. Organizations' CEPI sync, to avoid contention).                                                                                                                       | L      | Infrastructure        | TBD         |
| 5   | PDF generation library/service selection (Puppeteer vs. wkhtmltopdf vs. Azure Functions–hosted rendering) — affects hosting/cost/maintenance, not behavior.                                                                                                                  | L      | Engineering           | TBD         |
| 6   | Is typed-name electronic signature capture legally sufficient for state credentialing attestations, or is a stronger mechanism (drawn signature, third-party e-sign vendor) required?                                                                                       | H      | Legal / Compliance    | TBD         |
| 7   | How should Credentialing check current payment status for an application? `payments-api.yml` has no internal-service status-check endpoint — `GET /payments/search` is a scoped, permission-gated, org-context-requiring admin/UI endpoint, not built for this. Options: (a) rely purely on the `PaymentCompleted` event subscription with no synchronous check, or (b) request a new internal-service endpoint from the payments team (mirroring the pattern used for `GET /educators/{id}/ppr-clearance`). Also confirm `credentialing-domain.md`'s "Payment Requirement" (one fee per school year) has no counterpart on the payments side — per the FDD 07 review, `PaymentTransaction` has no academic-year concept, so this stays a credentialing-side dedup check before ever calling payments, not something payments needs to model. | H | Engineering (credentialing + payments) | TBD |
| 8   | Two Credentialing-to-Staffing integration points have no confirmed shape, surfaced during the FDD 10 (Staffing Admin) review: (a) `credentialing-domain.md`'s "Mentor Assignment Requirement for Permits" rule calls `GET /educators/{uniqueId}` on the Staffing API expecting "verify educator exists and holds valid, relevant credential" — `staffing-api.yml` has no such endpoint, and the closest one (`GET /educators/search`) only returns a free-text `activeCredentialSummary` string, while `staffing-domain.md`'s own Dependencies table has *Staffing* calling *Credentialing* for credential validation, the reverse of what this rule assumes; (b) `credentialing-sequences.md`'s "Submit Permit Application" sequence calls `Staffing: Verify employment assignment (if required)` with no corresponding endpoint anywhere in `staffing-api.yml` (closest is the org-scoped, permission-gated `GET /employee-roster/search`, not built as a single-lookup verification call). Options for (a): have Credentialing call its own domain data directly (no cross-domain call needed, since credential validity is credentialing-owned) and let the free-text `activeCredentialSummary` remain a UI-only convenience field on staffing's side; or add a purpose-built endpoint. Options for (b): confirm whether "employment assignment" verification is even a real requirement (no FDD source describes it) or add a purpose-built internal-service endpoint to `staffing-api.yml` if it is. | H | Engineering (credentialing + staffing) | TBD |
