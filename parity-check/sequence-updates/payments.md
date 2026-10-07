# Payments sequences: change log

File edited: design/working-docs/solution-areas/payments/payments-sequences.md

Counts: 11 diagrams before, 11 after (10 sections kept, 1 section split into 2, 2 sections merged into 1). Largest diagram is now 24 arrows (was 33). All diagrams are within the hard limits (7 participants, 25 arrows, nesting 2).

## Changes applied to every diagram

- Conventions list updated: Actor is humans only, participant is every non-human, API kind tags (APP, SVC, EXT, OUT) explained, box grouping explained, standing sentence about cached IAM authorization added.
- Participants renamed to canonical names and declared with box grouping: Browser (UI), MiEdWorkforce (AKS) (Payments API as PayApi, Credentialing API as CredApi, Event Bus as EventBus), External (CEPAS, File Transfer Service). Removed: Payments (bare name), Credentials, Communications, Monitoring, CircuitBreaker, ReconciliationJob, undeclared FinAdmin/User use in refund diagrams (now declared).
- Every request arrow between participants tagged. Verbs and paths taken from payments-api.yml (no /api/v1 prefix, no invented paths). Calls to CEPAS are tagged OUT and keep CEPAS's own path.
- Consumers of events (Credentialing, Communications, monitoring) removed as participants. Each published event has one Note on the Event Bus saying who consumes it. No email sends are drawn.
- Self arrows that only described authorization (Validate user permission, scope check) removed. Permission keys are now on the Who line.
- Plain OK responses removed. Redirect preamble no longer repeated in variants.

## Sections changed

1. Individual Payment Flow (Happy Path): re-tagged. Initiate is POST /payments/initiate (doc drew GET with query string; spec is POST). Initiate response shown as 200 cepasRedirectUrl (doc drew 302 from the API; spec returns JSON and the client redirects). Browser redirect to CEPAS drawn as OUT from UI. Return call is EXT GET /payments/confirm (public). Who gains Permission line.
2. Individual Payment Flow (Browser Closed - Missing Payment): title now "Payments - Individual Payment - Missing Payment Recovery". Starts at the divergence (Note about the closed browser) instead of repeating the initiate and redirect preamble. User and UI participants dropped. ReconciliationJob treated as the nightly job inside Payments API (see Questions). CEPAS posting the file to FTS is a Note, not an arrow.
3. Individual Payment Flow (Payment Failed): title now "Payments - Individual Payment - Payment Failed". Starts at the CEPAS decline and redirect back. Confirm call is EXT.
4. Bulk Payment Processing: re-tagged. Credentials renamed Credentialing API. Permission check self arrow removed, Who line gains Permission. Review and payment on CEPAS compressed into one Note that refers to the base flow, then the confirm call as in the base flow. Missing operation logged below. Existing typo `payments.transaction.initiate.bulk` in the Who sentence left as is (see Questions).
5. Refund Request and Approval (merged from the two Refund Request Processing sections, see Merges below). One alt on the amount threshold.
6. Daily Posting File Reconciliation: reduced to the batch level, retitled "Payments - Reconciliation - Reconcile Posting File". Monitoring participant and the nested alt for discrepancies removed (Note on the Event Bus instead).
7. CEPAS Service Unavailable: title now "Payments - Individual Payment - CEPAS Unavailable". CircuitBreaker participant removed (it is a component inside Payments API, drawn as a self arrow and an OUT HEAD health check to CEPAS). Monitoring alert became a Note. The repeated second "Pay Fee" call after cooldown is collapsed into the else branch plus a Note. Initiate response codes match the spec (503 when the circuit is open).
8. Payment Retry After Failure: title now "Payments - Individual Payment - Retry After Failure". Starts at divergence. The second half (CEPAS payment, confirm, PaymentCompleted) is replaced by a Note that refers to the happy path, which removes the PaymentCompleted event that was drawn but not listed in the section text. Initiate is POST.
9. Bulk Refund Processing: re-tagged. Search uses GET /payments/search (doc drew GET /payments?status=Paid&districtId=12345, not in spec). Who gains Permission line. Threshold and create steps merged into one self arrow.
10. Reconciliation Discrepancy Investigation: re-tagged, CEPAS participant dropped (unused), plain 200 OK responses removed. Who gains Permission line.

## Splits and merges

Split:
- Daily Posting File Reconciliation becomes "Daily Posting File Reconciliation" (diagram title "Payments - Reconciliation - Reconcile Posting File") and new section "Reconcile One Posting Record" (diagram title "Payments - Reconciliation - Reconcile One Posting Record"), placed right after it. Each has the full section shape and a See also line. Text blocks were divided between them: Permissions and Monitoring stay with the batch section; Key Decisions, State Changes and Events Published were divided by subject.
- The command code alt tree (codes 1, 5, 6, 9, 10, 15, 16) moved into a "Command Code Handling" table in the new section. Row content matches the original diagram. Suggested final home per the review (3.3): the capability doc; I could not edit it. The new diagram draws the recovered payment and the amount mismatch paths only (one alt).

Merged (review item 14):
- "Refund Request Processing (Single Approval ≤$500)" and "Refund Request Processing (Dual Approval >$500)" are now one section "Refund Request and Approval" with one diagram, alt on the threshold. Text blocks were combined with original wording (Key Decisions lines kept, both Permissions lines kept, State Changes gains one line for each path, Events Published marks the dual-approval-only events, Error Scenarios from the single section; the dual section had none). The Financial Admin rejection path is still only in Key Decisions text.

