# Communications - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Communications platform capability.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

---

## Event-Triggered Email Send

**What:** Domain event triggers automatic email send using pre-configured template  
**When:** Business event occurs (e.g., credential application approved, payment due)  
**Who:** System (automated)

```mermaid
---
title: Communications - Event-Triggered Email Send
---
sequenceDiagram
    participant Domain as Domain Service
    participant EventBus
    participant Comms as Communications Service
    participant TemplateRepo as Template Repository
    participant Resolver as Variable Resolver
    participant Identity
    participant SendGrid
    participant InstanceRepo as EmailInstance Repository

    Domain--)EventBus: Publish event (e.g., CredentialApplicationApproved)
    Note over EventBus: Event payload:<br/>{event_type, application_id, approved_at}

    EventBus->>Comms: Event received
    Comms->>Comms: Identify event type
    Comms->>TemplateRepo: Get active template for event type
    TemplateRepo-->>Comms: Template (with placeholders)

    alt No active template for event type
        Comms->>Comms: Log warning (no template configured)
        Note over Comms: Silent failure - no email sent<br/>Alert admin if critical event
    else Template found
        Comms->>Resolver: Resolve variables for event type
        Note over Resolver: Event-type-specific resolver:<br/>CredentialApplicationApprovedResolver

        Resolver->>Domain: GET /api/credentials/applications/{id}
        Domain-->>Resolver: Application data
        Resolver->>Identity: GET /api/identity/users/{id}
        Identity-->>Resolver: User data

        Resolver->>Resolver: Format values to user-facing strings<br/>e.g., "Jane Doe", "Elementary Education"
        Resolver-->>Comms: Resolved variables

        alt Mandatory variable missing/null
            Comms->>Comms: Block send, log error
            Comms--)EventBus: EmailSendFailed
            Note over Comms: Error: Mandatory variable unresolvable
        else All variables resolved
            Comms->>Comms: Render email (replace placeholders)
            Note over Comms: Rendered content:<br/>"Dear Jane Doe, your application<br/>for Elementary Education..."

            Comms->>Identity: Resolve recipient(s) from template config
            Identity-->>Comms: Recipient email(s)

            Comms->>SendGrid: POST /v3/mail/send
            Note over SendGrid: Email queued for delivery
            SendGrid-->>Comms: 202 Accepted (message_id)

            Comms->>InstanceRepo: Store EmailInstance
            Note over InstanceRepo: Stores ONLY rendered content<br/>NOT event payload or template structure
            InstanceRepo-->>Comms: instance_id

            Comms--)EventBus: EmailSent {instance_id, recipients, sent_at}
        end
    end
```

**Key Decisions:**
- **Event payload:** Lightweight (IDs only), resolver fetches display data
- **Variable resolution:** Event-type-specific resolver handles formatting
- **Silent failure:** No template = no email (prevents spam on misconfigured events)
- **Rendered content only:** EmailInstance stores final output, not intermediate data

**State Changes:**
- EmailInstance status: `None` > `Queued`

**Events Published:**
- `EmailSent` - Email successfully queued to SendGrid
- `EmailSendFailed` - Variable resolution failed or SendGrid rejected

**Error Scenarios:**
- No active template > Silent failure, log warning
- Mandatory variable null > Block send, publish failure event
- SendGrid API error > Retry with exponential backoff (max 3 attempts)

---

## Manual Email Send with Template Customization

**What:** User composes and sends one-time email using template as starting point  
**When:** User needs to send custom communication (e.g., follow-up, clarification)  
**Who:** Credentialing Administrator, Help Desk Staff

```mermaid
---
title: Communications - Manual Email Send with Template Customization
---
sequenceDiagram
    actor User
    participant UI
    participant Comms as Communications Service
    participant TemplateRepo as Template Repository
    participant Resolver as Variable Resolver
    participant Identity
    participant SendGrid
    participant InstanceRepo as EmailInstance Repository
    participant EventBus

    User->>UI: Navigate to "Send Email"
    UI->>Comms: GET /api/communications/templates?functional_area=Credentialing
    Comms->>TemplateRepo: Query active templates
    TemplateRepo-->>Comms: List of templates
    Comms-->>UI: Templates with event types

    User->>UI: Select template (e.g., "Application Approved")
    UI->>Comms: GET /api/communications/event-types/{event_type}/available-variables
    Comms-->>UI: Available variables for event type
    Note over UI: Displays variable picker:<br/><<ApplicantName>>, <<CertificateType>>, etc.

    User->>UI: Enter context data (e.g., application_id)
    UI->>Comms: POST /api/communications/preview
    Note over UI: Request: {template_id, context: {application_id}}

    Comms->>Resolver: Resolve variables using context
    Resolver->>Identity: Fetch user data
    Identity-->>Resolver: User details
    Resolver-->>Comms: Resolved variables

    Comms->>Comms: Render preview
    Comms-->>UI: Preview HTML

    User->>UI: Customize subject/body (optional)
    User->>UI: Enter recipient email, click Send

    UI->>Comms: POST /api/communications/send-manual
    Note over UI: Request: {template_id, context,<br/>custom_subject, custom_body, recipient}

    Comms->>Comms: Validate permissions
    Comms->>Comms: Render final email (with customizations)

    Comms->>SendGrid: POST /v3/mail/send
    SendGrid-->>Comms: 202 Accepted (message_id)

    Comms->>InstanceRepo: Store EmailInstance
    Note over InstanceRepo: Stores rendered content<br/>with user customizations applied
    InstanceRepo-->>Comms: instance_id

    Comms--)EventBus: EmailSent {instance_id, sent_by_user_id}
    Comms-->>UI: Success (instance_id)
    UI-->>User: Email sent confirmation
```

