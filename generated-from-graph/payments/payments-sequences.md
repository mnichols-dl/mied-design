# Payments - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Payments platform capability.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

---

## Individual Payment Flow (Happy Path)

**What:** User initiates payment for credential application fee, completes payment on CEPAS, and returns to MiEdWorkforce with confirmation  
**When:** User clicks "Pay Fee" link on pending credential application  
**Who:** Citizen User, Business User, or State Admin/Staff

```mermaid
---
title: Payments - Individual Payment Flow (Happy Path)
---
sequenceDiagram
    actor User
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant Credentials
    participant EventBus
    participant Comms as Communications

    User->>UI: Click "Pay Fee" link
    UI->>Payments: GET /payments/initiate?applicationId=APP-123

    Payments->>Payments: Create PaymentTransaction<br/>Status: Pending<br/>Amount: $100.00 (from credential type)<br/>Reference: APP-123
    Payments->>Payments: Encrypt payload with AES GCM<br/>(amount, ref, id, returnurl)

    Note over Payments: Encrypted format:<br/>aen=<key_id>|<12B_nonce><ciphertext+tag>

    Payments-->>UI: 302 Redirect to CEPAS<br/>URL: https://epay.michigan.gov/payment?aen=...
    UI-->>User: Browser redirects to CEPAS

    User->>CEPAS: Enter credit card details
    User->>CEPAS: Confirm payment ($100.00)

    CEPAS->>CEPAS: Process payment<br/>Generate confirmation: CONF-789<br/>Calculate hash: SHA-1(CONF-789 + 100.00 + SecurityKey)

    CEPAS-->>User: 302 Redirect to MiEdWorkforce<br/>returnurl?c=0&m=Success&o=CONF-789&t=100.00<br/>&d=2026-01-15&z=AUTH-456&i=SESSION-67890<br/>&ct=VISA&hash=a1b2c3d4...

    User->>UI: Browser redirects back
    UI->>Payments: GET /payments/confirm?c=0&o=CONF-789&...

    Payments->>Payments: Validate hash<br/>Expected: SHA-1(CONF-789 + 100.00 + SecurityKey)<br/>Received: a1b2c3d4...<br/>Match: YES

    Payments->>Payments: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>PaidDate: 2026-01-15<br/>AuthorizationCode: AUTH-456<br/>CardType: VISA

    Payments--)EventBus: PaymentCompleted
    EventBus--)Credentials: PaymentCompleted event
    Credentials->>Credentials: Update application<br/>Status: Pending Payment -> In Review

    EventBus--)Comms: PaymentCompleted event
    Comms--)User: Email: Payment confirmation

    Payments-->>UI: 200 OK (success page data)
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
title: Payments - Individual Payment Flow (Browser Closed - Missing Payment)
---
sequenceDiagram
    actor User
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant FTS as File Transfer Service
    participant ReconciliationJob
    participant EventBus
    participant Credentials
    participant Comms as Communications

    Note over User,CEPAS: User initiates payment, completes on CEPAS

    User->>UI: Click "Pay Fee"
    UI->>Payments: Initiate payment
    Payments-->>User: Redirect to CEPAS

    User->>CEPAS: Complete payment ($100.00)
    CEPAS->>CEPAS: Payment successful<br/>Generate CONF-789
    CEPAS-->>User: 302 Redirect to MiEdWorkforce

    User->>User: CLOSE BROWSER<br/>(Never returns to MiEdWorkforce)

    Note over Payments: PaymentTransaction remains<br/>Status: Pending<br/>(Confirmation never received)

    Note over CEPAS,FTS: Next morning at 3:00 AM EST

    CEPAS->>FTS: Post daily file via SFTP<br/>Filename: MiEdWorkforce_260115.txt

    ReconciliationJob->>FTS: Retrieve posting file (SFTP)
    FTS-->>ReconciliationJob: MiEdWorkforce_260115.txt

    ReconciliationJob->>ReconciliationJob: Parse fixed-width file<br/>AH (header), AD (transactions), AF (footer)

    Note over ReconciliationJob: AD Record found:<br/>ConfirmationNumber: CONF-789<br/>Amount: $100.00<br/>Reference: APP-123<br/>Command Code: 1 (Payment)

    ReconciliationJob->>Payments: Query transaction by APP-123
    Payments-->>ReconciliationJob: Found: Status=Pending

    ReconciliationJob->>Payments: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>Source: Posting File Reconciliation

    Payments--)EventBus: MissingPaymentDetected
    Payments--)EventBus: PaymentCompleted

    EventBus--)Credentials: PaymentCompleted event
    Credentials->>Credentials: Update application<br/>Status: Pending Payment -> In Review

    EventBus--)Comms: PaymentCompleted event
    Comms--)User: Email: Payment confirmation (delayed)

    ReconciliationJob->>ReconciliationJob: Log: Missing payment recovered<br/>Transaction: APP-123<br/>Confirmation: CONF-789
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

**Error Scenarios:**
None distinct from the reconciliation batch job's own error handling (see pay:BatchJob_PostingFileReconciliation).

---

## Individual Payment Flow (Payment Failed)

**What:** Payment fails on CEPAS (e.g., declined card, insufficient funds); user can retry  
**When:** CEPAS returns error code via redirect  
**Who:** User (after payment decline)

```mermaid
---
title: Payments - Individual Payment Flow (Payment Failed)
---
sequenceDiagram
    actor User
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant Credentials
    participant EventBus
    participant Comms as Communications

    User->>UI: Click "Pay Fee"
    UI->>Payments: Initiate payment
    Payments-->>User: Redirect to CEPAS

    User->>CEPAS: Enter card details
    User->>CEPAS: Confirm payment

    CEPAS->>CEPAS: Payment DECLINED<br/>Reason: Insufficient Funds<br/>Return Code: 101

    CEPAS-->>User: 302 Redirect to MiEdWorkforce<br/>returnurl?c=101&m=Declined - Insufficient Funds<br/>&t=100.00&i=SESSION-67890&hash=...

    User->>UI: Browser redirects back
    UI->>Payments: GET /payments/confirm?c=101&m=Declined...

    Payments->>Payments: Validate hash (partial validation)
    Payments->>Payments: Update PaymentTransaction<br/>Status: Pending -> Failed<br/>ReturnCode: 101<br/>ReturnMessage: Declined - Insufficient Funds<br/>FailedDate: 2026-01-15

    Payments--)EventBus: PaymentFailed
    EventBus--)Credentials: PaymentFailed event

    Note over Credentials: Application remains<br/>Status: Pending Payment<br/>(Not advanced)

    EventBus--)Comms: PaymentFailed event
    Comms--)User: Email: Payment failed notification<br/>Suggestion: Try different payment method

    Payments-->>UI: 200 OK (error page data)
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
**Who:** Business User with `payments.transaction.initiate.bulk` permission

