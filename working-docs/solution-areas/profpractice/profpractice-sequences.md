# Professional Practice Review - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Professional Practice Review domain.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human only
- **participant** = Every non-human: internal service, component or external system
- Every request arrow starts with an API-kind tag, then the verb and path from the API catalog: `APP` = Application API (user token, UI to owning API), `SVC` = Service API (in-cluster, API to API), `EXT` = External API (inbound call from an external system or provider), `OUT` = outbound call from a service to an external system
- Every application API call is authorized by the owning service through the cached IAM permission check (Service API). It is not drawn unless noted.

---

## Submit Self-Disclosure

**What:** Educator voluntarily reports a new criminal conviction or professional disciplinary action  
**When:** Educator receives conviction or disciplinary action and must report within required timeframe  
**Who:** Authenticated educator. Permission: `profpractice.disclosure.submit` (self-only)

```mermaid
---
title: Professional Practice Review - Submit Self-Disclosure
---
sequenceDiagram
    actor Educator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant DocsApi as Documents API
        participant EventBus as Event Bus
    end
    
    Educator->>UI: Navigate to "Report Disclosure"
    UI->>PprApi: APP GET /disclosure-questions
    PprApi-->>UI: Disclosure question configuration
    
    Educator->>UI: Select disclosure type (Misdemeanor/Felony/Disciplinary)
    UI-->>Educator: Display type-specific questions
    
    Educator->>UI: Enter conviction date, description
    Educator->>UI: Upload supporting documents (court orders, etc.)
    UI->>PprApi: APP POST /upload
    PprApi->>DocsApi: SVC POST /documents/upload/request
    DocsApi-->>PprApi: Document IDs
    PprApi-->>UI: Document IDs
    
    Educator->>UI: Submit disclosure
    UI->>PprApi: APP POST /disclosures
    
    PprApi->>PprApi: Create Disclosure aggregate
    Note over PprApi: State: Pending Review
    Note over PprApi: SourceType: SelfDisclosed
    
    PprApi->>PprApi: Route to OEE worklist based on type
    
    PprApi--)EventBus: DisclosureSubmitted
    PprApi--)EventBus: DisclosureRoutedToWorklist
    
    PprApi-->>UI: Disclosure ID
    UI-->>Educator: Confirmation - under review
```

**Key Decisions:**
- **Document Requirements:** At least one supporting document required for self-disclosures
- **Automatic Routing:** System routes to OEE worklist based on disclosure type
- **Conviction Date:** Cannot be future-dated

**State Changes:**
- Disclosure: `None` > `Pending Review`

**Events Published:**
- `DisclosureSubmitted` - Triggers communications to educator and PPR staff
- `DisclosureRoutedToWorklist` - Recorded for audit; routing/queue placement itself is handled synchronously by the PPRWorklist aggregate, not a downstream capability

**Error Scenarios:**
- No documents attached > Block submission, require at least one document
- Conviction date in future > Validation error
- Description exceeds 5000 characters > Validation error

---

## Submit Professional Practice Review Response

**What:** Educator completes annual PPR compliance check, potentially reporting new disclosures  
**When:** Annual PPR due date approaching or past due  
**Who:** Authenticated educator. Permission: `profpractice.response.submit` (self-only)

```mermaid
---
title: Professional Practice Review - Submit PPR Response
---
sequenceDiagram
    actor Educator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant DocsApi as Documents API
        participant EventBus as Event Bus
    end
    
    Educator->>UI: Navigate to "Complete Annual PPR"
    UI->>PprApi: APP GET /ppr-questions
    PprApi-->>UI: Current PPR question configuration
    
    Educator->>UI: Answer all mandatory questions
    Note over Educator,UI: Questions include:<br/>- New criminal convictions?<br/>- New disciplinary actions?<br/>- Pending charges?
    
    alt Any answer is "Yes"
        UI-->>Educator: Display disclosure detail forms
        
        Educator->>UI: Enter details for each "Yes" answer
        Educator->>UI: Upload supporting documents
        UI->>PprApi: APP POST /upload
        PprApi->>DocsApi: SVC POST /documents/upload/request
        DocsApi-->>PprApi: Document IDs
        PprApi-->>UI: Document IDs
    end
    
    Educator->>UI: Submit PPR response
    UI->>PprApi: APP POST /ppr-responses
    
    PprApi->>PprApi: Create ProfessionalPracticeResponse aggregate
    
    alt New disclosures reported (any "Yes" answers)
        loop For each "Yes" answer
            PprApi->>PprApi: Create Disclosure aggregate
            Note over PprApi: SourceType: PPRResponse
            
            PprApi->>PprApi: Route disclosure to OEE worklist
            PprApi--)EventBus: DisclosureSubmitted
            PprApi--)EventBus: DisclosureRoutedToWorklist
        end
        
        PprApi->>PprApi: Link disclosures to response
        Note over PprApi: Response: NewDisclosuresReported = true
    else All "No" answers
        Note over PprApi: Response: NewDisclosuresReported = false<br/>Response: ResponseStatus = No Review Needed
    end
    
    Note over PprApi: EducatorPPRStatus.LastPPRResponseDate updated
    
    PprApi--)EventBus: PPRResponseSubmitted
    
    PprApi-->>UI: Response ID
    UI-->>Educator: Confirmation - compliance updated
```