**Key Decisions:**
- **Preview before send:** User sees resolved content before sending
- **Customization scope:** User can modify subject/body but NOT template variables
- **Permissions:** Requires `communications.email.send-manual` permission
- **Audit trail:** EmailInstance captures who sent it (`sent_by_user_id`)

**State Changes:**
- EmailInstance status: `None` > `Queued`

**Events Published:**
- `EmailSent` - Includes `sent_by_user_id` for manual sends

**Error Scenarios:**
- Insufficient permissions > 403 Forbidden
- Invalid context data (e.g., application_id not found) > 400 Bad Request
- SendGrid error > Display error, allow retry

---

## Template Version Update and Activation

**What:** Admin creates new template version and activates it  
**When:** Template content needs updating (e.g., policy change, branding update)  
**Who:** Credentialing Administrator, System Administrator

```mermaid
---
title: Communications - Template Version Update and Activation
---
sequenceDiagram
    actor Admin
    participant UI
    participant Comms as Communications Service
    participant TemplateRepo as Template Repository
    participant EventBus

    Admin->>UI: Navigate to Template Management
    UI->>Comms: GET /api/communications/templates/{template_id}
    Comms->>TemplateRepo: Get template with all versions
    TemplateRepo-->>Comms: Template + versions
    Comms-->>UI: Template details

    Admin->>UI: Click "Create New Version"
    UI->>Comms: GET /api/communications/event-types/{event_type}/available-variables
    Comms-->>UI: Available variables for this template's event type
    Note over UI: Variable picker shows only<br/>context-appropriate variables

    Admin->>UI: Edit subject/body, insert variables
    Admin->>UI: Save as Draft

    UI->>Comms: POST /api/communications/templates/{template_id}/versions
    Note over UI: Request: {subject, body, status: Draft}

    Comms->>Comms: Validate permissions
    Comms->>Comms: Validate variable placeholders match event type

    Comms->>TemplateRepo: Create new TemplateVersion
    TemplateRepo-->>Comms: version_id

    Comms--)EventBus: EmailTemplateVersionCreated
    Comms-->>UI: Success (version_id)

    Note over Admin,UI: Admin reviews draft, tests with preview

    Admin->>UI: Click "Activate Version"
    UI->>Comms: POST /api/communications/templates/{template_id}/versions/{version_id}/activate

    Comms->>Comms: Validate permissions
    Comms->>TemplateRepo: Begin transaction
    TemplateRepo->>TemplateRepo: Deactivate current active version
    TemplateRepo->>TemplateRepo: Set new version as active
    TemplateRepo->>TemplateRepo: Commit transaction
    TemplateRepo-->>Comms: Success

    Note over TemplateRepo: Template now has:<br/>v1.0 (inactive), v2.0 (active)

    Comms--)EventBus: EmailTemplateActivated {template_id, version_id}
    Comms-->>UI: Success
    UI-->>Admin: "Template activated" confirmation
```

**Key Decisions:**
- **Single active version:** Activating v2.0 automatically deactivates v1.0
- **Draft-then-activate workflow:** Prevents accidental activation of untested content
- **Variable validation:** System validates placeholders match event type's available variables
- **Audit trail:** All versions preserved (never deleted)

**State Changes:**
- TemplateVersion: `Draft` > `Active`
- Previous active version: `Active` > `Inactive`

**Events Published:**
- `EmailTemplateVersionCreated` - New draft created
- `EmailTemplateActivated` - Version set as active

**Error Scenarios:**
- Invalid variable placeholder (not in event type) > 400 Bad Request
- Insufficient permissions > 403 Forbidden
- Concurrent activation conflict > 409 Conflict, retry

---

## Email History Search and Details View

**What:** User searches sent emails and views full details  
**When:** Help desk troubleshooting, user inquiry, audit review  
**Who:** Credentialing Administrator, Help Desk Staff, System Administrator

