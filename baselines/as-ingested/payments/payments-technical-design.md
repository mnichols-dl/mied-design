# Payments Technical Design

---

## Data Store Architecture

Payments uses **Azure SQL Database** rather than Cosmos DB. This is an intentional exception to the platform default and is driven by the nature of financial data.

**Why SQL:**
- **ACID transactions are non-negotiable for financial data.** Payment status updates must be atomic and immediately consistent. Eventual consistency is not acceptable — a user who completes payment must not be able to initiate a second payment because their status hasn't propagated yet.
- **Reconciliation requires relational queries.** Matching posting file records against internal transactions involves JOINs, amount comparisons, date range filtering, and aggregations. These are expensive and awkward in a document store.
- **Bulk payment atomicity.** When a bulk payment is confirmed, all child transaction records must update together. SQL transactions provide this natively; Cosmos cross-partition transactions do not.
- **Audit compliance.** SQL Server temporal tables provide built-in point-in-time history tracking required for the 7-year financial audit trail. Equivalent behavior in Cosmos requires custom change feed processing.

**Schema design follows directly from the domain model.** See the Payments capability doc for the full ERD. The core tables map to the domain aggregates: `PaymentTransactions`, `BulkPaymentTransactions`, `RefundRequests`, `ReconciliationBatches`, and `ReconciliationDiscrepancies`. All state-bearing tables use system-versioned temporal tables for immutable audit history.

---

## Concurrency Control

Payment confirmation updates use optimistic concurrency via a `rowversion` column. Any update to a payment transaction must supply the expected row version; if the row has been modified by another process in the interim, the update fails and the application layer retries with a fresh read.

This prevents lost updates in scenarios where two processes attempt to confirm the same transaction simultaneously — for example, a real-time redirect confirmation racing with the posting file reconciliation job.

---

## Transaction Lifecycle State Machine

Valid status transitions are enforced at the database layer via triggers, not just application logic. The key invariants:

- `Paid`, `Refunded`, and `PartiallyRefunded` transactions cannot revert to `Pending` or `Failed`
- `Refunded` is a terminal state — no further status changes are permitted
- Bulk payment confirmation updates parent and all child transactions atomically; if the child count after update does not match the expected item count, the transaction is rolled back

---

## Circuit Breaker Configuration

CEPAS availability is monitored via a circuit breaker to prevent cascading failures during outages.

| Parameter               | Value                  |
| ----------------------- | ---------------------- |
| Failure threshold       | 3 consecutive failures |
| Health check timeout    | 5 seconds              |
| Cooldown period         | 5 minutes              |
| Half-open test requests | 1                      |

**States:**
- **Closed (normal):** All requests pass through to CEPAS
- **Open (failing):** Requests blocked immediately; user shown availability error
- **Half-open (recovering):** Single test request allowed; success closes circuit, failure re-opens it

The circuit breaker applies to both the payment redirect health check and the refund API. Alerts fire when the circuit opens.

---

## Posting File Reconciliation Logic

The reconciliation job runs nightly at 3:00 AM EST, after CEPAS posts the previous day's transactions (typically ~2:00 AM EST).

Each AD record in the posting file is matched against internal payment transactions using the custom reference data field (which contains the application ID). Records are categorized as:

| Result                | Condition                              | Action              |
| --------------------- | -------------------------------------- | ------------------- |
| **Match**             | Found, status=Paid, amounts equal      | No update needed    |
| **Missing Payment**   | Found, status=Pending, amounts equal   | Auto-update to Paid |
| **Amount Mismatch**   | Found but amounts differ               | Flag as discrepancy |
| **Orphan Payment**    | No matching internal transaction found | Flag as discrepancy |
| **Unexpected Status** | Found but status is Failed or Refunded | Flag as discrepancy |

Auto-updates only apply to `Pending` transactions. `Failed` and `Refunded` transactions are never overwritten by the posting file — these require manual discrepancy resolution.

All auto-updates and discrepancy insertions for a given batch are wrapped in a single SQL transaction to ensure the batch is processed atomically.

---

## Performance Considerations

- Reconciliation queries use a composite index on `(application_id, status, created_date)` with `amount` and `confirmation_number` as included columns, supporting the posting file matching logic without full table scans.
- Confirmation number lookups and application ID + status lookups are each covered by dedicated indexes.
- The expected data volume (100–500 posting file records per day, ~4,000 Michigan educational organizations) is well within the buffer pool capacity of the provisioned SQL tier. Reconciliation batch processing targets completion within 10 minutes.
- Read replicas should be considered if primary database DTU utilization exceeds 70% sustained during reporting query load.

---

## Integration Details

The following integration specifications live in dedicated documents and are not duplicated here:

- **CEPAS Payment Redirect** (encrypted redirect flow, AES-GCM payload format, return parameters, credential management) -> See: *CEPAS Integration Spec*
- **CEPAS Refund API** (endpoint, authentication, request/response contract, retry behavior) -> See: *CEPAS Integration Spec*
- **CEPAS Posting File** (SFTP retrieval via FTS, fixed-width file format, AD record structure) -> See: *CEPAS Integration Spec*
- **Hash Validation** (SHA-1 calculation, timing attack prevention) -> See: *CEPAS Integration Spec*