**Key Decisions:**
- **Disclosure Creation:** System automatically creates Disclosure records for each "Yes" answer
- **Compliance Tracking:** LastPPRResponseDate updated regardless of whether new disclosures reported
- **Routing:** New disclosures route to OEE worklist same as self-disclosures

**State Changes:**
- ProfessionalPracticeResponse: `None` > `Pending Review` or `No Review Needed`
- Disclosure: `None` > `Pending Review` (for each "Yes" answer)
- EducatorPPRStatus: LastPPRResponseDate updated

**Events Published:**
- `PPRResponseSubmitted` - Records compliance activity
- `DisclosureSubmitted` - One per new disclosure
- `DisclosureRoutedToWorklist` - One per new disclosure

**Error Scenarios:**
- Not all mandatory questions answered > Block submission
- "Yes" answer without supporting details > Validation error

---

## Review Disclosure and Update Status

**What:** PPR staff reviews disclosure details and updates status through workflow  
**When:** Disclosure appears in reviewer's worklist  
**Who:** User with `profpractice.disclosure.update-status` permission assigned to worklist. Permission: `profpractice.status.update` (worklist-specific), with `profpractice.worklist.view` and `profpractice.disclosure.view` for the reads, `profpractice.disclosure.edit` for the optional edit and `profpractice.remarks.add` for remarks

```mermaid
---
title: Professional Practice Review - Review Disclosure and Update Status
---
sequenceDiagram
    actor Reviewer
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant DocsApi as Documents API
        participant EventBus as Event Bus
    end
    
    Reviewer->>UI: View assigned worklist
    UI->>PprApi: APP GET /worklists/{worklistId}/disclosures
    PprApi-->>UI: Queued disclosures
    
    Reviewer->>UI: Select disclosure to review
    UI->>PprApi: APP GET /disclosures/{disclosureId}
    PprApi->>DocsApi: SVC GET /documents/search
    DocsApi-->>PprApi: Supporting documents
    PprApi-->>UI: Disclosure details with supporting documents
    
    Note over Reviewer,UI: Reviewer examines:<br/>- Conviction details<br/>- Court documents<br/>- Prior disclosure history
    
    opt Need to edit disclosure details
        Reviewer->>UI: Edit conviction date, type, or description
        UI->>PprApi: APP PUT /disclosures/{disclosureId}
        PprApi->>PprApi: Update Disclosure aggregate
        Note over PprApi: Only allowed while status = "Under PPR Review"
    end
    
    Reviewer->>UI: Add internal remarks (staff notes)
    UI->>PprApi: APP POST /disclosures/{disclosureId}/add-remark
    PprApi->>PprApi: Add InternalRemarks
    
    Reviewer->>UI: Select new status (e.g., "Reviewed - Misdemeanor")
    UI->>PprApi: APP POST /disclosures/{disclosureId}/update-status
    
    alt Invalid status transition
        PprApi-->>UI: 400 Validation error
    else Valid status transition
        PprApi->>PprApi: Update Disclosure aggregate
        Note over PprApi: Previous: Under PPR Review<br/>New: Reviewed - Misdemeanor
        
        PprApi--)EventBus: DisclosureStatusChanged
        PprApi--)EventBus: DisclosureReviewCompleted (if "Reviewed" status)
        PprApi--)EventBus: PPRClearanceAssessmentChanged
        
        PprApi-->>UI: Status updated
    end
```

**Key Decisions:**
- **Edit Window:** Can only edit disclosure details while status = "Under PPR Review"
- **Status Transitions:** Must follow valid workflow paths (e.g., cannot go directly from "Pending Review" to "Reviewed - Felony")
- **Assessment Recalculation:** Status changes trigger clearance assessment recalculation

**State Changes:**
- Disclosure: `Under PPR Review` > `Reviewed - [Outcome]`

**Events Published:**
- `DisclosureStatusChanged` - Status transition recorded
- `DisclosureReviewCompleted` - Finalization to "Reviewed" status
- `PPRClearanceAssessmentChanged` - Triggers re-evaluation in Credentialing/Staffing domains

**Error Scenarios:**
- Invalid status transition > Validation error
- Attempt to edit finalized disclosure > Access denied (must revert first)

---

## Manage Account Markers

**What:** PPR Admin sets or clears account-level markers on educator  
**When:** Administrative decision requires heightened oversight or processing restrictions  
**Who:** User with marker-specific permissions (PPR Admin or Credentialing Admin). Permission: `profpractice.markers.manage` (system-wide), with `profpractice.markers.view` and `profpractice.disclosure.view` for the educator lookup