```mermaid
---
title: Communications - Email History Search and Details View
---
sequenceDiagram
    actor User
    participant UI
    participant Comms as Communications Service
    participant InstanceRepo as EmailInstance Repository
    participant Identity

    User->>UI: Navigate to Email History
    UI->>Comms: GET /api/communications/email-history?functional_area=Credentialing&page=1

    Comms->>Comms: Validate permissions (functional area scoped)
    Note over Comms: Checks: communications.credentials-emails.view-history

    Comms->>InstanceRepo: Query EmailInstances
    Note over InstanceRepo: SELECT * FROM email_instances<br/>WHERE functional_area='Credentialing'<br/>ORDER BY sent_at DESC<br/>LIMIT 50
    InstanceRepo-->>Comms: List of instances (metadata only)

    Comms-->>UI: Email list (sent_at, recipient, subject, status)
    UI-->>User: Display searchable table

    Note over User,UI: User applies filters

    User->>UI: Filter by recipient, date range, status
    UI->>Comms: GET /api/communications/email-history?recipient=jane@example.com&status=Bounced
    Comms->>InstanceRepo: Query with filters
    InstanceRepo-->>Comms: Filtered results
    Comms-->>UI: Updated list

    User->>UI: Click email to view details
    UI->>Comms: GET /api/communications/email-history/{instance_id}

    Comms->>Comms: Validate permissions
    Comms->>InstanceRepo: Get EmailInstance with full content
    InstanceRepo-->>Comms: Complete instance
    Note over InstanceRepo: Includes:<br/>- Rendered subject/body<br/>- Delivery events timeline<br/>- Template version used<br/>- Sent by user (if manual)

    alt Email content archived
        Comms-->>UI: Instance metadata only (content unavailable)
        Note over UI: Display: "Email content archived<br/>(retention period expired)"
    else Content available
        Comms-->>UI: Full instance details
        UI-->>User: Display email preview, metadata, delivery timeline
    end
```

**Key Decisions:**
- **Functional area scoping:** User only sees emails for their permitted functional areas
- **Pagination:** Default 50 results per page
- **Metadata vs content:** List view shows metadata, detail view fetches full content
- **Content availability:** UI indicates if content has been archived

**State Changes:**
- None (read-only operation)

**Events Published:**
- None

**Error Scenarios:**
- Insufficient permissions > 403 Forbidden
- Instance not found > 404 Not Found
- Content archived but audit metadata available > Display limited info

---

## Email Resend with Recipient Override

**What:** User resends previously sent email to new/corrected recipient  
**When:** Email bounced, user provided updated email, help desk escalation  
**Who:** Credentialing Administrator, Help Desk Staff

```mermaid
---
title: Communications - Email Resend with Recipient Override
---
sequenceDiagram
    actor User
    participant UI
    participant Comms as Communications Service
    participant InstanceRepo as EmailInstance Repository
    participant Identity
    participant SendGrid
    participant EventBus

    User->>UI: View email details (from history)
    UI->>Comms: GET /api/communications/email-history/{instance_id}
    Comms->>InstanceRepo: Get EmailInstance
    InstanceRepo-->>Comms: Instance details

    alt Email content archived
        Comms-->>UI: Content unavailable (archived)
        Note over UI: Resend button disabled<br/>"Content retention period expired"
    else Content available
        Comms-->>UI: Full instance with content
        UI-->>User: Display email with "Resend" button enabled

        User->>UI: Click "Resend"
        UI->>Comms: GET /api/communications/email-history/{instance_id}/resend-context

        Comms->>Comms: Validate resend permissions
        Comms->>InstanceRepo: Get original recipient user reference
        InstanceRepo-->>Comms: Original recipient (user_id, email)

        alt Recipient has user_id reference
            Comms->>Identity: GET /api/identity/users/{user_id}
            Identity-->>Comms: Current user email

            alt Email updated since original send
                Comms-->>UI: Resend form with hint
                Note over UI: Pre-filled: old@example.com<br/>Hint: "Replace with current:<br/>new@example.com"
            else Email unchanged
                Comms-->>UI: Resend form (original email)
            end
        else No user reference (external recipient)
            Comms-->>UI: Resend form (original email only)
        end

        User->>UI: Modify recipient email (or accept default)
        User->>UI: Click "Send"

        UI->>Comms: POST /api/communications/email-history/{instance_id}/resend
        Note over UI: Request: {recipient_email, cc_emails[]}

        Comms->>Comms: Validate permissions
        Comms->>InstanceRepo: Get original email content
        InstanceRepo-->>Comms: Rendered subject/body

        Comms->>SendGrid: POST /v3/mail/send
        Note over SendGrid: Uses original rendered content<br/>(no re-resolution)
        SendGrid-->>Comms: 202 Accepted (new message_id)

        Comms->>InstanceRepo: Create new EmailInstance
        Note over InstanceRepo: Links to original instance:<br/>resent_from_instance_id
        InstanceRepo-->>Comms: new_instance_id

        Comms--)EventBus: EmailResent {original_instance_id, new_instance_id}
        Comms-->>UI: Success
        UI-->>User: "Email resent" confirmation
    end
```