```mermaid
---
title: Payments - Bulk Payment Processing
---
sequenceDiagram
    actor DistrictStaff as District Staff
    participant UI as MiEdWorkforce UI
    participant Payments
    participant Credentials
    participant CEPAS
    participant EventBus
    participant Comms as Communications

    DistrictStaff->>UI: Navigate to bulk payment page
    UI->>Payments: GET /applications/pending-payment?districtId=12345

    Payments->>Credentials: Query applications<br/>Filter: District 12345, Status=Pending Payment
    Credentials-->>Payments: List of 10 applications<br/>App1: $50, App2: $75, App3-10: $50 each

    Payments-->>UI: Applications list
    UI-->>DistrictStaff: Display table with checkboxes<br/>Total available: 10 applications

    DistrictStaff->>UI: Select applications 1-10
    DistrictStaff->>UI: Click "Pay Fees" ($525 total)

    UI->>Payments: POST /payments/initiate-bulk<br/>applicationIds: [APP-1, APP-2, ..., APP-10]

    Payments->>Payments: Validate user permission<br/>Scope: District 12345<br/>All apps in scope: YES

    Payments->>Payments: Create BulkPaymentTransaction<br/>Total: $525 ($50×9 + $75×1)<br/>Reference: BULK-789|APP-1,APP-2,...,APP-10

    Payments->>Payments: Create 10 child PaymentTransactions<br/>Each linked to bulk parent<br/>All Status: Pending

    Payments->>Payments: Encrypt payload: amount=$525
    Payments-->>UI: 302 Redirect to CEPAS
    UI-->>DistrictStaff: Browser redirects to CEPAS

    DistrictStaff->>CEPAS: Review payment summary ($525)
    DistrictStaff->>CEPAS: Enter payment details
    DistrictStaff->>CEPAS: Confirm payment

    CEPAS->>CEPAS: Process payment<br/>Confirmation: CONF-999<br/>Amount: $525.00

    CEPAS-->>DistrictStaff: Redirect to MiEdWorkforce<br/>returnurl?c=0&o=CONF-999&t=525.00&...

    DistrictStaff->>UI: Browser redirects back
    UI->>Payments: GET /payments/confirm?c=0&o=CONF-999...

    Payments->>Payments: Validate hash
    Payments->>Payments: Validate amount: $525 = $525: YES

    Payments->>Payments: Update BulkPaymentTransaction: Paid<br/>Update all 10 child transactions: Paid<br/>ConfirmationNumber: CONF-999

    Payments--)EventBus: BulkPaymentCompleted<br/>Payload: applicationIds: [APP-1...APP-10]

    EventBus--)Credentials: BulkPaymentCompleted event
    Credentials->>Credentials: Update all 10 applications<br/>Status: Pending Payment -> In Review

    EventBus--)Comms: BulkPaymentCompleted event
    Comms--)DistrictStaff: Email: Bulk payment confirmation<br/>Itemized list of 10 applications

    Payments-->>UI: 200 OK (success page data)
    UI-->>DistrictStaff: Display success page<br/>"10 applications paid: $525"<br/>Confirmation: CONF-999
```