Review items not applied as a split: item 24 (bulk and individual variants). Variants were rebuilt to start at the divergence but kept as separate sections with their existing headings; no new sections were needed.

## Sections left alone

None left wholly alone. Text blocks (Key Decisions, State Changes, Events Published, Error Scenarios, Business Impact, Audit Trail, Circuit Breaker States, Resolution Types, Monitoring) are unchanged except where a split or merge required moving them.

## Missing operations (no operationId in any spec)

| Area | Tag | Verb and path | Section |
|---|---|---|---|
| payments | APP | GET /applications/pending-payment | Bulk Payment Processing (UI lists applications through Payments API; no such operation in payments-api.yml. Credentialing has listApplications, GET /applications, filter status=PendingPayment, but no district filter) |
| payments | APP | GET /reconciliation/discrepancies/{discrepancyId} | Reconciliation Discrepancy Investigation (spec has list and resolve, no detail operation) |

Not drawn as missing because they are now internal to Payments API (self arrows): create and update ReconciliationBatch, query by reference, create ReconciliationDiscrepancy, health check. Note there is also no operation for the manual reconciliation trigger (permission `payments.reconciliation.run`) or for viewing batch results (`payments.reconciliation.view`); neither is drawn.

Calls to CEPAS (POST /api/v1/payment/cancel, HEAD health check, SFTP GET) are OUT calls to an external system and are not in any OpenAPI spec by design.

## API kind observations

- `confirmPayment` (GET /payments/confirm) is x-access external-public, but the sequence shows the user's browser (the UI return page) calling it after the CEPAS redirect. It is tagged EXT (public) with the UI as caller. If the return URL is served by the UI and the UI then calls the API, the call would be Application (APP) without a user token; if CEPAS redirects the browser straight to the API hostname it is EXT. Needs an owner decision.
- `initiatePayment` and `initiateBulkPayment` are x-access internal-user. The API description names the Credentials domain service as a consumer for bulk initiation and the section Key Decisions say "the calling domain validates application eligibility before Payments is invoked", which suggests a Service API (SVC) call from Credentialing API. The sequence draws the UI calling them (APP). No operation is marked both external and application.
- `listApplications` (Credentialing, internal-user) is used by Payments API as a service call (SVC GET /applications) in Bulk Payment Processing. A service-to-service use of an internal-user operation suggests it needs an internal-service variant, or the list should come from the UI to Credentialing API directly.
- `confirmPayment` is used for both success and failure returns (200 with status Failed in the failed variant); spec says 400 only for hash failure or missing parameters.

## Questions

1. Is ReconciliationJob a scheduled job inside Payments API (as treated here, drawn as Payments API self arrows) or a separate worker that should be its own participant calling Payments API operations? The spec has no operations for batch creation or record updates.
2. "Monitoring" consumer: the Daily Posting File Reconciliation diagram had Monitoring receiving ReconciliationBatchCompleted and emailing the Payment Admin. Monitoring is not defined in solution-architecture.md. Kept as wording in a Note. Confirm who sends the Payment Admin notification (Communications consuming an event, or an alerting tool).
3. The original reconciliation alt listed codes 6, 9, 10, 15, 16 only. Code 14 (Partial Refund) appears in the field list and capability doc but has no handling. Is it also flagged as Discrepancy?
4. Capability doc says only 1 and 5 are actively reconciled; the document's code 5 row (match refund confirmation number, verify refund recorded) is kept as in the original.
5. Bulk Refund Processing shows synchronous processing and a 200 summary, while requestBulkRefund returns 202 accepted for processing. Which is intended? Also `BulkRefundCompleted` is published in the section text but is not in the capability doc event catalog (also: PaymentInitiated, PaymentReconciled, RefundFailed, DiscrepancyResolved are not all aligned between catalog and sequences; DiscrepancyResolved is drawn and listed here but absent from the catalog).
6. Initiate response: the spec returns 200 with cepasRedirectUrl, the original diagrams and prose say 302 redirect from Payments. Diagrams now follow the spec; the Happy Path Key Decisions and the capability doc still mention redirects. Confirm.
7. The Bulk Payment Who sentence says `payments.transaction.initiate.bulk` (with a dot) while the spec and Permissions block use `payments.transaction.initiate-bulk`. Left unchanged as prose; suggest fixing the typo.
8. Browser redirect arrows to CEPAS are tagged OUT (browser as caller, per the style guide). The tag table has no explicit tag for browser redirects. Confirm OUT is right.
9. File Transfer Service is drawn as an External participant. It is not in the solution-integrations.md canonical list (the CEPAS section names it, upgrade status is open). Confirm label and box.
10. Should the unhandled requirement for `payments.transaction.retry` (permissions doc) appear in the Retry section Who line? The spec uses payments.transaction.initiate for retries; the Who line uses that key.
11. Sections where the original text blocks referred to the old split sections: "Daily Posting File Reconciliation" is still referenced by name in capability doc sequence lists, as are "Refund Request Processing" and "Missing Payment Recovery". The merged section heading changed to "Refund Request and Approval", so the capability doc references to "Sequence: Refund Request Processing" need updating by the lead.
12. Review item 3.3 suggests the command code matrix lives in the capability doc. A table now sits in the new section; confirm whether to copy it to payments-capability.md.