**Key Decisions:**
- **Content availability:** Resend only available during retention period
- **Email update detection:** System detects if user's email changed in identity system
- **No re-resolution:** Resend uses original rendered content (not re-rendered)
- **New instance created:** Resend creates new EmailInstance linked to original

**State Changes:**
- New EmailInstance status: `None` > `Queued`

**Events Published:**
- `EmailResent` - Links original and new instances

**Error Scenarios:**
- Content archived > 400 Bad Request "Content no longer available"
- Insufficient permissions > 403 Forbidden
- SendGrid error > Display error, allow retry

---

## SendGrid Webhook - Delivery Status Update

**What:** Process incoming delivery events from SendGrid  
**When:** Email delivered, bounced, deferred, or dropped  
**Who:** SendGrid (external system)

```mermaid
---
title: Communications - SendGrid Webhook - Delivery Status Update
---
sequenceDiagram
    participant SendGrid
    participant Webhook as Webhook Endpoint
    participant Validator as Signature Validator
    participant Comms as Communications Service
    participant InstanceRepo as EmailInstance Repository
    participant EventBus

    SendGrid->>Webhook: POST /api/webhooks/sendgrid
    Note over SendGrid: Event: delivered, bounced, etc.<br/>Includes: message_id, timestamp, status

    Webhook->>Validator: Validate HMAC signature
    Note over Validator: Prevents spoofed events<br/>Uses shared secret from Key Vault

    alt Invalid signature
        Validator-->>Webhook: Signature mismatch
        Webhook-->>SendGrid: 401 Unauthorized
    else Valid signature
        Validator-->>Webhook: Signature valid

        Webhook->>Comms: Process delivery event
        Comms->>Comms: Extract message_id, status, substatus
        Comms->>InstanceRepo: Find EmailInstance by message_id

        alt Instance not found
            Comms->>Comms: Log warning (orphaned webhook)
            Note over Comms: Possible race condition:<br/>webhook arrived before instance saved
            Comms-->>Webhook: 200 OK (idempotent)
        else Instance found
            Comms->>InstanceRepo: Check for duplicate event
            Note over InstanceRepo: Idempotency check:<br/>event_id already processed?

            alt Duplicate event
                Comms->>Comms: Log duplicate, skip processing
                Comms-->>Webhook: 200 OK (already processed)
            else New event
                Comms->>InstanceRepo: Add EmailDeliveryEvent
                Note over InstanceRepo: Event: {status, substatus,<br/>timestamp, raw_payload}

                Comms->>InstanceRepo: Update instance delivery_status
                InstanceRepo-->>Comms: Success

                Comms--)EventBus: Publish appropriate event
                Note over EventBus: EmailDelivered, EmailBounced,<br/>EmailDeferred, or EmailDropped

                Comms-->>Webhook: 200 OK
                Webhook-->>SendGrid: 200 OK
            end
        end
    end
```

**Key Decisions:**
- **Signature validation:** HMAC SHA256 prevents spoofed webhooks
- **Idempotency:** Duplicate events ignored (SendGrid may retry)
- **Race condition handling:** Orphaned webhooks logged but accepted
- **Status mapping:** SendGrid statuses mapped to domain events

**State Changes:**
- EmailInstance `delivery_status`: `Queued` > `Delivered/Bounced/Deferred/Dropped`

**Events Published:**
- `EmailDelivered` - Successful delivery
- `EmailBounced` - Permanent failure (includes substatus)
- `EmailDeferred` - Temporary failure (will retry)
- `EmailDropped` - SendGrid rejected before sending

**Error Scenarios:**
- Invalid signature > 401 Unauthorized (prevents processing)
- Instance not found > Log warning, return 200 (idempotent)
- Duplicate event > Return 200 (already processed)

---

## Email Content Purge Job

**What:** Scheduled job permanently deletes old email content while retaining audit metadata  
**When:** Daily at 2:00 AM EST  
**Who:** System (automated)