**Key Decisions:**
- **Atomic success/failure:** All applications paid or none; no partial payments
- **Amount validation:** Total must equal sum of individual fees
- **Scope enforcement:** User can only select applications within authorized district/ISD; the calling domain validates application eligibility before Payments is invoked

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

## Refund Request Processing (Single Approval ≤$500)

**What:** Credential Admin requests refund for paid transaction; auto-approved if ≤$500, processed via CEPAS API  
**When:** Applicant withdraws application or refund needed for other reason  
**Who:** Credential Administrator

```mermaid
---
title: Payments - Refund Request Processing (Single Approval ≤$500)
---
sequenceDiagram
    actor CredAdmin as Credential Admin
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant Credentials
    participant EventBus
    participant Comms as Communications

    CredAdmin->>UI: Search for transaction
    UI->>Payments: GET /payments/search?confirmationNumber=CONF-789
    Payments-->>UI: Transaction found (Paid, $100.00)
    UI-->>CredAdmin: Display transaction details

    CredAdmin->>UI: Click "Request Refund"
    UI-->>CredAdmin: Display refund form

    CredAdmin->>UI: Enter refund amount: $100.00
    CredAdmin->>UI: Enter reason: "Applicant withdrew application"
    CredAdmin->>UI: Submit refund request

    UI->>Payments: POST /refunds/request<br/>transactionId, amount: 100, reason: "..."

    Payments->>Payments: Validate amount <= original: YES ($100 <= $100)
    Payments->>Payments: Check threshold: $100 <= $500: YES<br/>Single approval path

    Payments->>Payments: Create RefundRequest<br/>Status: Approved (auto-approved)<br/>Amount: $100.00<br/>Reason: "Applicant withdrew application"

    Payments->>CEPAS: POST /api/v1/payment/cancel<br/>Headers: ApplicationID, SecurityKey, PaymentChannel<br/>Body: {ConfirmationNumber: CONF-789, RefundAmount: 100.00}

    CEPAS->>CEPAS: Process refund
    CEPAS-->>Payments: 200 OK<br/>{RefundConfirmationNumber: REFUND-123,<br/>RefundAmount: 100.00,<br/>ProcessedDate: 2026-01-16,<br/>Status: Completed}

    Payments->>Payments: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-123<br/>ProcessedDate: 2026-01-16

    Payments->>Payments: Update PaymentTransaction: Refunded<br/>Link to RefundRequest

    Payments--)EventBus: RefundCompleted
    EventBus--)Credentials: RefundCompleted event
    Credentials->>Credentials: Update application status<br/>(Business logic: May revert to Draft)

    EventBus--)Comms: RefundCompleted event
    Comms--)CredAdmin: Email: Refund processed notification
    Comms--)User: Email: Refund confirmation to applicant

    Payments-->>UI: 200 OK (success message)
    UI-->>CredAdmin: Display: "Refund processed: REFUND-123"
```