```mermaid
---
title: Professional Practice Review - Manage Account Markers
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant EventBus as Event Bus
    end
    
    Admin->>UI: Search for educator
    UI->>PprApi: APP GET /educators/search
    PprApi-->>UI: Educator results
    
    Admin->>UI: Select educator
    UI->>PprApi: APP GET /educators/{educatorId}/ppr-status
    PprApi-->>UI: Current account markers and disclosure history
    
    Admin->>UI: Select marker action
    Note over Admin,UI: Options:<br/>- Set/Clear Mandatory Hold Requirement<br/>- Set/Clear Enhanced Monitoring Status<br/>- Set/Clear Re-Review Requirement
    
    Admin->>UI: Enter justification for marker change
    
    UI->>PprApi: APP POST /educators/{educatorId}/markers/mandatory-hold
    Note over UI,PprApi: Representative path (set Mandatory Hold).<br/>Other marker actions: see Marker Actions table
    
    PprApi->>PprApi: Update EducatorPPRStatus aggregate
    Note over PprApi: Marker flag changed<br/>MarkerReason recorded<br/>MarkerHistory audited
    
    opt Set Re-Review Requirement only
        PprApi->>PprApi: Re-route previously reviewed disclosures to worklist
        PprApi--)EventBus: DisclosureRoutedToWorklist (for each re-routed disclosure)
    end
    
    PprApi--)EventBus: PPRAccountMarkerChanged (markerType)
    PprApi--)EventBus: PPRClearanceAssessmentChanged (not published for Set Re-Review)
    
    PprApi-->>UI: Marker updated
    UI-->>Admin: Confirmation with audit trail
```

**Marker Actions:**

| Action | Request | Operation | Effect | Events |
|---|---|---|---|---|
| Set Mandatory Hold Requirement | `APP POST /educators/{educatorId}/markers/mandatory-hold` | `setMandatoryHold` | MandatoryHoldRequirement = true, MarkerReason recorded, MarkerHistory audited | `PPRAccountMarkerChanged` (markerType: MandatoryHold), `PPRClearanceAssessmentChanged` |
| Clear Mandatory Hold Requirement | `APP DELETE /educators/{educatorId}/markers/mandatory-hold` | `clearMandatoryHold` | MandatoryHoldRequirement = false | `PPRAccountMarkerChanged`, `PPRClearanceAssessmentChanged` |
| Set Enhanced Monitoring Status | `APP POST /educators/{educatorId}/markers/enhanced-monitoring` | `setEnhancedMonitoring` | Marker updated on EducatorPPRStatus | `PPRAccountMarkerChanged` (markerType: EnhancedMonitoring), `PPRClearanceAssessmentChanged` |
| Clear Enhanced Monitoring Status | `APP DELETE /educators/{educatorId}/markers/enhanced-monitoring` | `clearEnhancedMonitoring` | Not drawn in the original diagram | Not specified in the original diagram |
| Set Re-Review Requirement | `APP POST /educators/{educatorId}/markers/rereview` | `setReReview` | Marker updated, previously reviewed disclosures re-routed to worklist | `PPRAccountMarkerChanged` (markerType: ReReview), `DisclosureRoutedToWorklist` (for each re-routed disclosure) |
| Clear Re-Review Requirement | `APP DELETE /educators/{educatorId}/markers/rereview` | `clearReReview` | Not drawn in the original diagram | Not specified in the original diagram |

**Key Decisions:**
- **Justification Required:** All marker changes require reason text for audit trail
- **Permission Segregation:** Different permissions for setting vs. clearing markers
- **Immediate Effect:** Marker changes immediately affect clearance assessments and worklist routing

**State Changes:**
- EducatorPPRStatus: Marker flags toggled (MandatoryHoldRequirement, EnhancedMonitoringStatus, ReReviewRequirement)
- Disclosure: May be re-routed to worklist if Re-Review Requirement activated

**Events Published:**
- `PPRAccountMarkerChanged` - Marker state change
- `PPRClearanceAssessmentChanged` - Triggers downstream re-evaluation
- `DisclosureRoutedToWorklist` - If Re-Review Requirement forces disclosures back to review

**Error Scenarios:**
- Admin lacks permission > Access denied
- Justification missing > Validation error

---

## Receive Rap Back Notification

**What:** External Rap Back system sends notification of new criminal activity via webhook  
**When:** Michigan State Police detects new arrest or prosecution for enrolled educator  
**Who:** External system (MSP Rap Back). Permission: none (External API, client credentials, no user permission)

```mermaid
---
title: Professional Practice Review - Receive Rap Back Notification
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant Webhook as PPR Webhook Endpoint
        participant PprApi as PPR API
        participant EventBus as Event Bus
    end
    box External
        participant MspRapBack as MSP CHRISS Rap Back
        participant MiKey as Mi-Key
    end
    
    MspRapBack->>Webhook: EXT POST /webhooks/rapback
    Note over MspRapBack,Webhook: Payload includes:<br/>- TCN (Transaction Control Number)<br/>- NotificationDate<br/>- JudicialMarker<br/>- PIC (Person Identifier) [optional]
    
    Webhook->>PprApi: SVC Process Rap Back notification
    
    alt PIC field is empty
        PprApi->>MiKey: OUT POST /match-person
        MiKey-->>PprApi: Unique ID match
        Note over PprApi: If no match found: log unmatched notification,<br/>cannot create disclosure without educator match,<br/>acknowledge with 200 OK (see Error Scenarios)
    else PIC field populated
        Note over PprApi: Use PIC to identify educator
    end
    
    PprApi->>PprApi: Create ExternalBackgroundCheck aggregate
    Note over PprApi: CheckSource: RapBack<br/>State: Received<br/>RequiresEducatorResponse: true
    
    PprApi->>PprApi: Create placeholder Disclosure aggregate
    Note over PprApi: SourceType: RapBackTriggered<br/>State: Under PPR Review<br/>Description: "Pending educator response"
    
    PprApi->>PprApi: Route disclosure to OEE worklist
    
    PprApi--)EventBus: RapBackNotificationReceived
    PprApi--)EventBus: DisclosureSubmitted
    PprApi--)EventBus: DisclosureRoutedToWorklist
    PprApi--)EventBus: PPRClearanceAssessmentChanged
    
    Note over EventBus: Communications consumes events:<br/>- Sends email to educator requiring response<br/>- Alerts PPR staff of new notification
```

