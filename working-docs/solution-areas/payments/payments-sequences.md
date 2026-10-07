# Payments - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Payments platform capability.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget (events published to the Event Bus)
- **actor** = Human only
- **participant** = Every non-human (services, Event Bus, external systems)
- Every request arrow starts with an API kind tag: `APP` = Application API (UI to owning API, user token), `SVC` = Service API (API to API, in-cluster mTLS), `EXT` = External API (inbound call from outside, including public endpoints), `OUT` = outbound call to an external system. Responses carry no tag.
- Participants are grouped with `box`: Browser (UI), MiEdWorkforce (AKS) (services and Event Bus), External (external systems).
- Every application API call is authorized by the owning service through the cached IAM permission check (Service API). It is not drawn unless noted.

---

## Individual Payment Flow (Happy Path)

**What:** User initiates payment for credential application fee, completes payment on CEPAS, and returns to MiEdWorkforce with confirmation  
**When:** User clicks "Pay Fee" link on pending credential application  
**Who:** Citizen User, Business User, or State Admin/Staff. Permission: `payments.transaction.initiate` (implicit baseline; the user's own application). The confirmation step is a public endpoint (no permission; hash validated).

```mermaid
---
title: Payments - Individual Payment Flow (Happy Path)
---
sequenceDiagram
    actor User
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
    end

    User->>UI: Click "Pay Fee" link
    UI->>PayApi: APP POST /payments/initiate

    PayApi->>PayApi: Create PaymentTransaction<br/>Status: Pending<br/>Amount: $100.00 (from credential type)<br/>Reference: APP-123
    PayApi->>PayApi: Encrypt payload with AES GCM<br/>(amount, ref, id, returnurl)

    Note over PayApi: Encrypted format:<br/>aen=<key_id>|<12B_nonce><ciphertext+tag>

    PayApi-->>UI: 200 cepasRedirectUrl<br/>URL: https://epay.michigan.gov/payment?aen=...
    UI->>CEPAS: OUT Browser redirect to CEPAS

    User->>CEPAS: Enter credit card details
    User->>CEPAS: Confirm payment ($100.00)

    CEPAS->>CEPAS: Process payment<br/>Generate confirmation: CONF-789<br/>Calculate hash: SHA-1(CONF-789 + 100.00 + SecurityKey)

    CEPAS-->>UI: 302 Redirect to MiEdWorkforce<br/>returnurl?c=0&m=Success&o=CONF-789&t=100.00<br/>&d=2026-01-15&z=AUTH-456&i=SESSION-67890<br/>&ct=VISA&hash=a1b2c3d4...

    UI->>PayApi: EXT GET /payments/confirm (public)

    PayApi->>PayApi: Validate hash<br/>Expected: SHA-1(CONF-789 + 100.00 + SecurityKey)<br/>Received: a1b2c3d4...<br/>Match: YES

    PayApi->>PayApi: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>PaidDate: 2026-01-15<br/>AuthorizationCode: AUTH-456<br/>CardType: VISA

    PayApi--)EventBus: PaymentCompleted
    Note over EventBus: Consumed by Credentialing (application status moves from Pending Payment to In Review)<br/>and Communications (sends the payment confirmation email to the user)

    PayApi-->>UI: 200 success page data
    UI-->>User: Display success page<br/>Confirmation #: CONF-789<br/>Amount: $100.00
```

**Key Decisions:**
- **PCI Compliance:** MiEdWorkforce never handles card data; all sensitive processing on CEPAS
- **Hash validation:** Required security check prevents spoofed confirmations
- **Event-driven:** Credential workflow advancement triggered by PaymentCompleted event

**State Changes:**
- PaymentTransaction status: `Pending` -> `Paid`
- Credential application status: `Pending Payment` -> `In Review`

**Events Published:**
- `PaymentCompleted` - Fires after hash validation success

**Error Scenarios:**
- Hash validation fails -> Transaction marked `Failed`, security alert raised
- CEPAS returns error code -> See "Individual Payment Flow (Payment Failed)"

---

## Individual Payment Flow (Browser Closed - Missing Payment)

**What:** User completes payment on CEPAS but closes browser before redirect; posting file reconciliation catches missing payment next morning  
**When:** Nightly batch job processes CEPAS posting file  
**Who:** System (automated reconciliation)

```mermaid
---
title: Payments - Individual Payment - Missing Payment Recovery
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
        participant FTS as File Transfer Service
    end

    Note over PayApi,CEPAS: User completed payment on CEPAS ($100.00, CONF-789) but closed the browser<br/>and never returned to MiEdWorkforce.<br/>PaymentTransaction remains Status: Pending (confirmation never received)

    Note over CEPAS,FTS: Next morning at 3:00 AM EST, CEPAS posts the daily file via SFTP<br/>Filename: MiEdWorkforce_260115.txt

    PayApi->>FTS: OUT SFTP GET MiEdWorkforce_260115.txt (nightly reconciliation job)
    FTS-->>PayApi: MiEdWorkforce_260115.txt

    PayApi->>PayApi: Parse fixed-width file<br/>AH (header), AD (transactions), AF (footer)

    Note over PayApi: AD Record found:<br/>ConfirmationNumber: CONF-789<br/>Amount: $100.00<br/>Reference: APP-123<br/>Command Code: 1 (Payment)

    PayApi->>PayApi: Query transaction by APP-123<br/>Found: Status=Pending

    PayApi->>PayApi: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>Source: Posting File Reconciliation

    PayApi--)EventBus: MissingPaymentDetected
    PayApi--)EventBus: PaymentCompleted
    Note over EventBus: Consumed by Credentialing (application status moves from Pending Payment to In Review)<br/>and Communications (sends the payment confirmation email to the user, delayed)

    PayApi->>PayApi: Log: Missing payment recovered<br/>Transaction: APP-123<br/>Confirmation: CONF-789
```

**Key Decisions:**
- **Posting file is source of truth:** Catches all payments that CEPAS confirmed, even if redirect missed
- **Auto-update only Pending:** Failed or Refunded transactions not overwritten
- **Monitoring:** MissingPaymentDetected event tracks reliability of redirect flow

**State Changes:**
- PaymentTransaction status: `Pending` -> `Paid` (via batch job)
- Credential application status: `Pending Payment` -> `In Review` (may be delayed 1 day)

**Events Published:**
- `MissingPaymentDetected` - Fires when posting file payment not found in real-time flow
- `PaymentCompleted` - Standard payment completion event (triggers credential workflow)

**Business Impact:**
- User experience: Payment confirmation email delayed up to 24 hours
- Credential processing: Delayed by up to 1 business day
- Revenue protection: No payments lost due to browser issues

---

## Individual Payment Flow (Payment Failed)

**What:** Payment fails on CEPAS (e.g., declined card, insufficient funds); user can retry  
**When:** CEPAS returns error code via redirect  
**Who:** User (after payment decline). The confirmation step is a public endpoint (no permission; hash validated).

```mermaid
---
title: Payments - Individual Payment - Payment Failed
---
sequenceDiagram
    actor User
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
    end

    Note over User,CEPAS: User started payment as in "Individual Payment Flow (Happy Path)"<br/>and enters card details on CEPAS

    CEPAS->>CEPAS: Payment DECLINED<br/>Reason: Insufficient Funds<br/>Return Code: 101

    CEPAS-->>UI: 302 Redirect to MiEdWorkforce<br/>returnurl?c=101&m=Declined - Insufficient Funds<br/>&t=100.00&i=SESSION-67890&hash=...

    UI->>PayApi: EXT GET /payments/confirm (public)

    PayApi->>PayApi: Validate hash (partial validation)
    PayApi->>PayApi: Update PaymentTransaction<br/>Status: Pending -> Failed<br/>ReturnCode: 101<br/>ReturnMessage: Declined - Insufficient Funds<br/>FailedDate: 2026-01-15

    PayApi--)EventBus: PaymentFailed
    Note over EventBus: Consumed by Credentialing (application remains Status: Pending Payment, not advanced)<br/>and Communications (sends the payment failed notification<br/>Suggestion: Try different payment method)

    PayApi-->>UI: 200 error page data
    UI-->>User: Display error page<br/>"Payment declined. Try again with<br/>different card."<br/>[Retry Payment] button shown
```

**Key Decisions:**
- **Preserve failed transaction:** Audit trail maintained; retry creates new transaction
- **User-friendly error:** Generic message to user; detailed error logged for admin review
- **Unlimited retries:** User can attempt payment multiple times

**State Changes:**
- PaymentTransaction status: `Pending` -> `Failed`
- Credential application status: Remains `Pending Payment`

**Events Published:**
- `PaymentFailed` - Fires when CEPAS returns non-zero return code

**Error Scenarios:**
- Common return codes: 101 (Declined), 102 (Insufficient Funds), 103 (Invalid Card), 104 (Expired Card)

---

## Bulk Payment Processing

**What:** District staff selects multiple credential applications and processes single payment for total fees  
**When:** Business user (district staff) needs to pay fees for multiple applications  
**Who:** Business User with `payments.transaction.initiate.bulk` permission. Permission: `payments.transaction.initiate-bulk` (scoped to the caller's organization: District, ISD, or System-wide). The confirmation step is a public endpoint (no permission; hash validated).

```mermaid
---
title: Payments - Bulk Payment Processing
---
sequenceDiagram
    actor DistrictStaff as District Staff
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant CredApi as Credentialing API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
    end

    DistrictStaff->>UI: Navigate to bulk payment page
    UI->>PayApi: APP GET /applications/pending-payment

    PayApi->>CredApi: SVC GET /applications
    CredApi-->>PayApi: List of 10 applications<br/>App1: $50, App2: $75, App3-10: $50 each

    PayApi-->>UI: Applications list
    UI-->>DistrictStaff: Display table with checkboxes<br/>Total available: 10 applications

    DistrictStaff->>UI: Select applications 1-10
    DistrictStaff->>UI: Click "Pay Fees" ($525 total)

    UI->>PayApi: APP POST /payments/initiate-bulk

    PayApi->>PayApi: Create BulkPaymentTransaction<br/>Total: $525 ($50×9 + $75×1)<br/>Reference: BULK-789|APP-1,APP-2,...,APP-10<br/>Create 10 child PaymentTransactions<br/>Each linked to bulk parent<br/>All Status: Pending

    PayApi->>PayApi: Encrypt payload: amount=$525
    PayApi-->>UI: 200 cepasRedirectUrl
    UI->>CEPAS: OUT Browser redirect to CEPAS

    Note over DistrictStaff,CEPAS: District staff reviews payment summary ($525), enters payment details, and confirms on CEPAS<br/>CEPAS processes payment (Confirmation: CONF-999, Amount: $525.00)

    CEPAS-->>UI: 302 Redirect to MiEdWorkforce<br/>returnurl?c=0&o=CONF-999&t=525.00&...

    UI->>PayApi: EXT GET /payments/confirm (public)

    PayApi->>PayApi: Validate hash<br/>Validate amount: $525 = $525: YES
    PayApi->>PayApi: Update BulkPaymentTransaction: Paid<br/>Update all 10 child transactions: Paid<br/>ConfirmationNumber: CONF-999

    PayApi--)EventBus: BulkPaymentCompleted
    Note over EventBus: Payload: applicationIds: [APP-1...APP-10]<br/>Consumed by Credentialing (all 10 applications move from Pending Payment to In Review)<br/>and Communications (sends the bulk payment confirmation email with the itemized list)

    PayApi-->>UI: 200 success page data
    UI-->>DistrictStaff: Display success page<br/>"10 applications paid: $525"<br/>Confirmation: CONF-999
```

**Key Decisions:**
- **Atomic success/failure:** All applications paid or none; no partial payments
- **Amount validation:** Total must equal sum of individual fees
- **Scope enforcement:** User can only select applications within authorized district/ISD; the calling domain validates application eligibility before Payments is invoked

**Permissions:**
- Requires: `payments.transaction.initiate-bulk` at the appropriate organizational scope (District, ISD, or System-wide)

**State Changes:**
- BulkPaymentTransaction status: `Pending` -> `Paid`
- All 10 child PaymentTransactions: `Pending` -> `Paid`
- All 10 credential applications: `Pending Payment` -> `In Review`

**Events Published:**
- `BulkPaymentCompleted` - Includes array of all application IDs

**Error Scenarios:**
- Amount mismatch -> All transactions remain Pending, discrepancy flagged
- Any application out of scope -> Reject entire bulk payment request

---

## Refund Request and Approval

**What:** Credential Admin requests refund for paid transaction; auto-approved if ≤$500 and processed via CEPAS API; refunds >$500 require Financial Admin approval before processing  
**When:** Applicant withdraws application or refund needed for other reason; high-value refund needed  
**Who:** Credential Administrator, Financial Administrator. Permission: `payments.transaction.view` (search, scoped to the caller's organization), `payments.refund.request` (submit, system-wide), `payments.refund.approve` (list pending and approve or reject, system-wide)

```mermaid
---
title: Payments - Refund Request and Approval
---
sequenceDiagram
    actor CredAdmin as Credential Admin
    actor FinAdmin as Financial Admin
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
    end

    CredAdmin->>UI: Search for transaction
    UI->>PayApi: APP GET /payments/search
    PayApi-->>UI: Transaction found (Paid, $100.00)

    CredAdmin->>UI: Enter refund amount and reason, submit refund request
    UI->>PayApi: APP POST /refunds/request

    PayApi->>PayApi: Validate amount ≤ original<br/>Check threshold against $500

    alt Amount ≤ $500 (single approval, auto-approved)
        PayApi->>PayApi: Create RefundRequest<br/>Status: Approved (auto-approved)<br/>Amount: $100.00<br/>Reason: "Applicant withdrew application"

        PayApi->>CEPAS: OUT POST /api/v1/payment/cancel<br/>Headers: ApplicationID, SecurityKey, PaymentChannel<br/>Body: {ConfirmationNumber: CONF-789, RefundAmount: 100.00}
        CEPAS-->>PayApi: 200 OK<br/>{RefundConfirmationNumber: REFUND-123,<br/>RefundAmount: 100.00,<br/>ProcessedDate: 2026-01-16,<br/>Status: Completed}

        PayApi->>PayApi: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-123<br/>Update PaymentTransaction: Refunded<br/>Link to RefundRequest

        PayApi--)EventBus: RefundCompleted
        Note over EventBus: Consumed by Credentialing (business logic: may revert application to Draft)<br/>and Communications (sends refund processed emails to the Credential Admin and the applicant)

    else Amount > $500 (dual approval required)
        PayApi->>PayApi: Create RefundRequest<br/>Status: Requested (pending approval)<br/>Amount: $750.00<br/>RequestedBy: Credential Admin

        PayApi--)EventBus: RefundRequested
        Note over EventBus: Consumed by Communications (sends the refund approval needed email to the Financial Admin)

        Note over FinAdmin: Financial Admin reviews request

        FinAdmin->>UI: Navigate to refund approvals, review request details
        UI->>PayApi: APP GET /refunds/pending-approval
        PayApi-->>UI: List of pending refund requests

        FinAdmin->>UI: Click "Approve" and enter approver notes: "Verified with applicant"
        UI->>PayApi: APP POST /refunds/{refundRequestId}/approve

        PayApi->>PayApi: Update RefundRequest<br/>Status: Requested -> Approved<br/>ApprovedBy: Financial Admin<br/>ApprovedAt: 2026-01-16

        PayApi--)EventBus: RefundApproved

        PayApi->>CEPAS: OUT POST /api/v1/payment/cancel<br/>Body: {ConfirmationNumber: CONF-789, RefundAmount: 750.00}
        CEPAS-->>PayApi: 200 OK<br/>{RefundConfirmationNumber: REFUND-456,<br/>RefundAmount: 750.00,<br/>ProcessedDate: 2026-01-16}

        PayApi->>PayApi: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-456<br/>Update PaymentTransaction: Refunded

        PayApi--)EventBus: RefundCompleted
        Note over EventBus: Consumed by Communications (sends refund processed emails to the Credential Admin,<br/>the Financial Admin, and the applicant)
    end
```

**Key Decisions:**
- **Auto-approval threshold:** Configurable; default $500
- **Dual approval threshold:** Configurable; default >$500
- **CEPAS API idempotency:** Safe to retry on failure (same confirmation number)
- **Atomic update:** Transaction status only changes after CEPAS confirms
- **Approval workflow:** Sequential (request -> approve -> process), not parallel
- **Rejection path:** Financial Admin can reject with reason using the same `payments.refund.approve` permission (not shown in diagram)

**Permissions:**
- Requires: `payments.refund.request` to submit the refund request (Credential Admin)
- Requires: `payments.refund.approve` to approve or reject (Financial Admin); covers both actions

**State Changes:**
- RefundRequest status: `Requested` -> `Approved` (auto, at or below the threshold) -> `Completed`
- RefundRequest status: `Requested` -> `Approved` -> `Completed` (dual approval)
- PaymentTransaction status: `Paid` -> `Refunded`

**Events Published:**
- `RefundRequested` - Fires when Credential Admin submits request (dual approval path)
- `RefundApproved` - Fires when Financial Admin approves (dual approval path)
- `RefundCompleted` - Fires after CEPAS confirms refund processed

**Error Scenarios:**
- CEPAS API failure -> RefundRequest marked Failed, admin alerted for retry
- Amount exceeds original -> Validation error before CEPAS call

---

## Daily Posting File Reconciliation

**What:** Nightly batch job retrieves CEPAS posting file, matches transactions, updates missing payments, flags discrepancies  
**When:** Daily at 3:00 AM EST (automated)  
**Who:** System (ReconciliationJob). Permission: `payments.reconciliation.run` (system-wide; manual trigger only, the automated nightly job requires none)  
**See also:** "Reconcile One Posting Record" (handling of each posting file record)

```mermaid
---
title: Payments - Reconciliation - Reconcile Posting File
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant FTS as File Transfer Service
    end

    Note over PayApi: Nightly reconciliation job<br/>Scheduled: Daily 3:00 AM EST

    PayApi->>PayApi: Create ReconciliationBatch<br/>Status: Processing<br/>FileDate: 2026-01-15

    PayApi->>FTS: OUT SFTP GET MiEdWorkforce_260115.txt
    FTS-->>PayApi: Posting file (fixed-width text)

    PayApi->>PayApi: Parse file<br/>AH (header): Validate site, agency, application<br/>AD (transactions): Extract records<br/>AF (footer): Validate total records, total amount

    Note over PayApi: Header/Footer validation: PASS<br/>Expected records: 247<br/>Expected total: $24,750.00

    loop For each AD transaction record (247 records)
        PayApi->>PayApi: Reconcile one posting record<br/>(see "Reconcile One Posting Record")
    end

    PayApi->>PayApi: Update ReconciliationBatch<br/>Status: Processing -> Completed<br/>Summary:<br/>- Total Records: 247<br/>- Matched: 240<br/>- Missing Payments Recovered: 5<br/>- Discrepancies: 2

    PayApi--)EventBus: ReconciliationBatchCompleted
    Note over EventBus: Consumed by monitoring (when discrepancies > 0, raises an alert and notifies the Payment Admin:<br/>"2 payment discrepancies require review")
```

**Key Decisions:**
- **Source of truth:** Posting file is authoritative for payment status

**Permissions:**
- Requires: `payments.reconciliation.run` to manually trigger (automated nightly job requires no user permission)
- Requires: `payments.reconciliation.view` to view batch results
- Requires: `payments.reconciliation.discrepancy-view` to see flagged discrepancies

**State Changes:**
- ReconciliationBatch status: `Processing` -> `Completed`

**Events Published:**
- `ReconciliationBatchCompleted` - At batch completion with summary

**Monitoring:**
- Success rate: Target >95% auto-matched
- Alert if discrepancies >5% of total records
- Alert if batch fails to run

---

## Reconcile One Posting Record

**What:** For each AD record in the posting file, the reconciliation job matches the record to a PaymentTransaction by command code, recovers missing payments, and flags discrepancies  
**When:** Once per AD record while the nightly batch runs  
**Who:** System (ReconciliationJob)  
**See also:** "Daily Posting File Reconciliation" (batch retrieval, parsing and summary)

```mermaid
---
title: Payments - Reconciliation - Reconcile One Posting Record
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end

    PayApi->>PayApi: Extract fields:<br/>- Confirmation Number<br/>- Payment Amount<br/>- Custom Reference (APP-123)<br/>- Command Code

    PayApi->>PayApi: Command Code = 1 (Payment)<br/>Query by reference: APP-123

    alt Transaction found, Status=Pending, amounts equal
        PayApi->>PayApi: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>Source: Posting File
        PayApi--)EventBus: MissingPaymentDetected
        PayApi--)EventBus: PaymentCompleted
        Note over EventBus: Consumed by Credentialing (application status moves from Pending Payment to In Review)<br/>and Communications (sends the payment confirmation email to the user)
    else Transaction found, Amount mismatch
        PayApi->>PayApi: Create ReconciliationDiscrepancy<br/>Type: Amount Mismatch<br/>Expected: $100, Actual: $105
        PayApi--)EventBus: PaymentDiscrepancyDetected
    end
```

**Key Decisions:**
- **Auto-update restrictions:** Only `Pending` transactions updated; `Failed`/`Refunded` left unchanged
- **Discrepancy handling:** Flagged for manual review, not auto-resolved

**Command Code Handling:**

| Command Code | Condition | Handling |
|---|---|---|
| 1 (Payment) | Transaction found, Status=Pending | Update to Paid (Source: Posting File), publish `MissingPaymentDetected` and `PaymentCompleted`, log "Missing payment recovered" (shown in diagram) |
| 1 (Payment) | Transaction found, Status=Paid | Match confirmed, no action needed |
| 1 (Payment) | Transaction found, Amount mismatch | Create ReconciliationDiscrepancy (Type: Amount Mismatch), publish `PaymentDiscrepancyDetected` (shown in diagram) |
| 1 (Payment) | Transaction not found | Create ReconciliationDiscrepancy (Type: Orphan Payment; payment in CEPAS but no application), publish `PaymentDiscrepancyDetected` |
| 5 (Refund) | Any | Match refund confirmation number, verify refund recorded |
| 6, 9, 10, 15, 16 (Void, Chargeback, Chargeback Reversal, ACH Credit, Processor Void) | Any | Record as ReconciliationRecord with match_result = Discrepancy; not auto-resolved, no defined handling for these codes yet; flagged for manual classification |

Command codes: 1=Payment, 5=Refund, 6=Void, 9=Chargeback, 10=ChargebackReversal, 14=Partial, 15=ACHCredit, 16=ProcessorVoid.

**State Changes:**
- PaymentTransaction status: `Pending` -> `Paid` (for missing payments only)

**Events Published:**
- `MissingPaymentDetected` - Per recovered payment
- `PaymentCompleted` - Per recovered payment (triggers credential workflow)
- `PaymentDiscrepancyDetected` - Per unmatched or mismatched record

---

## CEPAS Service Unavailable

**What:** User attempts payment when CEPAS is down; system displays error and allows retry later  
**When:** CEPAS health check fails  
**Who:** Any user attempting payment. Permission: `payments.transaction.initiate` (implicit baseline; the user's own application)

```mermaid
---
title: Payments - Individual Payment - CEPAS Unavailable
---
sequenceDiagram
    actor User
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
    end
    box External
        participant CEPAS
    end

    User->>UI: Click "Pay Fee"
    UI->>PayApi: APP POST /payments/initiate

    PayApi->>CEPAS: OUT HEAD https://epay.michigan.gov/payment<br/>Circuit breaker health check, Timeout: 5 seconds

    alt CEPAS unavailable
        CEPAS-->>PayApi: Timeout (no response)
        PayApi->>PayApi: Record failure<br/>Failure count: 3/3<br/>Circuit state: CLOSED -> OPEN

        Note over PayApi: Alert: CEPAS circuit breaker opened<br/>Timestamp: 2026-01-15 10:30:00<br/>Circuit remains OPEN for 5 minutes (cooldown)

        PayApi-->>UI: 503 Service Unavailable<br/>Error: "Payment system temporarily unavailable"

        UI-->>User: Display error message<br/>"The payment system is temporarily unavailable.<br/>Please try again later.<br/>Your application has been submitted."

    else CEPAS available (after cooldown)
        CEPAS-->>PayApi: 200 OK
        PayApi->>PayApi: Health check success<br/>Circuit state: OPEN -> HALF-OPEN -> CLOSED<br/>Create PaymentTransaction<br/>Encrypt payload
        PayApi-->>UI: 200 cepasRedirectUrl
        UI->>CEPAS: OUT Browser redirect to CEPAS

        Note over User,CEPAS: After the 5-minute cooldown the user clicks "Pay Fee" (retry)<br/>Payment proceeds normally
    end
```

**Key Decisions:**
- **Circuit breaker pattern:** Prevents cascading failures and reduces load on struggling CEPAS
- **Health check before redirect:** Fail fast with clear error rather than broken redirect
- **Cooldown period:** 5 minutes (configurable) before retry attempts

**Circuit Breaker States:**
- `CLOSED` - Normal operation, all requests pass through
- `OPEN` - Failures exceeded threshold, all requests blocked
- `HALF-OPEN` - Testing if service recovered, limited requests allowed

**Monitoring:**
- Alert when circuit opens
- Track CEPAS availability percentage
- Monitor user impact (failed payment attempts)

---

## Payment Retry After Failure

**What:** User retries payment after previous attempt failed (declined card, timeout, etc.)  
**When:** User clicks retry button on failed payment error page  
**Who:** Any user with failed payment transaction. Permission: `payments.transaction.initiate` (implicit baseline; the user's own application)

```mermaid
---
title: Payments - Individual Payment - Retry After Failure
---
sequenceDiagram
    actor User
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
    end

    Note over User,PayApi: Previous payment attempt failed<br/>Transaction T1: Status=Failed

    User->>UI: View application (Status: Pending Payment)
    UI-->>User: Display "Pay Fee" link<br/>(Link always available for Pending Payment)

    User->>UI: Click "Pay Fee" (retry)
    UI->>PayApi: APP POST /payments/initiate

    PayApi->>PayApi: Check for existing Paid transaction<br/>Query: applicationId=APP-123, Status=Paid

    alt Existing Paid transaction found
        PayApi-->>UI: 409 Conflict<br/>Error: "Payment already completed"
        UI-->>User: Display: "This application has already been paid."

    else No Paid transaction (only Failed transaction exists)
        PayApi->>PayApi: Create NEW PaymentTransaction (T2)<br/>Status: Pending<br/>Amount: $100.00<br/>Reference: APP-123<br/>Encrypt payload

        Note over PayApi: Original failed transaction (T1)<br/>preserved in audit trail

        PayApi-->>UI: 200 cepasRedirectUrl

        Note over User,PayApi: User enters DIFFERENT card details on CEPAS and the payment completes<br/>as in "Individual Payment Flow (Happy Path)"<br/>Transaction T1: Status=Failed (preserved)<br/>Transaction T2: Status=Paid (successful retry)
    end
```

**Key Decisions:**
- **Each retry creates new transaction:** Failed transactions preserved for audit trail
- **Prevent double payment:** Check for existing Paid transaction before creating new one
- **Unlimited retries:** User can attempt payment as many times as needed

**State Changes:**
- New PaymentTransaction (T2) created: `Pending` -> `Paid`
- Original failed transaction (T1): Remains `Failed`

**Audit Trail:**
- All payment attempts linked to same application ID
- Timestamp and reason for failure preserved
- Full history available for support/troubleshooting

---

## Bulk Refund Processing

**What:** Credential Admin selects multiple paid transactions for bulk refund  
**When:** Multiple applications need refunds (e.g., district error, program cancellation)  
**Who:** Credential Administrator (with Financial Admin approval for high values). Permission: `payments.transaction.view` (search, scoped to the caller's organization), `payments.refund.request-bulk` (submit, system-wide), `payments.refund.approve` (only if the threshold is exceeded, system-wide)

```mermaid
---
title: Payments - Bulk Refund Processing
---
sequenceDiagram
    actor CredAdmin as Credential Admin
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end
    box External
        participant CEPAS
    end

    CredAdmin->>UI: Navigate to bulk refund page
    UI->>PayApi: APP GET /payments/search
    PayApi-->>UI: List of paid transactions

    UI-->>CredAdmin: Display paid transactions table

    CredAdmin->>UI: Select 5 transactions:<br/>CONF-100 ($50), CONF-101 ($50),<br/>CONF-102 ($50), CONF-103 ($75),<br/>CONF-104 ($50)

    CredAdmin->>UI: For each, enter refund amount:<br/>All full refunds (total: $275)

    CredAdmin->>UI: Enter reason: "District processing error"
    CredAdmin->>UI: Click "Submit Bulk Refund"

    UI->>PayApi: APP POST /refunds/bulk-request

    PayApi->>PayApi: Validate all amounts ≤ original: YES<br/>Check threshold: $275 ≤ $500: YES<br/>Single approval (auto-approved)<br/>Create 5 RefundRequest records<br/>All Status: Approved (auto)<br/>LinkedBulkId: BULK-REFUND-789

    loop For each refund (5 requests)
        PayApi->>CEPAS: OUT POST /api/v1/payment/cancel<br/>ConfirmationNumber: CONF-100<br/>RefundAmount: 50.00

        CEPAS-->>PayApi: 200 OK<br/>RefundConfirmationNumber: REFUND-200

        PayApi->>PayApi: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-200<br/>Update PaymentTransaction: Refunded
    end

    PayApi--)EventBus: BulkRefundCompleted
    Note over EventBus: Payload: refundIds: [5 IDs], totalAmount: $275<br/>Consumed by Communications (sends the bulk refund processed email: 5 refunds, $275 total)

    PayApi-->>UI: 200 OK<br/>Summary: 5 refunds processed

    UI-->>CredAdmin: Display success:<br/>"5 refunds processed successfully"<br/>Confirmation numbers: REFUND-200...REFUND-204
```

**Key Decisions:**
- **Sequential processing:** Refunds processed one-by-one via CEPAS API (not atomic batch)
- **Partial success handling:** If one fails, others continue; failure logged separately
- **Threshold calculation:** Total refund amount determines approval requirements

**Permissions:**
- Requires: `payments.refund.request-bulk` to submit the bulk refund request (Credential Admin)
- Requires: `payments.refund.approve` to approve or reject if threshold exceeded (Financial Admin)

**State Changes:**
- 5 RefundRequest records: `Requested` -> `Approved` -> `Completed`
- 5 PaymentTransaction records: `Paid` -> `Refunded`

**Events Published:**
- `BulkRefundCompleted` - Single event with summary of all refunds

**Error Scenarios:**
- If any CEPAS API call fails, that refund marked Failed, others continue
- Summary shows: 4 successful, 1 failed

---

## Reconciliation Discrepancy Investigation

**What:** Payment Admin investigates and manually resolves posting file discrepancy  
**When:** Reconciliation batch detects unmatched or mismatched transaction  
**Who:** Payment Administrator. Permission: `payments.reconciliation.discrepancy-view` (view), `payments.reconciliation.discrepancy-resolve` (resolve); both system-wide

```mermaid
---
title: Payments - Reconciliation Discrepancy Investigation
---
sequenceDiagram
    actor PaymentAdmin as Payment Admin
    box Browser
        participant UI
    end
    box MiEdWorkforce (AKS)
        participant PayApi as Payments API
        participant EventBus as Event Bus
    end

    Note over PayApi: Reconciliation batch detected discrepancy<br/>Type: Amount Mismatch<br/>Expected: $100, Actual: $105

    PaymentAdmin->>UI: Navigate to discrepancies page
    UI->>PayApi: APP GET /reconciliation/discrepancies
    PayApi-->>UI: List of unresolved discrepancies

    UI-->>PaymentAdmin: Display discrepancies table<br/>Discrepancy #1:<br/>- Type: Amount Mismatch<br/>- Application: APP-123<br/>- Expected: $100.00<br/>- CEPAS Posted: $105.00<br/>- Confirmation: CONF-789

    PaymentAdmin->>UI: Click discrepancy to investigate
    UI->>PayApi: APP GET /reconciliation/discrepancies/{discrepancyId}
    PayApi-->>UI: Discrepancy details + related transaction

    UI-->>PaymentAdmin: Display full details:<br/>- PaymentTransaction: APP-123, $100<br/>- Posting file record: CONF-789, $105<br/>- Date: 2026-01-15

    Note over PaymentAdmin: Investigation:<br/>1. Check CEPAS portal for transaction<br/>2. Contact CEPAS support (Sean Strom)<br/>3. Determine: $5 processing fee charged<br/>   (Not communicated in advance)

    PaymentAdmin->>UI: Enter resolution notes:<br/>"Contacted CEPAS support. $5 processing fee<br/>added by CEPAS. Confirmed with Sean Strom<br/>that this is expected. Updating fee schedule."

    PaymentAdmin->>UI: Select resolution action:<br/>"Accept CEPAS amount as correct"

    PaymentAdmin->>UI: Click "Resolve Discrepancy"

    UI->>PayApi: APP POST /reconciliation/discrepancies/{discrepancyId}/resolve

    PayApi->>PayApi: Update PaymentTransaction<br/>Amount: $100 -> $105<br/>Notes: "Adjusted per CEPAS posting file"

    PayApi->>PayApi: Update ReconciliationDiscrepancy<br/>Status: Unresolved -> Resolved<br/>ResolvedBy: Payment Admin<br/>ResolvedAt: 2026-01-16<br/>ResolutionNotes: "..."

    PayApi--)EventBus: DiscrepancyResolved

    UI-->>PaymentAdmin: Display: "Discrepancy resolved"

    Note over PaymentAdmin: Follow-up action:<br/>Update fee schedule to include<br/>$5 CEPAS processing fee
```

**Key Decisions:**
- **Manual resolution required:** No auto-resolution of discrepancies
- **Audit trail:** All resolution actions logged with notes
- **Resolution options:** Accept CEPAS, Reject (mark transaction Failed), Create adjustment

**Permissions:**
- Requires: `payments.reconciliation.discrepancy-view` to view discrepancies
- Requires: `payments.reconciliation.discrepancy-resolve` to mark as resolved; resolution notes are mandatory and audit logged

**Resolution Types:**
- **Accept CEPAS Amount:** Update MiEdWorkforce transaction to match posting file
- **Reject CEPAS Record:** Mark transaction as Failed, investigate further
- **Contact Applicant:** Discrepancy requires external verification
- **Adjustment Required:** Create manual adjustment transaction

**State Changes:**
- ReconciliationDiscrepancy status: `Unresolved` -> `Resolved`
- PaymentTransaction: May be updated based on resolution type

**Events Published:**
- `DiscrepancyResolved` - Fires when admin marks discrepancy resolved