```mermaid
---
title: Communications - Email Content Purge Job
---
sequenceDiagram
    participant Scheduler as Kubernetes CronJob
    participant Job as Content Purge Job
    participant CosmosDB as Cosmos DB (Email Content)
    participant SQL as SQL Database (Metadata)
    participant EventBus
    participant Monitoring
    
    Scheduler->>Job: Trigger daily job (2:00 AM EST)
    Job->>Job: Load configuration
    Note over Job: content_retention_days: 90<br/>batch_size: 1000
    
    Job->>SQL: Query instances eligible for purge
    Note over SQL: SELECT instance_id, sent_at<br/>FROM email_instances<br/>WHERE sent_at < NOW() - 90 days<br/>AND content_status = 'Active'<br/>LIMIT 1000
    SQL-->>Job: Batch of instance IDs
    
    loop For each instance in batch
        Job->>CosmosDB: DELETE email content document
        Note over CosmosDB: Delete by instance_id<br/>(subject, body_html, body_text)
        
        alt Delete successful
            CosmosDB-->>Job: Success
            
            Job->>SQL: Update instance metadata
            Note over SQL: content_status: Active > Purged<br/>content_purged_at: timestamp
            SQL-->>Job: Success
            
            Job--)EventBus: EmailContentPurged {instance_id, purged_at}
            Job->>Job: Increment success counter
        else Delete failed
            CosmosDB-->>Job: Error
            Job->>Job: Log error, increment failure counter
            Job->>Job: Add to retry queue (max 3 attempts)
        end
    end
    
    Job->>Job: Calculate batch metrics
    Note over Job: Processed: 1000<br/>Succeeded: 998<br/>Failed: 2
    
    alt Failures exceed threshold (>5%)
        Job->>Monitoring: Alert administrators
        Note over Monitoring: Critical: Content purge job failing<br/>Investigate Cosmos DB connectivity
    end
    
    alt More instances to process
        Job->>Job: Schedule next batch (immediate)
    else All instances processed
        Job->>Monitoring: Log completion metrics
        Job-->>Scheduler: Job complete
    end
```

**Key Decisions:**
- **Batch processing:** 1000 instances per batch prevents memory issues
- **Retry logic:** Failed deletes retried up to 3 times
- **Alert threshold:** >5% failure rate triggers admin alert
- **Two-phase delete:** Cosmos DB content deleted first, then SQL metadata updated
- **Metadata preservation:** SQL audit metadata retained indefinitely for compliance

**State Changes:**
- EmailInstance `content_status`: `Active` > `Purged`

**Events Published:**
- `EmailContentPurged` - Per instance successfully purged

**Error Scenarios:**
- Cosmos DB unavailable > Retry batch, alert if persistent
- SQL connection lost > Job fails, retries on next schedule
- Individual delete fails > Retry up to 3 times, then alert
- Cosmos delete succeeds but SQL update fails > Manual reconciliation required (logged)

---

## Template Variable Resolution (Internal)

**What:** Event-type-specific resolver fetches and formats data for template variables  
**When:** Email send triggered (event-driven or manual)  
**Who:** Variable Resolver (internal component)

```mermaid
---
title: Communications - Template Variable Resolution (Internal)
---
sequenceDiagram
    participant Comms as Communications Service
    participant Resolver as CredentialApplicationApprovedResolver
    participant CredAPI as Credentials API
    participant IdentityAPI as Identity API
    participant Cache

    Comms->>Resolver: Resolve variables for event
    Note over Comms: Event: CredentialApplicationApproved<br/>Context: {application_id: 12345}

    Resolver->>Resolver: Parse template variables
    Note over Resolver: Found: <<ApplicantName>>,<br/><<CertificateType>>, <<ApprovalDate>>

    Resolver->>CredAPI: GET /api/credentials/applications/12345
    CredAPI-->>Resolver: Application data
    Note over Resolver: {applicant_id, certificate_type_code,<br/>approved_at, approved_by_id}

    Resolver->>Cache: Check cache for applicant
    Cache-->>Resolver: Cache miss

    Resolver->>IdentityAPI: GET /api/identity/users/{applicant_id}
    IdentityAPI-->>Resolver: User data
    Note over Resolver: {first_name: "Jane", last_name: "Doe",<br/>email: "jane.doe@example.com"}

    Resolver->>Cache: Store user data (TTL: 1 hour)

    Resolver->>Resolver: Format values to user-facing strings
    Note over Resolver: ApplicantName: "Jane Doe"<br/>(NOT "janeDoe" or "JANE DOE")<br/><br/>CertificateType: "Elementary Education"<br/>(NOT "elem_ed" or "ELEM_ED")<br/><br/>ApprovalDate: "January 15, 2026"<br/>(NOT "2026-01-15T10:30:00Z")

    Resolver-->>Comms: Resolved variables map
    Note over Comms: {<br/>  "ApplicantName": "Jane Doe",<br/>  "CertificateType": "Elementary Education",<br/>  "ApprovalDate": "January 15, 2026"<br/>}

    Comms->>Comms: Replace placeholders in template
    Note over Comms: Original:<br/>"Dear <<ApplicantName>>, your<br/>application for <<CertificateType>>..."<br/><br/>Rendered:<br/>"Dear Jane Doe, your application<br/>for Elementary Education..."
```

**Key Decisions:**
- **Event-type-specific resolvers:** Each event type has dedicated resolver class
- **Lightweight events:** Event carries only IDs, resolver fetches display data
- **User-facing formatting:** All values formatted for human readability
- **Caching:** Frequently accessed data (user profiles) cached for 1 hour
- **Batch API calls:** Resolver batches requests when resolving multiple variables from same source

**State Changes:**
- None (internal process)

**Events Published:**
- None (internal process)