**Key Decisions:**
- **Person Matching:** If PIC empty, use Mi-Key to match SSN+DOB+Name to Unique ID
- **Placeholder Disclosure:** System creates disclosure requiring educator to provide details
- **Automatic Routing:** Routes to OEE worklist for initial review

**State Changes:**
- ExternalBackgroundCheck: `None` > `Received`
- Disclosure: `None` > `Under PPR Review` (placeholder)

**Events Published:**
- `RapBackNotificationReceived` - External data received
- `DisclosureSubmitted` - Placeholder disclosure created
- `DisclosureRoutedToWorklist` - Routed to OEE for review
- `PPRClearanceAssessmentChanged` - Active disclosure affects clearance

**Error Scenarios:**
- Person match fails > Log unmatched notification, return 200 OK (acknowledge receipt)
- Duplicate notification (same TCN) > Update existing record, do not create duplicate

---

## Retrieve Full Rap Sheet for Review

**What:** PPR reviewer initiates on-demand retrieval of complete rap sheet from CHRISS  
**When:** Reviewer needs detailed criminal history beyond notification metadata  
**Who:** User with `profpractice.rapback.retrieve-rapsheet` permission. Permission: `profpractice.rapsheet.retrieve` (system-wide), with `profpractice.rapback.view` for the metadata read

```mermaid
---
title: Professional Practice Review - Retrieve Full Rap Sheet
---
sequenceDiagram
    actor Reviewer
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
    end
    box External
        participant MspRapBack as MSP CHRISS Rap Back
    end
    
    Reviewer->>UI: Review Rap Back notification
    UI->>PprApi: APP GET /external-checks/{checkId}
    PprApi-->>UI: Rap Back metadata (TCN, judicial marker)
    
    Reviewer->>UI: Click "View Full Rap Sheet"
    UI->>PprApi: APP POST /rapback/retrieve-rapsheet
    
    PprApi->>MspRapBack: OUT SOAP GET_RAPBACK
    MspRapBack-->>PprApi: Full rap sheet text
    
    Note over PprApi: **CRITICAL: Rap sheet content<br/>is NEVER stored in database**
    
    PprApi->>PprApi: Log retrieval action in ExternalBackgroundCheck
    Note over PprApi: RapSheetRetrievalLog: {user, timestamp}
    
    PprApi-->>UI: Rap sheet text (transient)
    UI-->>Reviewer: Display rap sheet in modal (read-only)
    
    Note over Reviewer,UI: Reviewer reviews details,<br/>makes notes in disclosure remarks,<br/>updates disclosure status
    
    Note over UI: When modal closed,<br/>rap sheet content discarded from memory
```

**Key Decisions:**
- **Display-Only:** Rap sheet is NEVER persisted; only displayed transiently
- **Audit Logging:** System logs fact that retrieval occurred (who, when) but not content
- **On-Demand:** Retrieval only happens when reviewer explicitly requests it

**State Changes:**
- ExternalBackgroundCheck: RapSheetRetrievalLog updated (audit only)

**Events Published:**
- None (audit log entry only)

**Error Scenarios:**
- CHRISS API unavailable > Display error, allow retry
- Invalid TCN > Display error message from CHRISS
- Network timeout > Display error, suggest contacting support

---

## Process NASDTEC Nightly Batch

**What:** Scheduled job pulls data from NASDTEC API and matches to Michigan educators  
**When:** Nightly at configured time (e.g., 2:00 AM)  
**Who:** System (scheduled job). Permission: `profpractice.nasdtec.view` (Service API call from the scheduler)

```mermaid
---
title: Professional Practice Review - Process NASDTEC Nightly Batch
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant Scheduler as Job Scheduler
        participant PprApi as PPR API
        participant EventBus as Event Bus
    end
    box External
        participant NASDTEC as NASDTEC Clearinghouse
        participant MiKey as Mi-Key
    end
    
    Scheduler->>PprApi: SVC POST /admin/jobs/process-nasdtec-batch
    
    PprApi->>NASDTEC: OUT GET /v1/clearinghouse/people
    NASDTEC-->>PprApi: List of educators with TransactionDate in current month
    
    Note over PprApi: Request uses currmonth=YYYYMM. Response includes:<br/>- LastName, FirstName, MiddleName, SuffixName<br/>- BirthDate<br/>- CertificationID (SSN)<br/>- Jurisdiction<br/>- TransactionDate<br/>- ClearinghouseId<br/>- ClearinghouseUrl
    
    loop For each record not already processed (ClearinghouseId)
        PprApi->>MiKey: OUT POST /match-person
        MiKey-->>PprApi: Unique ID match
        
        alt Match found
            PprApi->>PprApi: Create ExternalBackgroundCheck and store NASDTEC record locally
            Note over PprApi: CheckSource: NASDTEC<br/>State: Received<br/>RequiresReview: true<br/>Stored fields: LastName, FirstName, BirthDate,<br/>CertificationID, Jurisdiction,<br/>ClearinghouseId, ClearinghouseUrl
            
            PprApi->>PprApi: Route to PPR worklist
            
            PprApi--)EventBus: NASDTECRecordMatched
            
        else No match found
            PprApi->>PprApi: Log unmatched record
            Note over PprApi: Store for manual review if needed
        end
    end
    
    PprApi->>PprApi: Generate batch summary report
    Note over PprApi: Report: {total records, matched, unmatched, errors}
    
    PprApi-->>Scheduler: Batch complete
```

