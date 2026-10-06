# Payments - Permissions Catalog

This document defines all atomic permissions for the Payments platform capability.

---

## Permissions

| Category               | Permission ID                                 | Description                                                                                | Applicable Scopes                                   | Notes                                                                                        |
| ---------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **Payment Initiation** | `payments.transaction.initiate`               | Initiate payment for a credential application                                              | Individual                                          | Implicit baseline — any authenticated user can pay fees for their own applications           |
| **Payment Initiation** | `payments.transaction.initiate-bulk`          | Initiate bulk payment for multiple applications within scope                               | District, ISD (transitive), System-wide             | District staff paying fees for multiple applications in their organization                   |
| **Payment Viewing**    | `payments.transaction.view`                   | View payment transactions within organizational scope                                      | Individual, District, ISD (transitive), System-wide | System-wide grants full visibility across all transactions                                   |
| **Payment Retry**      | `payments.transaction.retry`                  | Retry a failed payment transaction                                                         | Individual, System-wide                             | Implicit baseline — user can re-initiate after failure; each retry creates a new transaction |
| **Refunds**            | `payments.refund.request`                     | Submit a refund request for a paid transaction                                             | System-wide                                         | Credential Admin; requires justification; audit logged                                       |
| **Refunds**            | `payments.refund.request-bulk`                | Submit refund requests for multiple transactions at once                                   | System-wide                                         | Credential Admin; audit logged                                                               |
| **Refunds**            | `payments.refund.approve`                     | Approve or reject refund requests                                                          | System-wide                                         | Financial Admin; dual approval required for amounts >$500; audit logged                      |
| **Refunds**            | `payments.refund.view`                        | View refund requests and history                                                           | System-wide                                         | Credential Admin, Financial Admin                                                            |
| **Reconciliation**     | `payments.reconciliation.view`                | View reconciliation batch results and daily posting file processing                        | System-wide                                         | Payment Admin                                                                                |
| **Reconciliation**     | `payments.reconciliation.run`                 | Manually trigger reconciliation batch job                                                  | System-wide                                         | Payment Admin; audit logged                                                                  |
| **Reconciliation**     | `payments.reconciliation.discrepancy-view`    | View unresolved reconciliation discrepancies                                               | System-wide                                         | Payment Admin                                                                                |
| **Reconciliation**     | `payments.reconciliation.discrepancy-resolve` | Manually resolve reconciliation discrepancies                                              | System-wide                                         | Payment Admin; requires resolution notes; audit logged                                       |
| **Reconciliation**     | `payments.reconciliation.export`              | Export reconciliation data for financial audit                                             | System-wide                                         | Payment Admin, Financial Admin                                                               |
| **Reports**            | `payments.report.view`                        | View all payment reports (transactions, refunds, reconciliation, revenue, failed payments) | System-wide                                         | Includes export; Credential Admin, Payment Admin, Financial Admin                            |
| **Audit**              | `payments.audit.view`                         | View and export payment audit logs with advanced filtering                                 | System-wide                                         | System Admin, Compliance Officer; includes search and export                                 |
| **Administration**     | `payments.admin.force-status`                 | Manually override a payment transaction status                                             | System-wide                                         | System Admin only; emergency use; requires justification; audit logged                       |
| **Administration**     | `payments.admin.circuit-breaker-reset`        | Reset CEPAS circuit breaker after outage                                                   | System-wide                                         | Payment Admin; audit logged                                                                  |

---

## Implicit Baseline (All Authenticated Users)

The following capabilities are available to any authenticated user without an explicit permission grant.

> **Note:** Payments does not independently verify application ownership. The calling domain or UI is responsible for ensuring a user may act on a given application before invoking payment flows. These baseline capabilities describe what Payments will honor once a legitimate, authenticated request arrives.

| Capability                                      | Notes                                                                                        |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Initiate payment for own credential application | "Pay Fee" link shown when application requires payment; amount is immutable after initiation |
| View own payment transaction history            | Status, confirmation numbers, amounts for own applications                                   |
| Retry a failed payment                          | Each retry creates a new transaction; original failed transaction preserved in audit trail   |
| View payment confirmation/receipt               | Accessible after successful payment                                                          |

---

## Scope Definitions

**System-wide:** Permission applies across the entire system with no organizational scoping.

**District:** Permission applies to a district and all its constituent buildings.

**ISD:** Permission applies to an ISD and all constituent districts and their buildings.

**Transitive:** If granted at ISD level, automatically applies to all constituent districts and buildings within that ISD. If granted at District level, applies to all buildings within that district.

**Individual:** Permission applies only to the authenticated user's own payment transactions and applications. Common for citizen-type user access.

**Example (transitive):**
- User has `payments.transaction.view` at **District 12345** (transitive)
- User can view payment transactions for all credential applications submitted within District 12345 and its buildings
- User has `payments.transaction.initiate-bulk` at **District 12345**
- User can submit bulk payments for multiple applications for staff within District 12345

---

## Special Cases & Notes

### Refund Approval Thresholds
- Refunds **≤$500** require a single approval from Financial Admin
- Refunds **>$500** require dual approval: Credential Admin initiates (`payments.refund.request`), Financial Admin authorizes (`payments.refund.approve`)
- The threshold is configurable; the default is $500
- System processes approved refunds automatically via CEPAS API — no manual API interaction by admin

### Bulk Payment Scope Enforcement
- Users with `payments.transaction.initiate-bulk` can only select applications within their authorized organizational scope
- System validates all selected applications are within scope before payment initiation
- If any application is out of scope, the entire bulk payment request is rejected

### Emergency Status Override
- `payments.admin.force-status` is for emergency use only (e.g., CEPAS confirmed payment but system error prevented status update)
- All manual overrides require justification text and are logged to the audit trail
- Should trigger notification to Financial Admin and System Admin

### CEPAS Configuration
- Encryption keys, security keys, API credentials, and SFTP credentials are managed via Azure Key Vault and infrastructure configuration — not through any application UI or API
- No application-level permissions are defined for CEPAS credential management; access is controlled at the platform/infrastructure level