**Error Scenarios:**
- API unavailable > Retry with exponential backoff
- Mandatory variable null > Throw exception, block send
- Invalid data format > Log error, use fallback value if available

---

## Dashboard Alert Creation and Display

**What:** System creates dashboard alert for user  
**When:** Important action required (e.g., pending authorization, data quality issue)  
**Who:** System (automated)

```mermaid
---
title: Communications - Dashboard Alert Creation and Display
---
sequenceDiagram
    participant Domain as Domain Service
    participant EventBus
    participant Comms as Communications Service
    participant AlertRepo as Alert Repository
    participant Dashboard as Dashboard UI
    participant User

    Domain--)EventBus: Publish event (e.g., DataQualityIssueDetected)
    EventBus->>Comms: Event received

    Comms->>Comms: Check alert template for event type
    Comms->>Comms: Evaluate alert targeting rules
    Note over Comms: Target: Users with role<br/>"District Data Steward"<br/>at District XYZ

    Comms->>Comms: Resolve alert content variables
    Note over Comms: Alert: "You have 5 data quality<br/>issues requiring attention"

    Comms->>AlertRepo: Create Alert instances
    Note over AlertRepo: One alert per targeted user<br/>Status: Active<br/>Expires: 30 days from now
    AlertRepo-->>Comms: alert_ids[]

    Comms--)EventBus: DashboardAlertCreated {alert_id, user_id}

    Note over User,Dashboard: User logs in

    User->>Dashboard: Access dashboard
    Dashboard->>Comms: GET /api/communications/alerts?user_id={id}&status=Active

    Comms->>Comms: Validate user permissions
    Comms->>AlertRepo: Query active alerts for user
    Note over AlertRepo: SELECT * FROM alerts<br/>WHERE user_id = {id}<br/>AND status = 'Active'<br/>AND (expiration_date IS NULL<br/>  OR expiration_date > NOW())
    AlertRepo-->>Comms: List of alerts

    Comms-->>Dashboard: Alerts with action links
    Dashboard-->>User: Display alert banner/widget
    Note over User: Alert: "5 data quality issues<br/>require your attention"<br/>[View Issues]

    User->>Dashboard: Click alert action link
    Dashboard->>User: Navigate to target page (e.g., data quality dashboard)
```

**Key Decisions:**
- **Role-based targeting:** Alerts sent to users with specific roles at specific scopes
- **Automatic expiration:** Alerts expire after configured period (default 30 days)
- **Action links:** Each alert includes deep link to relevant page
- **One alert per user:** System creates individual alert instances (not shared)

**State Changes:**
- Alert status: `None` > `Active`

**Events Published:**
- `DashboardAlertCreated` - Per user who received alert

**Error Scenarios:**
- No users match targeting rules > Alert template misconfigured, log warning
- User lacks access to action link > Display alert but disable link

---

## User Resolves Dashboard Alert

**What:** User completes action and dismisses alert  
**When:** User clicks alert action link and completes required task  
**Who:** District Data Steward, Authorization Approver, etc.

```mermaid
---
title: Communications - User Resolves Dashboard Alert
---
sequenceDiagram
    actor User
    participant Dashboard
    participant Comms as Communications Service
    participant AlertRepo as Alert Repository
    participant EventBus

    User->>Dashboard: Complete alert action (e.g., fix data quality issue)
    Dashboard->>Comms: POST /api/communications/alerts/{alert_id}/resolve
    Note over Dashboard: Request: {resolution_type: "Completed"}

    Comms->>Comms: Validate user owns alert
    Comms->>AlertRepo: Get alert
    AlertRepo-->>Comms: Alert details

    alt User does not own alert
        Comms-->>Dashboard: 403 Forbidden
    else User owns alert
        Comms->>AlertRepo: Update alert status
        Note over AlertRepo: status: Active > Resolved<br/>resolved_at: timestamp<br/>resolved_by: user_id
        AlertRepo-->>Comms: Success

        Comms--)EventBus: DashboardAlertResolved {alert_id, user_id}
        Comms-->>Dashboard: Success

        Dashboard->>Dashboard: Remove alert from UI
        Dashboard-->>User: Alert dismissed
    end
```

**Key Decisions:**
- **User ownership validation:** Only alert owner can resolve
- **Resolution types:** "Completed", "Dismissed", "No Action Needed"
- **Automatic resolution:** Some alerts auto-resolve when underlying condition cleared
- **Audit trail:** Resolution timestamp and user ID captured

**State Changes:**
- Alert status: `Active` > `Resolved`

**Events Published:**
- `DashboardAlertResolved` - Alert completed by user

**Error Scenarios:**
- User does not own alert > 403 Forbidden
- Alert already resolved > 409 Conflict (idempotent response)
- Alert expired > 400 Bad Request "Alert expired"

---

## Mass Email with Consolidation

**What:** System consolidates multiple related events into single email  
**When:** Multiple data quality issues detected for same user within consolidation window  
**Who:** System (automated)