**Key Decisions:**
- **Incremental Pull:** Pull `currmonth` nightly to get recent transactions
- **Local Storage:** NASDTEC records stored locally for search/reference
- **Person Matching:** Use CertificationID (SSN) + BirthDate + Name to match via Mi-Key
- **Duplicate Prevention:** Check ClearinghouseId to avoid re-processing same record

**State Changes:**
- ExternalBackgroundCheck: `None` > `Received` (for matched records)

**Events Published:**
- `NASDTECRecordMatched` - One per successful match

**Error Scenarios:**
- NASDTEC API unavailable > Retry with exponential backoff, alert admin if persistent failure
- Mi-Key API unavailable > Skip person matching, log for manual follow-up
- Malformed API response > Log error, skip record

---

## Evaluate PPR Clearance for Credential Application

**What:** Credentialing domain requests PPR clearance assessment for application  
**When:** Educator submits credential application or application status changes  
**Who:** Credentialing domain (API consumer). Permission: `profpractice.clearance.view` (Service API call)

```mermaid
---
title: Professional Practice Review - Evaluate PPR Clearance
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant CredApi as Credentialing API
        participant PprApi as PPR API
    end
    
    CredApi->>PprApi: SVC GET /educators/{educatorId}/ppr-clearance
    Note over PprApi: Evaluates conditions in priority order<br/>(see Decision Table)
    PprApi-->>CredApi: PPRClearanceAssessment value object
    
    Note over CredApi: Credentialing uses assessment to:<br/>- Block/hold/flag applications<br/>- Display district notifications<br/>- Require felony acknowledgment<br/>Final credential approval decision<br/>combines PPR + other factors
```

**Decision Table:**

PPR API loads the EducatorPPRStatus aggregate and all active and finalized disclosures, then evaluates in priority order. The first matching row applies.

| Priority | Condition | ClearanceStatus | Assessment details |
|---|---|---|---|
| 1 | Mandatory Hold Requirement active | Hold | BlockingReason: "Mandatory Hold Requirement active"; DistrictNotifications: [hold message] |
| 2 | "Reviewed - Listed" disclosure exists | Blocked | BlockingReason: "Enumerated offense on record"; DistrictNotifications: [blocked messages] |
| 3 | "Reviewed - Felony" disclosure exists | ConditionalClearance | RequiresFelonyAcknowledgment: true; DistrictNotifications: [felony form message] |
| 4 | Active disclosure ("Under PPR Review", "PPR Hold", etc.) | Hold | BlockingReason: "Disclosure under active review" |
| 5 | Other reviewed disclosures | Not specified in the original diagram | Not specified in the original diagram |
| 6 | Enhanced Monitoring Status active | ConditionalClearance | ReviewTriggers: ["EnhancedMonitoringStatusActive"] |
| 7 | No PPR concerns | Clear | None |

**Key Decisions:**
- **Calculated On-Demand:** Assessment is never cached; computed fresh on each request
- **Priority Order:** Most restrictive condition takes precedence
- **Not Final Decision:** PPR provides assessment; Credentialing makes approval/denial decision

**State Changes:**
- None (read-only query)

**Events Published:**
- None (synchronous query)

**Error Scenarios:**
- Educator not found > Return 404
- Database unavailable > Return 503, Credentialing should retry

---

## Evaluate Roster Eligibility for Employment

**What:** Staffing domain requests PPR eligibility assessment before roster addition  
**When:** District HR attempts to add educator to employment roster  
**Who:** Staffing domain (API consumer). Permission: `profpractice.eligibility.view` (Service API call)

```mermaid
---
title: Professional Practice Review - Evaluate Roster Eligibility
---
sequenceDiagram
    box MiEdWorkforce (AKS)
        participant StaffApi as Staffing API
        participant PprApi as PPR API
    end
    
    StaffApi->>PprApi: SVC GET /educators/{educatorId}/roster-eligibility
    Note over PprApi: Evaluates conditions in priority order<br/>(see Decision Table)
    PprApi-->>StaffApi: RosterEligibilityAssessment value object
    
    Note over StaffApi: Staffing uses assessment to:<br/>- Block roster addition if Ineligible<br/>- Display district notifications<br/>- Allow addition with warnings if EligibleWithNotification
```

**Decision Table:**

PPR API loads the EducatorPPRStatus aggregate and all active and finalized disclosures, then evaluates in priority order. The first matching row applies.