**Key Decisions:**
- **Auto-approval threshold:** Configurable; default $500
- **CEPAS API idempotency:** Safe to retry on failure (same confirmation number)
- **Atomic update:** Transaction status only changes after CEPAS confirms

**State Changes:**
- RefundRequest status: `Requested` -> `Approved` (auto) -> `Completed`
- PaymentTransaction status: `Paid` -> `Refunded`

**Events Published:**
- `RefundCompleted` - Fires after CEPAS confirms refund processed

**Error Scenarios:**
CEPAS API failure -> RefundRequest marked Failed, admin alerted for retry. Amount exceeds original -> Validation error before CEPAS API call.

---

## Refund Request Processing (Dual Approval >$500)

**What:** Credential Admin requests refund >$500; requires Financial Admin approval before processing  
**When:** High-value refund needed  
**Who:** Credential Administrator, Financial Administrator

```mermaid
---
title: Payments - Refund Request Processing (Dual Approval >$500)
---
sequenceDiagram
    actor CredAdmin as Credential Admin
    actor FinAdmin as Financial Admin
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant EventBus
    participant Comms as Communications

    CredAdmin->>UI: Request refund: $750
    CredAdmin->>UI: Reason: "Duplicate payment processing error"

    UI->>Payments: POST /refunds/request<br/>transactionId, amount: 750, reason: "..."

    Payments->>Payments: Validate amount <= original: YES
    Payments->>Payments: Check threshold: $750 > $500: YES<br/>Dual approval required

    Payments->>Payments: Create RefundRequest<br/>Status: Requested (pending approval)<br/>Amount: $750.00<br/>RequestedBy: Credential Admin

    Payments--)EventBus: RefundRequested
    EventBus--)Comms: RefundRequested event
    Comms--)FinAdmin: Email: Refund approval needed<br/>Amount: $750<br/>Reason: "Duplicate payment..."<br/>Link to approval page

    Payments-->>UI: 200 OK
    UI-->>CredAdmin: Message: "Refund request submitted.<br/>Requires Financial Admin approval."

    Note over FinAdmin: Financial Admin reviews request

    FinAdmin->>UI: Navigate to refund approvals
    UI->>Payments: GET /refunds/pending-approval
    Payments-->>UI: List of pending refund requests
    UI-->>FinAdmin: Display pending refunds

    FinAdmin->>UI: Review request details
    FinAdmin->>UI: Click "Approve"
    FinAdmin->>UI: Enter approver notes: "Verified with applicant"

    UI->>Payments: POST /refunds/{id}/approve<br/>approverNotes: "Verified with applicant"

    Payments->>Payments: Update RefundRequest<br/>Status: Requested -> Approved<br/>ApprovedBy: Financial Admin<br/>ApprovedAt: 2026-01-16

    Payments--)EventBus: RefundApproved

    Payments->>CEPAS: POST /api/v1/payment/cancel<br/>Body: {ConfirmationNumber: CONF-789, RefundAmount: 750.00}

    CEPAS->>CEPAS: Process refund
    CEPAS-->>Payments: 200 OK<br/>{RefundConfirmationNumber: REFUND-456,<br/>RefundAmount: 750.00,<br/>ProcessedDate: 2026-01-16}

    Payments->>Payments: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-456

    Payments->>Payments: Update PaymentTransaction: Refunded

    Payments--)EventBus: RefundCompleted
    EventBus--)Comms: RefundCompleted event
    Comms--)CredAdmin: Email: Refund approved and processed
    Comms--)FinAdmin: Email: Refund processed confirmation
    Comms--)User: Email: Refund confirmation to applicant

    Payments-->>UI: 200 OK
    UI-->>FinAdmin: Display: "Refund processed: REFUND-456"
```

**Key Decisions:**
- **Dual approval threshold:** Configurable; default >$500
- **Approval workflow:** Sequential (request -> approve -> process), not parallel
- **Rejection path:** Financial Admin can reject with reason using the same `payments.refund.approve` permission (not shown in diagram)

**State Changes:**
- RefundRequest status: `Requested` -> `Approved` -> `Completed`
- PaymentTransaction status: `Paid` -> `Refunded`