```mermaid
---
title: Communications - Mass Email with Consolidation
---
sequenceDiagram
    participant DataQuality as Data Quality Service
    participant EventBus
    participant Comms as Communications Service
    participant Consolidator as Event Consolidator
    participant TemplateRepo as Template Repository
    participant Resolver as Variable Resolver
    participant SendGrid
    participant InstanceRepo as EmailInstance Repository

    loop Multiple events over time
        DataQuality--)EventBus: DataQualityIssueDetected (issue_id: 101)
        DataQuality--)EventBus: DataQualityIssueDetected (issue_id: 102)
        DataQuality--)EventBus: DataQualityIssueDetected (issue_id: 103)
    end

    EventBus->>Comms: Events received
    Comms->>Consolidator: Buffer events
    Note over Consolidator: Consolidation window: 15 minutes<br/>Group by: recipient_user_id

    Consolidator->>Consolidator: Wait for window to close
    Note over Consolidator: Timer started on first event

    Note over Consolidator: 15 minutes elapsed

    Consolidator->>Consolidator: Group events by recipient
    Note over Consolidator: User_123: [101, 102, 103]<br/>User_456: [104, 105]

    loop For each recipient group
        Consolidator->>TemplateRepo: Get consolidation template
        Note over TemplateRepo: Template: "Data Quality Issues Summary"
        TemplateRepo-->>Consolidator: Template

        Consolidator->>Resolver: Resolve variables
        Note over Resolver: <<IssueCount>>: "3"<br/><<IssueList>>: formatted list of issues

        Resolver->>DataQuality: GET /api/data-quality/issues?ids=101,102,103
        DataQuality-->>Resolver: Issue details

        Resolver->>Resolver: Format as user-facing list
        Note over Resolver: "1. Missing certification date<br/>2. Invalid endorsement code<br/>3. Duplicate employee record"

        Resolver-->>Consolidator: Resolved variables

        Consolidator->>Consolidator: Render email
        Note over Consolidator: "You have 3 data quality issues<br/>requiring your attention:<br/><br/>1. Missing certification date..."

        Consolidator->>SendGrid: POST /v3/mail/send
        SendGrid-->>Consolidator: 202 Accepted

        Consolidator->>InstanceRepo: Store EmailInstance
        Note over InstanceRepo: Links to all source events:<br/>consolidated_event_ids: [101,102,103]
        InstanceRepo-->>Consolidator: instance_id

        Consolidator--)EventBus: EmailSent (consolidated: true)
    end
```

**Key Decisions:**
- **Consolidation window:** 15 minutes default (configurable per event type)
- **Grouping key:** Typically recipient user, but can be customized
- **Special templates:** Consolidation templates support list variables
- **Event tracking:** EmailInstance links to all consolidated event IDs

**State Changes:**
- EmailInstance status: `None` > `Queued`

**Events Published:**
- `EmailSent` with `consolidated: true` flag

**Error Scenarios:**
- Consolidation window timeout > Process buffered events even if incomplete
- Template not found > Fall back to individual emails per event

---

## Integration Flows

### Event Type Variable Registry API

**Purpose:** Backend provides UI with available variables for each event type

**API Endpoint:**
```
GET /api/communications/event-types/{event_type}/available-variables
```

**Example Response:**
```json
{
  "event_type": "CredentialApplicationApproved",
  "functional_area": "Credentialing",
  "available_variables": [
    {
      "name": "ApplicantName",
      "display_name": "Applicant Name",
      "description": "Full name of the credential applicant",
      "example_value": "Jane Doe",
      "is_mandatory": true,
      "data_type": "string"
    },
    {
      "name": "CertificateType",
      "display_name": "Certificate Type",
      "description": "Type of certificate being applied for",
      "example_value": "Elementary Education",
      "is_mandatory": true,
      "data_type": "string"
    },
    {
      "name": "ApprovalDate",
      "display_name": "Approval Date",
      "description": "Date the application was approved",
      "example_value": "January 15, 2026",
      "is_mandatory": false,
      "data_type": "string"
    },
    {
      "name": "ApplicationId",
      "display_name": "Application ID",
      "description": "Unique identifier for the application",
      "example_value": "APP-2026-12345",
      "is_mandatory": false,
      "data_type": "string"
    }
  ]
}
```

**Usage in UI:**
1. User creates/edits template
2. User selects event type from dropdown
3. UI calls this endpoint to fetch available variables
4. Variable picker populated with context-appropriate options
5. User inserts variables into template content using `<<VariableName>>` syntax

---

### SendGrid Integration - Email Delivery

**Purpose:** Send email via SendGrid and track delivery status