| Priority | Condition | EligibilityStatus | Assessment details |
|---|---|---|---|
| 1 | "Reviewed - Listed" disclosure exists | Ineligible | BlockingReason: "Enumerated offense"; DistrictNotifications: [employment prohibition] |
| 2 | Active disclosure ("Under PPR Review", etc.) | Ineligible | BlockingReason: "Review in progress" |
| 3 | "Reviewed - Felony" disclosure exists | EligibleWithNotification | DistrictNotifications: [felony acknowledgment required] |
| 4 | No PPR concerns | Eligible | None |

**Key Decisions:**
- **Stricter Than Credentialing:** Employment eligibility has absolute blocks (Listed offenses, active reviews)
- **Calculated On-Demand:** Never cached
- **District Notification:** EligibleWithNotification requires district acknowledgment but allows employment

**State Changes:**
- None (read-only query)

**Events Published:**
- None (synchronous query)

**Error Scenarios:**
- Educator not found > Return 404
- Database unavailable > Return 503

---

## Log Non-System Action

**What:** PPR reviewer documents external activities (phone calls, emails, manual document reviews)  
**When:** Reviewer takes action outside the system that should be audited  
**Who:** User with `profpractice.disclosure.log-non-system-action` permission. Permission: `profpractice.actions.log` (system-wide or worklist-specific)

```mermaid
---
title: Professional Practice Review - Log Non-System Action
---
sequenceDiagram
    actor Reviewer
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant EventBus as Event Bus
    end
    
    Reviewer->>UI: Review disclosure
    Note over Reviewer: Reviewer takes external action:<br/>- Calls educator for clarification<br/>- Emails school district<br/>- Reviews external court records<br/>- Refers case to specialist
    
    Reviewer->>UI: Click "Log Non-System Action"
    UI-->>Reviewer: Display action logging form
    
    Reviewer->>UI: Select action type
    Note over Reviewer,UI: Options:<br/>- NotifiedSchoolDistrict<br/>- ReferredToPPR<br/>- NotifiedMSPRemoval<br/>- ContactedEducator<br/>- ReviewedExternalDocuments
    
    Reviewer->>UI: Enter action description
    Reviewer->>UI: Submit log entry
    
    UI->>PprApi: APP POST /disclosures/{disclosureId}/log-action
    
    PprApi->>PprApi: Update Disclosure aggregate
    Note over PprApi: NonSystemActions: [<br/>  { actionType, description,<br/>    loggedBy, loggedAt }<br/>]
    
    PprApi--)EventBus: NonSystemActionLogged
    
    PprApi-->>UI: Action logged
    UI-->>Reviewer: Confirmation
```

**Key Decisions:**
- **Audit Trail:** All non-system actions recorded for compliance and appeals
- **Free-Form Description:** Allows flexible documentation of varied activities
- **No State Change:** Logging action does not change disclosure status

**State Changes:**
- Disclosure: NonSystemActions collection updated (audit only, no state change)

**Events Published:**
- `NonSystemActionLogged` - For audit purposes

**Error Scenarios:**
- Description missing > Validation error
- Action type not recognized > Validation error

---

## Configure PPR Worklist

**What:** PPR Admin creates or modifies worklist routing rules  
**When:** New processing queue needed or routing logic changes  
**Who:** User with `profpractice.worklist.configure` permission (PPR Admin). Permission: `profpractice.worklist.configure` (system-wide), with `profpractice.worklist.view` for the list read

```mermaid
---
title: Professional Practice Review - Configure PPR Worklist
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
    end
    
    Admin->>UI: Navigate to "Manage Worklists"
    
    UI->>PprApi: APP GET /worklists
    PprApi-->>UI: List of worklists
    
    alt Creating New Worklist
        Admin->>UI: Click "Create Worklist"
        Admin->>UI: Enter worklist name (e.g., "Felony Review Queue")
        
        Admin->>UI: Define routing rules
        Note over Admin,UI: Configure conditions:<br/>- Disclosure type = Felony<br/>- Status = Under PPR Review<br/>- Date range filters
        
        Admin->>UI: Set priority rules (sort by conviction date desc)
        Admin->>UI: Configure visibility settings
        
        Admin->>UI: Submit worklist
        UI->>PprApi: APP POST /worklists
        
        PprApi->>PprApi: Create PPRWorklist aggregate
        Note over PprApi: State: Active
        
        PprApi-->>UI: Worklist created
        UI-->>Admin: Confirmation
        
    else Editing Existing Worklist
        Admin->>UI: Select worklist to edit
        UI->>PprApi: APP GET /worklists/{worklistId}
        PprApi-->>UI: Worklist configuration
        
        Admin->>UI: Modify routing rules or priority settings
        Admin->>UI: Submit changes
        
        UI->>PprApi: APP PUT /worklists/{worklistId}
        PprApi->>PprApi: Update PPRWorklist aggregate
        PprApi-->>UI: Worklist updated
        UI-->>Admin: Confirmation
    end
```

**Key Decisions:**
- **Routing Rules:** Define which disclosures automatically populate this worklist
- **Priority Settings:** Determine sort order within worklist
- **Visibility Settings:** Control which statuses/markers appear in this queue

**State Changes:**
- PPRWorklist: `None` > `Active` (new) or configuration updated (edits)

**Events Published:**
- None (configuration change only)

**Error Scenarios:**
- Conflicting routing rules across worklists > Validation error
- Admin lacks permission > Access denied