**Events Published:**
- `RefundRequested` - Fires when Credential Admin submits request
- `RefundApproved` - Fires when Financial Admin approves
- `RefundCompleted` - Fires after CEPAS confirms refund

**Error Scenarios:**
Not separately enumerated beyond the single-approval flow's error scenarios.

---

## Daily Posting File Reconciliation

**What:** Nightly batch job retrieves CEPAS posting file, matches transactions, updates missing payments, flags discrepancies  
**When:** Daily at 3:00 AM EST (automated)  
**Who:** System (ReconciliationJob)

```mermaid
---
title: Payments - Daily Posting File Reconciliation
---
sequenceDiagram
    participant ReconciliationJob
    participant FTS as File Transfer Service
    participant CEPAS
    participant Payments
    participant EventBus
    participant Monitoring

    Note over ReconciliationJob: Scheduled: Daily 3:00 AM EST

    ReconciliationJob->>ReconciliationJob: Start reconciliation batch
    ReconciliationJob->>Payments: Create ReconciliationBatch<br/>Status: Processing<br/>FileDate: 2026-01-15

    ReconciliationJob->>FTS: SFTP connect
    ReconciliationJob->>FTS: GET MiEdWorkforce_260115.txt
    FTS-->>ReconciliationJob: Posting file (fixed-width text)

    ReconciliationJob->>ReconciliationJob: Parse file<br/>AH (header): Validate site, agency, application<br/>AD (transactions): Extract records<br/>AF (footer): Validate total records, total amount

    Note over ReconciliationJob: Header/Footer validation: PASS<br/>Expected records: 247<br/>Expected total: $24,750.00

    loop For each AD transaction record (247 records)
        ReconciliationJob->>ReconciliationJob: Extract fields:<br/>- Confirmation Number<br/>- Payment Amount<br/>- Custom Reference (APP-123)<br/>- Command Code (1=Payment, 5=Refund, 6=Void, 9=Chargeback, 10=ChargebackReversal, 14=Partial, 15=ACHCredit, 16=ProcessorVoid)

        alt Command Code = 1 (Payment)
            ReconciliationJob->>Payments: Query by reference: APP-123

            alt Transaction found, Status=Pending
                Payments->>Payments: Update PaymentTransaction<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-789<br/>Source: Posting File
                Payments--)EventBus: MissingPaymentDetected
                Payments--)EventBus: PaymentCompleted
                ReconciliationJob->>ReconciliationJob: Log: Missing payment recovered
            else Transaction found, Status=Paid
                ReconciliationJob->>ReconciliationJob: Match confirmed<br/>No action needed
            else Transaction found, Amount mismatch
                ReconciliationJob->>Payments: Create ReconciliationDiscrepancy<br/>Type: Amount Mismatch<br/>Expected: $100, Actual: $105
                ReconciliationJob--)EventBus: PaymentDiscrepancyDetected
            else Transaction not found
                ReconciliationJob->>Payments: Create ReconciliationDiscrepancy<br/>Type: Orphan Payment<br/>(Payment in CEPAS but no application)
                ReconciliationJob--)EventBus: PaymentDiscrepancyDetected
            end

        else Command Code = 5 (Refund)
            ReconciliationJob->>Payments: Match refund confirmation number
            ReconciliationJob->>ReconciliationJob: Verify refund recorded
        else Command Code = 6, 9, 10, 15, or 16 (Void, Chargeback, Chargeback Reversal, ACH Credit, Processor Void)
            ReconciliationJob->>ReconciliationJob: Record as ReconciliationRecord with match_result = Discrepancy
            Note over ReconciliationJob: Not auto-resolved - no defined handling for these codes yet; flagged for manual classification
        end
    end

    ReconciliationJob->>Payments: Update ReconciliationBatch<br/>Status: Processing -> Completed<br/>Summary:<br/>- Total Records: 247<br/>- Matched: 240<br/>- Missing Payments Recovered: 5<br/>- Discrepancies: 2

    ReconciliationJob--)EventBus: ReconciliationBatchCompleted
    EventBus--)Monitoring: ReconciliationBatchCompleted event

    alt Discrepancies > 0
        Monitoring--)EventBus: Alert: Manual review needed
        EventBus--)Monitoring: Email: Payment Admin notification<br/>Subject: "2 payment discrepancies require review"
    end

    ReconciliationJob->>ReconciliationJob: End batch job
```