**Send Email:**
```
POST https://api.sendgrid.com/v3/mail/send
Authorization: Bearer {API_KEY}

{
  "personalizations": [{
    "to": [{"email": "jane.doe@example.com", "name": "Jane Doe"}],
    "subject": "Your credential application has been approved"
  }],
  "from": {"email": "noreply@michigan.gov", "name": "MiEdWorkforce"},
  "content": [{
    "type": "text/html",
    "value": "<p>Dear Jane Doe, your application for Elementary Education...</p>"
  }],
  "custom_args": {
    "instance_id": "uuid-12345",
    "functional_area": "Credentialing"
  }
}
```

**Response:**
```
202 Accepted
X-Message-Id: abc123def456
```

**Webhook Configuration:**
- **Endpoint:** `https://miedworkforce.michigan.gov/api/webhooks/sendgrid`
- **Events:** `delivered`, `bounce`, `deferred`, `dropped`, `processed`
- **Signature Verification:** HMAC SHA256 using shared secret

**Webhook Payload Example:**
```json
{
  "email": "jane.doe@example.com",
  "timestamp": 1737044400,
  "event": "delivered",
  "sg_message_id": "abc123def456",
  "instance_id": "uuid-12345"
}
```

---

### Variable Resolver Registry

**Purpose:** Map event types to their corresponding resolver implementations

**Resolver Interface:**
```typescript
interface IVariableResolver {
  eventType: string;
  availableVariables: VariableDefinition[];
  resolve(context: EventContext): Promise<Map<string, string>>;
}
```

**Resolver Registration:**
```typescript
// Registry pattern
const resolverFactory = new VariableResolverFactory();

resolverFactory.register(
  'CredentialApplicationApproved',
  new CredentialApplicationApprovedResolver()
);

resolverFactory.register(
  'StaffingRecordSubmitted',
  new StaffingRecordSubmittedResolver()
);

// Usage
const resolver = resolverFactory.get('CredentialApplicationApproved');
const variables = await resolver.resolve({ application_id: '12345' });
```

**Resolver Implementation Example:**
```typescript
class CredentialApplicationApprovedResolver implements IVariableResolver {
  eventType = 'CredentialApplicationApproved';
  
  availableVariables = [
    { name: 'ApplicantName', mandatory: true },
    { name: 'CertificateType', mandatory: true },
    { name: 'ApprovalDate', mandatory: false }
  ];
  
  async resolve(context: EventContext): Promise<Map<string, string>> {
    const application = await this.credentialsApi.getApplication(context.application_id);
    const user = await this.identityApi.getUser(application.applicant_id);
    
    return new Map([
      ['ApplicantName', this.formatName(user)],
      ['CertificateType', this.formatCertificateType(application.certificate_type_code)],
      ['ApprovalDate', this.formatDate(application.approved_at)]
    ]);
  }
  
  private formatName(user: User): string {
    return `${user.first_name} ${user.last_name}`;
  }
  
  private formatCertificateType(code: string): string {
    // Lookup display name from reference data
    return this.certificateTypeMap.get(code) || code;
  }
  
  private formatDate(date: Date): string {
    return date.toLocaleDateString('en-US', { 
      year: 'numeric', 
      month: 'long', 
      day: 'numeric' 
    });
  }
}
```

---

# Notes

---

## Template Variable Formatting Standards

All variable resolvers must produce user-facing formatted strings. Raw technical values (database codes, ISO timestamps, boolean flags) should never appear in rendered email content.

- **Dates:** Long format — "January 15, 2026"
- **Names:** "First Last" — never all-caps or last-name-first
- **Currency:** "$1,234.56"
- **Credential/Certificate Types:** Human-readable display name — "Elementary Education", never "elem_ed" or "ELEMENTARY_EDUCATION"
- **Boolean values:** Context-appropriate phrasing — "Yes / No" or "Approved / Denied"
- **Lists:** Numbered for ordered items, bulleted for unordered; use HTML lists in HTML emails

---

## Event Payload Design Standards

Domain events that trigger communications should carry only IDs and intrinsic core data. Display data (names, labels, formatted values) is the resolver's responsibility, not the event publisher's.

This keeps events lightweight, prevents payload bloat, and ensures emails always use current data at send time rather than data captured when the event was published.

---

## Resend vs. New Send Decision Matrix

| Scenario                           | Action                      | Rationale                                            |
| ---------------------------------- | --------------------------- | ---------------------------------------------------- |
| Email bounced (invalid address)    | Resend to corrected address | Content is correct, just wrong recipient             |
| User claims "never received"       | Resend to same address      | May have been filtered to spam                       |
| Email content has errors           | Send new email              | Resend uses original rendered content                |
| New information to communicate     | Send new email              | Resend is retransmission only, not an update         |
| Forwarding to manager or help desk | Resend with CC              | Shares the exact content originally sent             |
| Content archived (90+ days)        | Send new email              | Resend unavailable after retention period            |
| Template has been updated          | Send new email              | Resend uses the version active at original send time |
| Underlying data has changed        | Send new email              | Resend does not re-resolve variables                 |