---

## View Disclosure History

**What:** Educator views their complete disclosure history with status details  
**When:** Educator wants to review past submissions and outcomes  
**Who:** Authenticated educator. Permission: `profpractice.disclosure.view` (self-only), with `profpractice.attachments.view` for supporting documents

```mermaid
---
title: Professional Practice Review - View Disclosure History
---
sequenceDiagram
    actor Educator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant DocsApi as Documents API
    end
    
    Educator->>UI: Navigate to "My Disclosure History"
    UI->>PprApi: APP GET /disclosures
    PprApi-->>UI: All disclosures (self-disclosed, PPR responses, Rap Back, NASDTEC)
    
    Note over UI: Display table:<br/>- Date Reported<br/>- Disclosure Type<br/>- Conviction Date<br/>- Status<br/>- Source (Self/PPR/RapBack/NASDTEC)
    
    Educator->>UI: Select disclosure to view details
    UI->>PprApi: APP GET /disclosures/{disclosureId}
    PprApi->>DocsApi: SVC GET /documents/search
    DocsApi-->>PprApi: Supporting documents
    PprApi-->>UI: Full disclosure details with supporting documents
    
    UI-->>Educator: Display disclosure details with documents
    
    Note over Educator,UI: Educator can:<br/>- View status history<br/>- Download supporting documents<br/>- See external remarks (if any)<br/>- Print disclosure record
```

**Key Decisions:**
- **Self-Only:** Educators can only view their own disclosures
- **Read-Only:** Cannot edit finalized disclosures; must contact PPR for corrections
- **External Remarks Visibility:** External remarks may be visible to educator; internal remarks are not

**State Changes:**
- None (read-only operation)

**Events Published:**
- None (or optional audit log entry)

**Error Scenarios:**
- No disclosures on file > Display message indicating no history
- Disclosure not found > Display error

---

## Export Disclosure Data

**What:** Educator or admin downloads disclosure records for offline review  
**When:** User needs records for personal files, legal proceedings, or compliance audits  
**Who:** Authenticated educator (self-only) or admin with appropriate permissions. Permission: `profpractice.disclosure.view` (self-only for educators; system-wide or worklist-specific for admins)

```mermaid
---
title: Professional Practice Review - Export Disclosure Data
---
sequenceDiagram
    actor User
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant ExportService as Export Service
    end
    
    User->>UI: Navigate to disclosure history
    UI->>PprApi: APP GET /disclosures
    PprApi-->>UI: Disclosures (filtered by permission scope)
    
    User->>UI: Click "Export to PDF/CSV"
    UI->>ExportService: APP POST /export/disclosures
    
    Note over ExportService: Generate export file with:<br/>- Disclosure details<br/>- Status history<br/>- Remarks (filtered by permission)<br/>- Document references
    
    ExportService-->>UI: Export file ready
    UI-->>User: Download export file
```

**Key Decisions:**
- **Format Options:** PDF for official records, CSV for data analysis
- **Permission Filtering:** Educators see only their own; admins see based on scope
- **Document References:** Export includes document IDs/names but not file content

**State Changes:**
- None (read-only operation)

**Events Published:**
- None (or optional audit log entry)

**Error Scenarios:**
- Export generation fails > Display error, allow retry
- No data to export > Display message

You're right on both counts! Let me add those two missing sequences:

---

## Review NASDTEC Disciplinary Record

**What:** PPR reviewer examines out-of-state disciplinary action and determines impact on Michigan credentials  
**When:** NASDTEC record appears in worklist after nightly batch matching  
**Who:** User with `profpractice.disclosure.view` permission assigned to worklist. Permission: `profpractice.nasdtec.view` (system-wide), with `profpractice.nasdtec.access` for the clearinghouse link, `profpractice.remarks.add` for remarks and `profpractice.disclosure.submit` for creating a disclosure from the record

```mermaid
---
title: Professional Practice Review - Review NASDTEC Disciplinary Record
---
sequenceDiagram
    actor Reviewer
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
        participant EventBus as Event Bus
    end
    
    Reviewer->>UI: View NASDTEC worklist
    UI->>PprApi: APP GET /worklists/{worklistId}/external-checks
    PprApi-->>UI: NASDTEC records requiring review
    
    Reviewer->>UI: Select NASDTEC record to review
    UI->>PprApi: APP GET /external-checks/{checkId}
    PprApi-->>UI: NASDTEC record details
    
    Note over UI: Display:<br/>- Educator name, DOB<br/>- Jurisdiction (state)<br/>- TransactionDate<br/>- ClearinghouseId<br/>- ClearinghouseUrl
    
    Reviewer->>UI: Click ClearinghouseUrl link
    Note over Reviewer: Opens NASDTEC clearinghouse site<br/>in new browser tab/window<br/>Views full disciplinary details externally<br/>Reviewer examines:<br/>- Type of disciplinary action<br/>- Reason for action<br/>- Severity and dates<br/>- Current status
    
    Reviewer->>UI: Return to system, add internal remarks
    UI->>PprApi: APP POST /external-checks/{checkId}/add-remark
    PprApi->>PprApi: Add reviewer notes to ExternalBackgroundCheck
    
    alt Create Michigan disclosure from NASDTEC finding
        Reviewer->>UI: Click "Create Disclosure from NASDTEC Record"
        UI->>PprApi: APP POST /disclosures/from-nasdtec
        
        PprApi->>PprApi: Create Disclosure aggregate
        Note over PprApi: SourceType: NASDTECTriggered<br/>State: Under PPR Review<br/>Description: Summary from NASDTEC
        
        PprApi->>PprApi: Link disclosure to ExternalBackgroundCheck
        
        PprApi--)EventBus: DisclosureSubmitted
        PprApi--)EventBus: PPRClearanceAssessmentChanged
        
        Note over Reviewer: Disclosure now appears in standard<br/>disclosure worklist for full review
        
    else No action required
        Reviewer->>UI: Update NASDTEC record status to "Reviewed - No Action"
        UI->>PprApi: APP PUT /external-checks/{checkId}/status
        
        PprApi->>PprApi: Update ExternalBackgroundCheck aggregate
        Note over PprApi: State: Reviewed - No Action
    end
    
    UI-->>Reviewer: Confirmation
```