**Key Decisions:**
- **Source of truth:** Posting file is authoritative for payment status
- **Auto-update restrictions:** Only `Pending` transactions updated; `Failed`/`Refunded` left unchanged
- **Discrepancy handling:** Flagged for manual review, not auto-resolved

**State Changes:**
- ReconciliationBatch status: `Processing` -> `Completed`
- PaymentTransaction status: `Pending` -> `Paid` (for missing payments only)

**Events Published:**
- `MissingPaymentDetected` - Per recovered payment
- `PaymentCompleted` - Per recovered payment (triggers credential workflow)
- `PaymentDiscrepancyDetected` - Per unmatched or mismatched record
- `ReconciliationBatchCompleted` - At batch completion with summary

**Error Scenarios:**
Not separately enumerated in this sequence beyond pay:BatchJob_PostingFileReconciliation's own errorHandling.

---

## CEPAS Service Unavailable

**What:** User attempts payment when CEPAS is down; system displays error and allows retry later  
**When:** CEPAS health check fails  
**Who:** Any user attempting payment

```mermaid
---
title: Payments - CEPAS Service Unavailable
---
sequenceDiagram
    actor User
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant CircuitBreaker
    participant Monitoring

    User->>UI: Click "Pay Fee"
    UI->>Payments: GET /payments/initiate?applicationId=APP-123

    Payments->>CircuitBreaker: Check CEPAS health
    CircuitBreaker->>CEPAS: HTTP HEAD https://epay.michigan.gov/payment<br/>Timeout: 5 seconds

    alt CEPAS unavailable
        CEPAS-->>CircuitBreaker: Timeout (no response)
        CircuitBreaker->>CircuitBreaker: Record failure<br/>Failure count: 3/3<br/>Circuit state: CLOSED -> OPEN

        CircuitBreaker--)Monitoring: Alert: CEPAS circuit breaker opened<br/>Timestamp: 2026-01-15 10:30:00

        CircuitBreaker-->>Payments: Health check failed<br/>Circuit state: OPEN

        Payments-->>UI: 503 Service Unavailable<br/>Error: "Payment system temporarily unavailable"

        UI-->>User: Display error message<br/>"The payment system is temporarily unavailable.<br/>Please try again later.<br/>Your application has been submitted."

        Note over CircuitBreaker: Circuit remains OPEN<br/>for 5 minutes (cooldown)

    else CEPAS available (after cooldown)
        Note over CircuitBreaker: After 5-minute cooldown

        User->>UI: Click "Pay Fee" (retry)
        UI->>Payments: GET /payments/initiate?applicationId=APP-123

        Payments->>CircuitBreaker: Check CEPAS health
        CircuitBreaker->>CEPAS: HTTP HEAD https://epay.michigan.gov/payment
        CEPAS-->>CircuitBreaker: 200 OK

        CircuitBreaker->>CircuitBreaker: Health check success<br/>Circuit state: OPEN -> HALF-OPEN -> CLOSED

        CircuitBreaker-->>Payments: Health check passed
        Payments->>Payments: Create PaymentTransaction<br/>Encrypt payload
        Payments-->>UI: 302 Redirect to CEPAS
        UI-->>User: Browser redirects to CEPAS

        Note over User,CEPAS: Payment proceeds normally
    end
```

**Key Decisions:**
- **Circuit breaker pattern:** Prevents cascading failures and reduces load on struggling CEPAS
- **Health check before redirect:** Fail fast with clear error rather than broken redirect
- **Cooldown period:** 5 minutes (configurable) before retry attempts

**Events Published:**
None (circuit breaker alert to Monitoring only, not a modeled sd:DomainEvent).

**Error Scenarios:**
CEPAS unreachable -> 503 Service Unavailable, circuit opens for 5-minute cooldown.

---

## Payment Retry After Failure

**What:** User retries payment after previous attempt failed (declined card, timeout, etc.)  
**When:** User clicks retry button on failed payment error page  
**Who:** Any user with failed payment transaction

