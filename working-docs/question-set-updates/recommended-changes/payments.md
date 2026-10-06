# Recommended Changes: Payments

**Impact: Medium, mostly clarification.** The Payments model itself does not need to change. Files under `design/working-docs/solution-areas/payments/`.

## Principle

Question Sets never calls Payments. The sequencing of review, question responses and payment belongs to Credentialing, which owns the application state and fee computation. Payments owns the transaction.

## Changes

1. **payments-sequences.md, Individual Payment Flow (L14-85).** Add a precondition note: the payment link is available only when Credentialing has made it available. For applications with a requirement set to review before payment, Credentialing withholds the link until that review is released. The existing step `Pending Payment to In Review` on `PaymentCompleted` (L60) remains correct for the common after-payment case.
2. **payments-capability.md business rule (L466-470, CEPAS-down handling).** The rule says submission is not blocked and users can submit and pay later. Reconcile the wording with the gating flow: submission is separate from payment timing, and the point at which a payment link becomes available is an input from Credentialing. This also addresses the partial contradiction between "pay later" and "Pending Payment to In Review".
3. **Scope statement (L38-47).** No change; fee schedules and pricing remain in Credentialing. Add a one-line example that review-before-payment is a Credentialing path, not a Payments feature.
4. **Dependencies (L329-352).** No new coupling. All integration stays event-based (`PaymentCompleted`, `PaymentFailed`, `RefundCompleted`, `BulkPaymentCompleted`).
5. **Bulk payments.** `BulkPaymentCompleted` currently moves 10 applications together (L297, L317). If some of the 10 are in pre-payment review, they cannot be in the bulk set. Add the guard: only applications with an available payment link are eligible for bulk payment.
6. **Refunds.** If an application is denied after pre-payment review, no payment was taken; if denied after payment, existing refund rules apply. No new rule, but make sure the denial sequences state which case they are in.

## Credentialing technical question that bears on this

Open Technical Question #7 (how Credentialing learns payment status; Payments has no internal status endpoint). The review-before-payment path adds weight to the event-only answer (subscribe to `PaymentCompleted`), since Credentialing now also needs to know that a link has not yet been issued, which is its own state.

## Tracking references

OQ-QS-10 (review before payment; already logged 2026-09-23 in `hub/tracking/open-questions.md`). See [../tracking-proposals.md](../tracking-proposals.md).