**Key Decisions:**
- **External Review:** Full disciplinary details viewed on NASDTEC site, not in MiEdWorkforce
- **Disclosure Creation:** Reviewer decides whether NASDTEC finding warrants Michigan disclosure
- **Link Preservation:** Disclosure remains linked to original NASDTEC record for audit trail

**State Changes:**
- ExternalBackgroundCheck: `Received` > `Reviewed - No Action` or remains `Received` if disclosure created
- Disclosure: `None` > `Under PPR Review` (if created from NASDTEC finding)

**Events Published:**
- `DisclosureSubmitted` - If disclosure created from NASDTEC record
- `PPRClearanceAssessmentChanged` - If disclosure created

**Error Scenarios:**
- ClearinghouseUrl inaccessible > Display error, suggest manual follow-up
- Educator match no longer valid > Log issue, flag for admin review

---

## Generate PPR Compliance Report

**What:** Admin accesses reporting interface to view disclosure processing metrics and compliance data  
**When:** Ad-hoc reporting needs, compliance audits, or management review  
**Who:** User with `profpractice.reports.disclosure-metrics.view` or `profpractice.reports.compliance.view` permission. Permission: `profpractice.report.view` (scope-aware: system-wide, entity, district, ISD)

```mermaid
---
title: Professional Practice Review - Generate PPR Compliance Report
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant PprApi as PPR API
    end
    
    Admin->>UI: Navigate to "PPR Reports"
    UI-->>Admin: Display available report types
    Note over UI: Report options:<br/>- Disclosure Processing Metrics<br/>- Annual PPR Compliance Status<br/>- Review Timeline Analysis<br/>- Status Distribution<br/>- Worklist Performance
    
    Admin->>UI: Select report type and filters
    Note over Admin,UI: Filter options:<br/>- Date range<br/>- Disclosure type<br/>- Status<br/>- Worklist<br/>- Reviewer<br/>- District/ISD (scope-aware)
    
    Admin->>UI: Click "Generate Report"
    
    UI->>PprApi: APP GET /reports/data
    Note over PprApi: Query respects permission scope:<br/>System-wide vs Entity vs District
    
    PprApi->>PprApi: Aggregate disclosure data
    Note over PprApi: Calculate metrics:<br/>- Total disclosures by status<br/>- Average review time<br/>- Overdue PPR responses<br/>- Clearance assessment distribution<br/>- Worklist volume and throughput
    
    PprApi-->>UI: Report data (JSON)
    
    Note over UI: Browser renders the report in Power BI (embedded)<br/>and generates export files (PDF/Excel) there.<br/>No MiEdWorkforce API call is made for rendering or export
    UI-->>Admin: Display interactive report
    
    opt Admin wants to export
        Admin->>UI: Click "Export Report"
        UI-->>Admin: Download export
    end
```

**Key Decisions:**
- **External Reporting Tool:** Reports rendered via Power BI or similar business intelligence platform
- **PPR Domain Responsibility:** PPR Service provides data endpoints; reporting platform handles visualization
- **Scope-Aware:** Reports filter data based on user's permission scope (system-wide, district, ISD)
- **Real-Time Data:** Reports query current state, not cached/snapshot data

**Report Types Supported:**
1. **Disclosure Processing Metrics**
   - Total disclosures submitted (by period)
   - Status distribution (Under Review, Reviewed - Misdemeanor, etc.)
   - Average time in each status
   - Bottleneck identification

2. **Annual PPR Compliance Status**
   - Educators with compliant PPR responses (since June 30)
   - Educators overdue for PPR
   - Reminder escalation tracking
   - Compliance rate by district/ISD

3. **Review Timeline Analysis**
   - Time from submission to final review
   - Reviewer workload distribution
   - SLA compliance (if defined)

4. **Account Marker Summary**
   - Count of educators with each marker type
   - Marker duration analysis
   - Marker reason categories

5. **External Integration Status**
   - Rap Back notifications received and processed
   - NASDTEC records matched and reviewed
   - Integration success/failure rates

**State Changes:**
- None (read-only operation)

**Events Published:**
- None (or optional audit log entry for report access)

**Error Scenarios:**
- No data matching filters > Display message
- Reporting engine unavailable > Display error, suggest retry
- Permission denied for requested scope > Access denied