```mermaid
---
title: Payments - Payment Retry After Failure
---
sequenceDiagram
    actor User
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant EventBus

    Note over User,Payments: Previous payment attempt failed<br/>Transaction T1: Status=Failed

    User->>UI: View application (Status: Pending Payment)
    UI-->>User: Display "Pay Fee" link<br/>(Link always available for Pending Payment)

    User->>UI: Click "Pay Fee" (retry)
    UI->>Payments: GET /payments/initiate?applicationId=APP-123

    Payments->>Payments: Check for existing Paid transaction<br/>Query: applicationId=APP-123, Status=Paid

    alt Existing Paid transaction found
        Payments-->>UI: 409 Conflict<br/>Error: "Payment already completed"
        UI-->>User: Display: "This application has already been paid."

    else No Paid transaction (only Failed transaction exists)
        Payments->>Payments: Create NEW PaymentTransaction (T2)<br/>Status: Pending<br/>Amount: $100.00<br/>Reference: APP-123

        Note over Payments: Original failed transaction (T1)<br/>preserved in audit trail

        Payments->>Payments: Encrypt payload
        Payments-->>UI: 302 Redirect to CEPAS
        UI-->>User: Browser redirects to CEPAS

        User->>CEPAS: Enter DIFFERENT card details
        User->>CEPAS: Confirm payment

        CEPAS->>CEPAS: Payment successful<br/>Confirmation: CONF-999

        CEPAS-->>User: Redirect to MiEdWorkforce
        User->>UI: Browser redirects back
        UI->>Payments: GET /payments/confirm?c=0&o=CONF-999...

        Payments->>Payments: Validate hash
        Payments->>Payments: Update PaymentTransaction T2<br/>Status: Pending -> Paid<br/>ConfirmationNumber: CONF-999

        Payments--)EventBus: PaymentCompleted

        Payments-->>UI: 200 OK
        UI-->>User: Display success page<br/>Confirmation: CONF-999

        Note over Payments: Transaction T1: Status=Failed (preserved)<br/>Transaction T2: Status=Paid (successful retry)
    end
```

**Key Decisions:**
- **Each retry creates new transaction:** Failed transactions preserved for audit trail
- **Prevent double payment:** Check for existing Paid transaction before creating new one
- **Unlimited retries:** User can attempt payment as many times as needed

**State Changes:**
- New PaymentTransaction (T2) created: `Pending` -> `Paid`
- Original failed transaction (T1): Remains `Failed`

**Events Published:**
PaymentCompleted - Fires on successful retry.

**Error Scenarios:**
Existing Paid transaction found for the application -> 409 Conflict, "This application has already been paid."

---

## Bulk Refund Processing

**What:** Credential Admin selects multiple paid transactions for bulk refund  
**When:** Multiple applications need refunds (e.g., district error, program cancellation)  
**Who:** Credential Administrator (with Financial Admin approval for high values)

```mermaid
---
title: Payments - Bulk Refund Processing
---
sequenceDiagram
    actor CredAdmin as Credential Admin
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant EventBus
    participant Comms as Communications

    CredAdmin->>UI: Navigate to bulk refund page
    UI->>Payments: GET /payments?status=Paid&districtId=12345
    Payments-->>UI: List of paid transactions

    UI-->>CredAdmin: Display paid transactions table

    CredAdmin->>UI: Select 5 transactions:<br/>CONF-100 ($50), CONF-101 ($50),<br/>CONF-102 ($50), CONF-103 ($75),<br/>CONF-104 ($50)

    CredAdmin->>UI: For each, enter refund amount:<br/>All full refunds (total: $275)

    CredAdmin->>UI: Enter reason: "District processing error"
    CredAdmin->>UI: Click "Submit Bulk Refund"

    UI->>Payments: POST /refunds/bulk-request<br/>transactions: [{id, amount, reason}, ...]<br/>Total: $275

    Payments->>Payments: Validate all amounts <= original: YES
    Payments->>Payments: Check threshold: $275 <= $500: YES<br/>Single approval (auto-approved)

    Payments->>Payments: Create 5 RefundRequest records<br/>All Status: Approved (auto)<br/>LinkedBulkId: BULK-REFUND-789

    loop For each refund (5 requests)
        Payments->>CEPAS: POST /api/v1/payment/cancel<br/>ConfirmationNumber: CONF-100<br/>RefundAmount: 50.00

        CEPAS->>CEPAS: Process refund
        CEPAS-->>Payments: 200 OK<br/>RefundConfirmationNumber: REFUND-200

        Payments->>Payments: Update RefundRequest: Completed<br/>RefundConfirmationNumber: REFUND-200

        Payments->>Payments: Update PaymentTransaction: Refunded
    end

    Payments--)EventBus: BulkRefundCompleted<br/>Payload: refundIds: [5 IDs], totalAmount: $275

    EventBus--)Comms: BulkRefundCompleted event
    Comms--)CredAdmin: Email: Bulk refund processed<br/>5 refunds: $275 total

    Payments-->>UI: 200 OK<br/>Summary: 5 refunds processed

    UI-->>CredAdmin: Display success:<br/>"5 refunds processed successfully"<br/>Confirmation numbers: REFUND-200...REFUND-204
```

