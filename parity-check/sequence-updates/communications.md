# Communications sequences: change log

Doc edited: design/working-docs/solution-areas/communications/communications-sequences.md
Diagrams before: 11. Diagrams after: 15. Splits: 4 (2 flagged in 3.1 row 16, 2 applied from the style guide).

## Conventions block

Updated: Actor is humans only; every other party is a participant; box grouping; API-kind tags APP / SVC / EXT / OUT; added the standing sentence on IAM authorization. Async arrows now described as events.

## Sections changed

All diagrams: removed repository, database and cache participants (Template Repository, EmailInstance Repository, Alert Repository, Cosmos DB, SQL Database, Cache) and replaced them with self arrows or notes; removed Validator and Resolver participants (now self arrows on Communications API); removed "Validate permissions" self arrows and the permission notes; removed plain OK/confirmation responses; declared every participant; added boxes Browser, MiEdWorkforce (AKS), External; UI alias is UI; Identity became IAM API; Domain Service became Credentialing API; Dashboard became UI; SendGrid sits in box External; paths copied from the spec (no /api prefix, no invented paths).

- Manual Email Send with Template Customization: tags added, paths fixed (templates, available-variables, preview, send-manual); Resolver and repositories dropped; Who gained permissions. Not split (3.1 row 16 gives no split for it).
- Email History Search and Details View: tags, paths fixed (/email-history, /email-history/{instanceId}); repositories and Identity participant (unused) dropped; SQL note and permission note removed; Who gained permissions. One alt kept (archived content).
- SendGrid Webhook - Delivery Status Update (3.2 depth, 3.4 row 2): SendGrid is an external participant calling Webhook Endpoint with EXT POST /webhooks/sendgrid; Validator participant folded into a note on the endpoint; nested alts reduced to one (invalid signature); orphan and duplicate branches moved to Error Scenarios (one line extended with the race condition remark). Four events drawn as one arrow.
- Email Content Purge Job: Cosmos DB and SQL participants removed (self arrows); three alts reduced to one (delete failed); threshold alert and next-batch logic merged into a Monitoring arrow and one note. Scheduler trigger arrow is untagged because it is not an API call.
- Template Variable Resolution (Internal) (3.2 notes): Resolver and Cache participants removed; notes reduced to 3; paths tagged SVC. Section kept (internal component, not flagged for removal).
- User Resolves Dashboard Alert: Dashboard became UI; tag and path fixed (POST /alerts/{alertId}/resolve); permission on Who line.
- Mass Email with Consolidation (3.1 row 25): Consolidator, Resolver, template and instance participants removed; notes cut from 9 to 2; consolidation rules (window, timer, grouping, template and list variables) moved to a new Key Decisions bullet "Consolidation rules". Data Quality Service call tagged SVC.
- Notes section: three standards (Template Variable Formatting, Event Payload Design, Resend vs. New Send matrix) kept in place with a "Proposed home:" line each (3.3).
- Integration Flows (3.3): moved below Notes and retitled "Reference Material - Integration Flows (not sequences)" with a proposed home line. Only change inside it: the endpoint path now matches the spec (GET /event-types/{eventType}/available-variables). Webhook endpoint URL in it still shows /api/webhooks/sendgrid (see Questions).

## Splits

- Event-Triggered Email Send to "Event-Triggered Email Send - Render Email from Event" and "Event-Triggered Email Send - Send and Record Email" (3.1 row 16).
- Email Resend with Recipient Override to "... - Load Resend Context" and "... - Resend Email" (3.1 row 16).
- Template Version Update and Activation (3.2, arrows) to "... - Create Draft Version" and "... - Activate Version". Own judgement: two verbs (create, activate) with a pause for review.
- Dashboard Alert Creation and Display (own judgement from 2.1: publisher plus consumer, and 7 notes) to "... - Create Alert from Event" and "... - Display Alerts".
Key Decisions, State Changes, Events and Error Scenarios lines were divided between the new sections without rewording, except: Mass Email gained one bullet, Create Draft shows the state change `None > Draft` (original had Draft > Active for both steps).

## Sections left alone

None left fully alone. Manual Email Send and Mass Email were not split (no split proposed).

## Missing operations (no operationId in any spec)

| Area | Tag | Verb and path | Section |
|---|---|---|---|
| communications | APP | GET /email-history/{instanceId}/resend-context | Email Resend - Load Resend Context |
| communications | SVC | process delivery event (Webhook Endpoint to Communications API; no path) | SendGrid Webhook - Delivery Status Update |
| data quality (no area spec) | SVC | GET /issues?ids=101,102,103 | Mass Email with Consolidation |
| communications | OUT | POST /v3/mail/send (SendGrid REST, external; not a MiEdWorkforce operation) | several; listed for completeness |

Event-driven steps (event consumption, alert creation from event, recipient resolution from template config) have no operation; they are not drawn as API calls.

## API kind observations

- receiveSendGridWebhook is x-access internal-service and "not exposed via APIM", yet its description says SendGrid calls it directly with HMAC. The diagram draws SendGrid calling a Webhook Endpoint as EXT, per the architecture rule that webhooks are External APIs. The spec should probably be external-client (or a distinct external-facing endpoint) with a separate internal hop.
- getApplication (credentialing) and getUser (IAM) are x-access internal-user only, but Communications calls them service to service (SVC) to resolve template variables and recipients, with no user token. They likely also need internal-service access. No operation in these specs is marked both external and application.
- listTemplates, getAvailableVariables, previewEmail, sendManualEmail, searchEmailHistory, getEmailInstance, resendEmail, listAlerts, resolveAlert, activateTemplateVersion, createTemplateVersion, getTemplate: used as APP, consistent with internal-user.

## Questions

1. Permission keys: the spec and the sequences use `communications.credentials-*`, the permissions doc uses `communications.credentialing-*`. Which is right? The Who lines use the spec spelling.
2. listAlerts and resolveAlert both require `communications.alerts.view`, which the permissions doc describes as viewing alert templates for a functional area. Is a separate user-level permission for own alerts intended?
3. Spec paths /communications/send-manual and /communications/preview repeat the service prefix under a base URL that already has /communications. Intended?
4. The resend-context call (GET /email-history/{instanceId}/resend-context) has no operation. Add one, or fold the recipient hint into getEmailInstance?
5. Who owns Data Quality (publisher of DataQualityIssueDetected and the issues lookup)? It is drawn as "Data Quality Service" because no canonical API name exists.
6. Is Monitoring inside AKS or an Azure service (External)? Drawn inside the AKS box for now.
7. Event-triggered recipient resolution calls IAM "from template config"; the sequence reuses GET /users/{userId}. Is there a dedicated operation, and how are recipients that are not users handled?
8. SendGrid webhook URL in the reference material shows /api/webhooks/sendgrid and the host miedworkforce.michigan.gov; spec says /webhooks/sendgrid under api.miedworkforce.mi.gov. Which is current?
9. Original text said the SendGrid rejection also publishes EmailSendFailed; it is drawn only in the render diagram (variable failure). Should the send diagram show it too?
10. Open question from the original diagrams removed from diagrams: "Alert admin if critical event" (silent failure note) is now only in the Error Scenarios text as it was; who is alerted and how is undefined.
11. Template Variable Resolution (Internal) is not a workflow; consider moving to the technical design doc.
12. Purge job: per-record handling could be split from the batch loop per 2.1; not done since not flagged.