**Key Decisions:**
- **Sequential processing:** Refunds processed one-by-one via CEPAS API (not atomic batch)
- **Partial success handling:** If one fails, others continue; failure logged separately
- **Threshold calculation:** Total refund amount determines approval requirements

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
**Who:** Payment Administrator

```mermaid
---
title: Payments - Reconciliation Discrepancy Investigation
---
sequenceDiagram
    actor PaymentAdmin as Payment Admin
    participant UI as MiEdWorkforce UI
    participant Payments
    participant CEPAS
    participant EventBus

    Note over Payments: Reconciliation batch detected discrepancy<br/>Type: Amount Mismatch<br/>Expected: $100, Actual: $105

    PaymentAdmin->>UI: Navigate to discrepancies page
    UI->>Payments: GET /reconciliation/discrepancies?status=Unresolved
    Payments-->>UI: List of unresolved discrepancies

    UI-->>PaymentAdmin: Display discrepancies table<br/>Discrepancy #1:<br/>- Type: Amount Mismatch<br/>- Application: APP-123<br/>- Expected: $100.00<br/>- CEPAS Posted: $105.00<br/>- Confirmation: CONF-789

    PaymentAdmin->>UI: Click discrepancy to investigate
    UI->>Payments: GET /reconciliation/discrepancies/{id}
    Payments-->>UI: Discrepancy details + related transaction

    UI-->>PaymentAdmin: Display full details:<br/>- PaymentTransaction: APP-123, $100<br/>- Posting file record: CONF-789, $105<br/>- Date: 2026-01-15

    Note over PaymentAdmin: Investigation:<br/>1. Check CEPAS portal for transaction<br/>2. Contact CEPAS support (Sean Strom)<br/>3. Determine: $5 processing fee charged<br/>   (Not communicated in advance)

    PaymentAdmin->>UI: Enter resolution notes:<br/>"Contacted CEPAS support. $5 processing fee<br/>added by CEPAS. Confirmed with Sean Strom<br/>that this is expected. Updating fee schedule."

    PaymentAdmin->>UI: Select resolution action:<br/>"Accept CEPAS amount as correct"

    PaymentAdmin->>UI: Click "Resolve Discrepancy"

    UI->>Payments: POST /reconciliation/discrepancies/{id}/resolve<br/>resolution: "Accept CEPAS amount"<br/>notes: "..."<br/>correctedAmount: 105.00

    Payments->>Payments: Update PaymentTransaction<br/>Amount: $100 -> $105<br/>Notes: "Adjusted per CEPAS posting file"

    Payments->>Payments: Update ReconciliationDiscrepancy<br/>Status: Unresolved -> Resolved<br/>ResolvedBy: Payment Admin<br/>ResolvedAt: 2026-01-16<br/>ResolutionNotes: "..."

    Payments--)EventBus: DiscrepancyResolved

    Payments-->>UI: 200 OK
    UI-->>PaymentAdmin: Display: "Discrepancy resolved"

    Note over PaymentAdmin: Follow-up action:<br/>Update fee schedule to include<br/>$5 CEPAS processing fee
```

**Key Decisions:**
- **Manual resolution required:** No auto-resolution of discrepancies
- **Audit trail:** All resolution actions logged with notes
- **Resolution options:** Accept CEPAS, Reject (mark transaction Failed), Create adjustment

**State Changes:**
- ReconciliationDiscrepancy status: `Unresolved` -> `Resolved`
- PaymentTransaction: May be updated based on resolution type

**Events Published:**
- `DiscrepancyResolved` - Fires when admin marks discrepancy resolved

**Error Scenarios:**
Not separately enumerated in this sequence.
