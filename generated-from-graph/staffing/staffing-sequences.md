# Staffing - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Staffing domain.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

---

## Add New Employee

**What:** District user adds a new employee to the roster by searching for existing identity or creating new record, triggering Unique ID assignment via Mi-Key service.  
**When:** District hires new staff member and needs to report employment within 30 days (MCL 388.1619)  
**Who:** Staffing Authorized User (School District role)

```mermaid
---
title: Staffing - Add New Employee
---
---
title: Staffing - Add New Employee
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    participant IAM_API
    participant ProfPracticeAPI
    participant CommAPI

    DistrictUser->>StaffingUI: Navigate to "Add New Employee"
    StaffingUI->>StaffingAPI: GET /employee-roster/search (name, DOB)
    StaffingAPI-->>StaffingUI: Search results or empty

    alt Match found - use existing Unique ID
        DistrictUser->>StaffingUI: Select matching record
        StaffingUI->>StaffingAPI: POST /employee-roster/add-employee (pre-populated demographics + Unique ID)
    else No match - create new record
        DistrictUser->>StaffingUI: Enter demographic data (no Unique ID)
        StaffingUI->>StaffingAPI: POST /employee-roster/add-employee (demographics, no Unique ID)
    end

    StaffingAPI->>StaffingAPI: Validate demographics (field-level rules)

    alt Validation fails
        StaffingAPI-->>StaffingUI: 400 Bad Request (validation errors)
        StaffingUI-->>DistrictUser: Display errors
    else Validation passes
        StaffingAPI->>StaffingAPI: Create employee record (status: Pending)

        alt No Unique ID yet (new record)
            StaffingAPI->>IAM_API: POST /identity-resolution/requests (origin=BusinessUserStaffing, type=NewId, employeeRecordId, demographics)

            alt IAM/Mi-Key returns Match or No-Match
                IAM_API-->>StaffingAPI: 200/201 {uniqueId}
                StaffingAPI->>StaffingAPI: Associate Unique ID with employee record
            else IAM/Mi-Key returns Near Match (Requires Resolution)
                IAM_API-->>StaffingAPI: 202 Accepted {requestId, potentialMatches[]}
                StaffingAPI-->>StaffingUI: 202 Accepted (near match resolution required)
                StaffingUI-->>DistrictUser: Show near-match resolution screen (backed by IAM's identity-resolution API)
                Note over DistrictUser,IAM_API: See iam-sequences.md: "Business User - Request New ID (Near Match Escalation)"
            end
        end

        StaffingAPI->>IAM_API: POST /permissions/check (user, entity, staffing.employee-roster.add-employee)
        IAM_API-->>StaffingAPI: 200 OK {allowed: true}

        StaffingAPI->>ProfPracticeAPI: GET /educators/{uniqueId}/roster-eligibility
        ProfPracticeAPI-->>StaffingAPI: 200 OK {eligible: true, pprFlags: []}

        alt PPR flags restrict employment
            StaffingAPI-->>StaffingUI: 403 Forbidden (PPR restriction)
            StaffingUI-->>DistrictUser: Display error: "Employee cannot be added due to PPR restrictions"
        else Eligible for employment
            StaffingAPI--)CommAPI: Publish EmployeeAddedToRoster event
            StaffingAPI-->>StaffingUI: 201 Created {employeeRecordId, uniqueId}
            StaffingUI-->>DistrictUser: Success - redirect to employment details entry
        end
    end
```

**Key Decisions:**
- System requires user to search before adding to prevent duplicate Unique ID associations within same district
- Staffing never calls Mi-Key directly — identity resolution (matching, Near Match handling) is entirely IAM's, triggered here via API call; see `iam-domain.md`/`iam-sequences.md`
- PPR clearance check is blocking; employee cannot be added if restricted by professional practice flags

**State Changes:**
- EmployeeRoster: N/A > Pending (awaiting employment details, and possibly awaiting Unique ID from IAM)

**Events Published:**
- `EmployeeAddedToRoster` - Triggers notification to citizen account if exists, PPR clearance re-validation

**Error Scenarios:**
- Demographic validation fails > 400 Bad Request with field-level errors
- PPR flags restrict employment > 403 Forbidden, block employee addition
- IAM/Mi-Key service timeout > 503 Service Unavailable, user retries later
- Unique ID already in entity roster > 409 Conflict, user selects existing record instead

---

## Update Employee Demographics

**What:** District user or citizen updates demographic information for employee, triggering Mi-Key synchronization and history tracking.  
**When:** Name change (marriage, legal name change), address update, or correction of demographic data  
**Who:** Staffing Authorized User (School District role) or Citizen User (self-only)

```mermaid
---
title: Staffing - Update Employee Demographics
---
---
title: Staffing - Update Employee Demographics
---
sequenceDiagram
    actor User
    participant StaffingUI
    participant StaffingAPI
    participant IAM_API
    participant CommAPI

    User->>StaffingUI: Navigate to employee record > "Update Demographics"
    StaffingUI->>StaffingAPI: GET /employee-roster/{employeeId}/demographics
    StaffingAPI-->>StaffingUI: 200 OK {demographics, uniqueId, updateHistory[]}
    StaffingUI-->>User: Display demographic form (read-only fields editable)

    User->>StaffingUI: Update demographic fields + submit
    StaffingUI->>StaffingAPI: PATCH /employee-roster/{employeeId}/demographics (changedFields)

    StaffingAPI->>StaffingAPI: Validate changed fields (field-level rules)

    alt Validation fails
        StaffingAPI-->>StaffingUI: 400 Bad Request (validation errors)
        StaffingUI-->>User: Display errors
    else Validation passes
        StaffingAPI->>IAM_API: POST /identity-resolution/requests (origin=BusinessUserStaffing, type=DemographicUpdate, employeeRecordId, changedFields)

        alt IAM/Mi-Key returns Match (record updated)
            IAM_API-->>StaffingAPI: 200 OK {updateTimestamp}
            StaffingAPI->>StaffingAPI: Store demographic update in history (modified by, date)
            StaffingAPI--)CommAPI: Publish DemographicDataUpdated event

            alt Updated by District User
                CommAPI--)User: Email citizen account (if exists) notifying of district update
            else Updated by Citizen User
                CommAPI--)StaffingAPI: Trigger alert to district users for entity
            end

            StaffingAPI-->>StaffingUI: 200 OK {updateTimestamp}
            StaffingUI-->>User: Success confirmation with history view

        else IAM/Mi-Key returns Near Match (Requires Resolution)
            IAM_API-->>StaffingAPI: 202 Accepted {requestId, candidateUniqueId, matchScore}
            StaffingAPI->>StaffingAPI: Set employee record status to "Requires Resolution"; submitted data held by IAM in temporary storage (not persisted to EmployeeRoster)
            StaffingAPI-->>StaffingUI: 202 Accepted (near match resolution required)
            StaffingUI-->>User: Show near-match resolution screen (submitted data vs. same-Unique-ID candidate only, backed by IAM's identity-resolution API)

            Note over User,IAM_API: Update-path Near Match differs from Add-New-Employee: only the candidate sharing the submitted Unique ID is shown (other Unique IDs are never surfaced here); user may not "Request New ID" on this path (FDD 15.3) — self-resolved by the user, never escalated to the Identity Administrator except via auto-cancel timeout

            alt User confirms this is the correct individual
                User->>StaffingUI: Review comparison table; answer "Replace master record with submitted data?" (Yes/No); agree to attestation
                StaffingUI->>IAM_API: POST /identity-resolution/requests/{requestId}/resolve (selectedUniqueId, replaceMasterRecord: true|false)
                IAM_API->>IAM_API: Resolve near match; if replaceMasterRecord, update Mi-Key Master Record
                IAM_API--)StaffingAPI: IdentityResolutionCompleted / PersonRecordUpdated event
                StaffingAPI->>StaffingAPI: Store demographic update in history; clear "Requires Resolution" status
                StaffingAPI--)CommAPI: Publish DemographicDataUpdated event (cross-notification per update source)
                StaffingAPI-->>StaffingUI: 200 OK {updateTimestamp}
                StaffingUI-->>User: Success confirmation

            else User cancels the near match
                User->>StaffingUI: Select "Cancel"
                StaffingUI->>IAM_API: POST /identity-resolution/requests/{requestId}/cancel
                IAM_API--)StaffingAPI: IdentityRequestCancelled event
                StaffingAPI->>StaffingAPI: Discard temporary data; clear "Requires Resolution" status; no update applied
                StaffingAPI-->>StaffingUI: 200 OK
                StaffingUI-->>User: Update cancelled - record unchanged
            end
        end
    end
```

**Key Decisions:**
- All demographic updates flow through IAM's identity resolution API (which in turn talks to Mi-Key) to maintain a single source of truth for identity matching — Staffing holds no Near Match state of its own beyond a "Requires Resolution" flag on its own record
- Update history stored locally for audit trail (modified by, modified date)
- Cross-notification pattern: citizen notified of district updates, district notified of citizen updates
- Near Match on the Update path is intentionally narrower than on Add New Employee: only the candidate sharing the submitted Unique ID is ever displayed, and the user cannot escalate to "Request New ID" — they may only confirm (optionally replacing the master record) or cancel (FDD 15.3, "If the request is generated from an Update Record the user will be able to review and confirm the update or cancel the request... not able to Create a New ID when updating")
- An unresolved Near Match auto-cancels after an admin-configurable timeframe (FDD 15.3 Gap Analysis names this "X days," not yet numerically defined — see `iam-domain.md` Open Question #11)

**State Changes:**
- EmployeeRoster.demographics: Updated values (or unchanged, if user selects "No" to replace or cancels)
- EmployeeRoster.updateHistory: New entry appended
- EmployeeRoster record status: Active > Requires Resolution > Active (on confirm or cancel)

**Events Published:**
- `DemographicDataUpdated` - Triggers cross-notifications based on update source

**Error Scenarios:**
- IAM/Mi-Key update fails due to conflict (concurrent update) > 409 Conflict, user refreshes and retries
- Citizen updates demographics but district does not have notification configured > No error, event published but no email sent
- Invalid SSN format > 400 Bad Request before IAM call
- Near Match left unresolved past the configured timeout > IAM auto-cancels, sends cancellation to Mi-Key, discards temporary data, publishes `IdentityRequestCancelled` (FDD 15.3)

---

## Assign Employee to Position

**What:** District user assigns an employee to a position, triggering credential validation and spot allocation. If credentials are missing, user submits justification or applies for temporary credential.  
**When:** Employee is hired or reassigned to different position/course  
**Who:** Staffing Authorized User (School District role)

```mermaid
---
title: Staffing - Assign Employee to Position
---
---
title: Staffing - Assign Employee to Position
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    participant CredentialingAPI
    participant CommAPI

    DistrictUser->>StaffingUI: Navigate to position > "Add Filled Spot"
    StaffingUI->>StaffingAPI: GET /positions/{positionId}/details
    StaffingAPI-->>StaffingUI: 200 OK {position, educationJobType, localJobCategory, approvedFTE}

    StaffingUI-->>DistrictUser: Step 1: Spot Details (SCED course?, grade levels, FTE allocation, start date)
    DistrictUser->>StaffingUI: Enter spot details > Next

    StaffingUI-->>DistrictUser: Step 2: Select Employee (search + toggle "Only appropriately certified")
    StaffingUI->>StaffingAPI: GET /employee-roster/search-active (entityCode, filters)
    StaffingAPI-->>StaffingUI: 200 OK {employees[], activeEmploymentStatus}

    DistrictUser->>StaffingUI: Select employee > Next
    StaffingUI->>StaffingAPI: POST /assignments/validate-placement (employeeId, positionId, scedCode?, gradeSpan?)

    StaffingAPI->>CredentialingAPI: GET /educators/{uniqueId}/credentials
    CredentialingAPI-->>StaffingAPI: 200 OK {credentials[], endorsements[]}

    alt Administrator/Instructional position requires credential
        StaffingAPI->>StaffingAPI: Validate credential exists for educationJobType

        alt Credential valid
            StaffingAPI-->>StaffingUI: 200 OK {meetsRequirements: true}
        else Credential missing/invalid
            StaffingAPI-->>StaffingUI: 200 OK {meetsRequirements: false, missingCredential: true}
        end
    end

    alt Teacher position with course details requires endorsement
        StaffingAPI->>StaffingAPI: Validate endorsement covers SCED code + grade span

        alt Endorsement valid
            StaffingAPI-->>StaffingUI: 200 OK {meetsRequirements: true}
        else Endorsement missing/invalid
            StaffingAPI-->>StaffingUI: 200 OK {meetsRequirements: false, missingEndorsement: true}
        end
    end

    alt PPR clearance required
        StaffingAPI->>StaffingAPI: Check for PPR flags (if disclosure prevents assignment)

        alt PPR flag blocks assignment
            StaffingAPI-->>StaffingUI: 403 Forbidden {pprRestriction: true}
            StaffingUI-->>DistrictUser: Blocking error - cannot assign due to PPR disclosure
        end
    end

    StaffingUI-->>DistrictUser: Step 3: Confirm and Add (display credentials, endorsements, validation results)

    alt Meets all requirements
        DistrictUser->>StaffingUI: Confirm assignment
        StaffingUI->>StaffingAPI: POST /assignments/create (employeeId, positionId, spotDetails)
        StaffingAPI->>StaffingAPI: Create assignment (status: Active)
        StaffingAPI->>StaffingAPI: Update position FTE health indicator
        StaffingAPI--)CommAPI: Publish EmployeeAssignedToPosition event
        StaffingAPI-->>StaffingUI: 201 Created {assignmentId}
        StaffingUI-->>DistrictUser: Success - assignment created, return to position view

    else Does not meet requirements
        StaffingUI-->>DistrictUser: Show options: "Apply for Temporary Credential" or "Submit Justification"

        alt Apply for Temporary Credential
            DistrictUser->>StaffingUI: Select "Apply for Temporary Credential"
            StaffingUI-->>DistrictUser: Redirect to Credentialing domain temporary credential workflow
            Note over DistrictUser: After temporary credential granted, return to assignment

        else Submit Credential Error Justification
            DistrictUser->>StaffingUI: Select "Submit Justification" + enter free-form text
            StaffingUI->>StaffingAPI: POST /assignments/create-with-justification (employeeId, positionId, spotDetails, justificationText)
            StaffingAPI->>StaffingAPI: Create assignment (status: Justified - credential error)
            StaffingAPI--)CommAPI: Publish CredentialErrorJustificationSubmitted event
            CommAPI--)AuditorRole: Alert SOM Auditor of placement issue
            StaffingAPI-->>StaffingUI: 201 Created {assignmentId, status: Justified}
            StaffingUI-->>DistrictUser: Warning - assignment created with justification, potential aid deduction
        end
    end
```

**Key Decisions:**
- Three-step assignment workflow (spot details > select employee > confirm) ensures complete data entry before validation
- Credential validation is non-blocking but forces justification or temporary credential path if missing
- PPR flags can block assignment entirely (blocking error) if disclosure indicates risk
- FTE health indicator recalculates after spot creation but does not block assignment

**State Changes:**
- Assignment: N/A > Active or Justified (if credential error)
- Position.filledFTE: Updated with spot FTE allocation
- Position.spotCount: Incremented

**Events Published:**
- `EmployeeAssignedToPosition` - Triggers notification to employee if citizen account exists
- `CredentialErrorJustificationSubmitted` - Alerts SOM Auditor via inclusion in the audit item list

**Error Scenarios:**
- PPR flag blocks assignment > 403 Forbidden, user cannot proceed
- Invalid SCED code or grade span > 400 Bad Request, user corrects and resubmits
- Employee already assigned to same position/course > 409 Conflict, user reviews existing assignment
- FTE allocation exceeds available FTE for position > Warning only, assignment allowed

---

## Certify Collection

**What:** District user runs quality review, resolves errors, provides justifications, and certifies collection after attestation, triggering creation of an audit work item for the district's ISD/RESA entity.  
**When:** End of collection period (e.g., Fall 2024 Employee Roster due by legislative deadline)  
**Who:** Staffing Authorized User (School District role) with certification permission

```mermaid
---
title: Staffing - Certify Collection
---
---
title: Staffing - Certify Collection
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    participant CommAPI

    DistrictUser->>StaffingUI: Navigate to Collection Certification page
    StaffingUI->>StaffingAPI: GET /collections/{collectionId}/certification-status
    StaffingAPI-->>StaffingUI: 200 OK {preTasksComplete, errorCount, warningCount, status}

    StaffingUI-->>DistrictUser: Display workflow steps (pre-tasks > quality review > justifications > attestation > certify)

    alt Pre-certification tasks incomplete
        StaffingUI-->>DistrictUser: Block quality review - complete required tasks first
        DistrictUser->>StaffingUI: Complete tasks (update records, resolve errors, run reports)
    end

    DistrictUser->>StaffingUI: Run Quality Review
    StaffingUI->>StaffingAPI: POST /collections/{collectionId}/quality-review
    StaffingAPI->>StaffingAPI: Execute validation rules (record-level, cross-record, historical, EEM alignment)
    StaffingAPI-->>StaffingUI: 200 OK {errors[], warnings[], justificationRequired[]}

    StaffingUI-->>DistrictUser: Display quality review results

    alt Errors exist
        StaffingUI-->>DistrictUser: Block certification - correct errors first
        DistrictUser->>StaffingUI: Navigate to error records > correct data > re-run quality review
        Note over DistrictUser: Loop until all errors resolved
    end

    alt Justification-required conditions exist
        StaffingUI-->>DistrictUser: Display justification-required items
        DistrictUser->>StaffingUI: Enter justification text for each item
        StaffingUI->>StaffingAPI: POST /collections/{collectionId}/justifications (justifications[])
        StaffingAPI->>StaffingAPI: Store justifications linked to collection
        StaffingAPI-->>StaffingUI: 200 OK
    end

    StaffingUI-->>DistrictUser: Display attestation text + e-signature field
    DistrictUser->>StaffingUI: Agree to attestation + e-sign
    StaffingUI->>StaffingAPI: POST /collections/{collectionId}/certify (attestationAgreed, eSignature)

    StaffingAPI->>StaffingAPI: Verify all pre-conditions met (no errors, all justifications, attestation)

    alt Preconditions not met
        StaffingAPI-->>StaffingUI: 400 Bad Request (specific reason)
        StaffingUI-->>DistrictUser: Display error - cannot certify yet
    else Preconditions met
        StaffingAPI->>StaffingAPI: Mark collection as Certified (timestamp, certifying user)
        StaffingAPI->>StaffingAPI: Lock all records in collection (read-only except via reopen)
        StaffingAPI->>StaffingAPI: Create AuditWorkItem for constituent ISD/RESA entity (status: New)
        StaffingAPI--)CommAPI: Publish CollectionCertified event
        CommAPI--)ISDAuditor: Email alert of new district submission ready for review

        StaffingAPI-->>StaffingUI: 200 OK {certificationTimestamp}
        StaffingUI-->>DistrictUser: Success - collection certified, data finalized
    end
```

**Key Decisions:**
- Quality review must be run before certification but can be run multiple times as errors are corrected
- Errors are blocking, warnings are informational only
- Justifications are required only for specific validation conditions (e.g., all employees have same separation reason)
- Attestation and e-signature legally bind the certifying user to data accuracy

**State Changes:**
- Collection: In Progress > Quality Review > Certified
- All EmployeeRecord and Position records: Active > Certified (read-only)
- AuditWorkItem: N/A > New (visible in the ISD Auditor's audit item list)

**Events Published:**
- `CollectionCertified` - Triggers audit work item creation for the constituent ISD/RESA entity, email notifications

**Error Scenarios:**
- Error records still exist when certification attempted > 400 Bad Request, user corrects errors
- Justifications missing for required items > 400 Bad Request, user provides justifications
- Legislative deadline passed without certification > User must request Collection Exception
- Network failure during certification > 500 Internal Server Error, user retries (idempotent)

---

## ISD Auditor Review District Submission

**What:** ISD Auditor reviews district-certified collection, validates appropriate placement, requests additional documentation if needed, and finalizes audit report.  
**When:** After district certifies Employee Roster and Assignment Details collection  
**Who:** ISD Auditor (for constituent districts within ISD/RESA)

```mermaid
---
title: Staffing - ISD Auditor Review District Submission
---
---
title: Staffing - ISD Auditor Review District Submission
---
sequenceDiagram
    actor ISDAuditor
    participant AuditUI
    participant StaffingAPI
    participant CommAPI

    Note over ISDAuditor,StaffingAPI: Triggered by CollectionCertified event creating an audit work item

    ISDAuditor->>AuditUI: View Audit Items
    AuditUI->>StaffingAPI: GET /audit/items (isdCode)
    StaffingAPI-->>AuditUI: 200 OK {auditItems[], statuses[], dueDates[], credentialErrorJustifications[]}

    AuditUI-->>ISDAuditor: Display audit item list (district, audit type, due date, status)

    ISDAuditor->>AuditUI: Select district > View Data Collection Report
    AuditUI->>StaffingAPI: GET /audit/{auditItemId}/district-report (districtCode)
    StaffingAPI-->>AuditUI: 200 OK {sections: {employeesNotAppropriated[], employeesNoAssignment[], changesSummary[], ...}}

    AuditUI-->>ISDAuditor: Display report sections (employees not appropriately placed, etc.)

    alt Issues identified requiring documentation
        ISDAuditor->>AuditUI: Request Additional Documentation (specify doc type)
        AuditUI->>StaffingAPI: POST /audit/{auditItemId}/request-documentation (docType, details)
        StaffingAPI->>StaffingAPI: Create document request (status: Awaiting District Response)
        StaffingAPI--)CommAPI: Publish DocumentRequestCreated event
        CommAPI--)DistrictUser: Email alert to Staffing Data role users for district
        StaffingAPI-->>AuditUI: 201 Created {documentRequestId}

        Note over DistrictUser: District uploads requested documentation via Documents domain

        StaffingAPI->>StaffingAPI: Update audit item status to Awaiting District Response
    end

    alt Placement issues require formal finding
        ISDAuditor->>AuditUI: Create Audit Finding
        AuditUI->>StaffingAPI: POST /audit/{auditItemId}/findings (districtCode, individualDetails, issue, justification, uploadedDocs[])
        StaffingAPI->>StaffingAPI: Store audit finding linked to audit item
        StaffingAPI-->>AuditUI: 201 Created {findingId}
    end

    ISDAuditor->>AuditUI: All required tasks complete > Finalize Audit Report
    AuditUI->>StaffingAPI: GET /audit/{auditItemId}/certification-status
    StaffingAPI-->>AuditUI: 200 OK {allTasksComplete, findingsCount}

    alt Tasks incomplete
        StaffingAPI-->>AuditUI: 400 Bad Request (incomplete tasks listed)
        AuditUI-->>ISDAuditor: Display error - complete all tasks before finalization
    else Tasks complete
        AuditUI-->>ISDAuditor: Display attestation + e-signature field
        ISDAuditor->>AuditUI: Agree to attestation + e-sign
        AuditUI->>StaffingAPI: POST /audit/{auditItemId}/finalize (attestationAgreed, eSignature)

        StaffingAPI->>StaffingAPI: Mark audit as Completed (timestamp, auditor user)
        StaffingAPI->>StaffingAPI: Lock audit report (read-only except during de-certification window)
        StaffingAPI--)CommAPI: Publish AuditReportFinalized event
        CommAPI--)SOMAuditor: Email alert of finalized ISD audit for SOM review

        StaffingAPI-->>AuditUI: 200 OK {finalizationTimestamp}
        AuditUI-->>ISDAuditor: Success - audit report finalized
    end
```

**Key Decisions:**
- ISD Auditor's audit item list auto-populated by district certification event
- Document requests are tracked as separate entities linked to audit item, with status updates
- Audit findings stored for SOM Auditor review (downstream workflow)
- Finalized audit reports are read-only but can be de-certified during allowable window for corrections

**State Changes:**
- AuditWorkItem: New > In Progress > Awaiting District Response (if doc requested) > Pending SOM Review > Completed
- AuditFinding: N/A > Created

**Events Published:**
- `DocumentRequestCreated` - Triggers email to district Staffing Data role users
- `AuditReportFinalized` - Triggers SOM Auditor notification, adds item to SOM Auditor's audit item list

**Error Scenarios:**
- District does not respond to document request within deadline > Audit status remains Awaiting District Response, escalation to SOM
- ISD Auditor attempts to finalize without completing all tasks > 400 Bad Request
- Network failure during finalization > 500 Internal Server Error, user retries (idempotent)

---

## Request Collection Exception

**What:** District user requests permission to submit/certify collection data past legislative deadline, providing justification for delay.  
**When:** Collection deadline has passed and district needs additional time to complete submission  
**Who:** Staffing Authorized User (School District role)

```mermaid
---
title: Staffing - Request Collection Exception
---
---
title: Staffing - Request Collection Exception
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    actor StaffingDataAdmin
    participant CommAPI

    Note over DistrictUser,StaffingAPI: Context: Collection deadline passed, certification blocked

    DistrictUser->>StaffingUI: Navigate to Collection > "Request Exception"
    StaffingUI->>StaffingAPI: GET /collections/{collectionId}/exception-eligibility
    StaffingAPI-->>StaffingUI: 200 OK {deadlinePassed: true, exceptionAvailable: true}

    StaffingUI-->>DistrictUser: Display exception request form (requested extension dates, justification)
    DistrictUser->>StaffingUI: Enter justification + requested dates > Submit
    StaffingUI->>StaffingAPI: POST /collection-exceptions/request (collectionId, entityCode, justification, requestedDates)

    StaffingAPI->>StaffingAPI: Validate request (deadline passed, dates future-dated, justification provided)

    alt Validation fails
        StaffingAPI-->>StaffingUI: 400 Bad Request (validation errors)
        StaffingUI-->>DistrictUser: Display errors
    else Validation passes
        StaffingAPI->>StaffingAPI: Create exception request (status: Pending)
        StaffingAPI--)CommAPI: Publish CollectionExceptionRequested event
        StaffingAPI-->>StaffingUI: 201 Created {exceptionRequestId}
        StaffingUI-->>DistrictUser: Success - request pending admin review

        Note over StaffingDataAdmin: Staffing Data Admin reviews pending exception requests (staffing.admin.process-exceptions)

        alt Admin approves request
            StaffingDataAdmin->>StaffingAPI: POST /admin/collection-exceptions/{requestId}/approve (approvedDates)
            StaffingAPI->>StaffingAPI: Mark exception as Approved
            StaffingAPI->>StaffingAPI: Update collection open/close dates
            StaffingAPI--)CommAPI: Publish CollectionExceptionApproved event
            CommAPI--)DistrictUser: Email notification of approval with new dates

        else Admin denies request
            StaffingDataAdmin->>StaffingAPI: POST /admin/collection-exceptions/{requestId}/deny (denialReason)
            StaffingAPI->>StaffingAPI: Mark exception as Denied
            StaffingAPI--)CommAPI: Publish CollectionExceptionDenied event
            CommAPI--)DistrictUser: Email notification of denial with reason
        end
    end
```

**Key Decisions:**
- Exception requests only available after deadline has passed (prevents preemptive extensions)
- Staffing Data Admin has discretion to approve/deny based on justification
- Approved exceptions update collection dates, allowing continued submission
- Denial reason required to help district understand expectations

**State Changes:**
- CollectionException: N/A > Pending > Approved/Denied
- Collection.openDate/closeDate: Updated if exception approved

**Events Published:**
- `CollectionExceptionRequested` - Pending Staffing Data Admin review
- `CollectionExceptionApproved` - Notifies district, reopens collection
- `CollectionExceptionDenied` - Notifies district with denial reason

**Error Scenarios:**
- Request submitted before deadline > 400 Bad Request, not eligible for exceptionyet
- Requested dates in past > 400 Bad Request, dates must be future-dated
- Empty justification > 400 Bad Request, justification required
- Admin approval fails to update collection dates > 500 Internal Server Error, admin retries

---

## Open New Collection

**What:** System automatically initializes a new school-year collection at the start of the academic year, carrying forward active Position Roster and Employee Roster records from the prior collection and excluding ended/cancelled ones. District users can view the prior year's roster read-only and see data-quality indicators flagged immediately after rollover.  
**When:** Start of a new school year, per the Staffing Data Admin's defined collection schedule  
**Who:** System-triggered (no user action); Staffing Authorized Users view the result

```mermaid
---
title: Staffing - Open New Collection
---
---
title: Staffing - Open New Collection
---
sequenceDiagram
    participant Scheduler
    participant StaffingAPI
    participant CommAPI
    actor DistrictUser
    participant StaffingUI

    Scheduler->>StaffingAPI: Trigger new school-year collection initialization (entityCode, schoolYear)
    StaffingAPI->>StaffingAPI: Create Collection (status: Open) for entity/schoolYear
    StaffingAPI->>StaffingAPI: Load prior collection's Position Roster and Employee Roster records

    loop For each prior-collection Position/EmployeeRecord
        alt Record has no end date (or, for positions, no Cancelled status + reason)
            StaffingAPI->>StaffingAPI: Carry record forward into new collection with existing metadata
        else Record has end date and separation/cancellation reason
            StaffingAPI->>StaffingAPI: Exclude record from new collection; remains queryable in historical view
        end
    end

    StaffingAPI->>StaffingAPI: Recalculate FTE health indicators and data-quality flags for carried-forward positions
    StaffingAPI->>StaffingAPI: Log rollover outcome (counts carried forward, counts excluded, errors)
    StaffingAPI--)CommAPI: Publish CollectionOpened event

    DistrictUser->>StaffingUI: Log in after rollover, navigate to Position/Employee Roster
    StaffingUI->>StaffingAPI: GET /collections?schoolYear={new}&entityCode={entity}
    StaffingAPI-->>StaffingUI: 200 OK {collection, carriedForwardRecords[], dataQualityIndicators[]}
    StaffingUI-->>DistrictUser: Display new collection as basis for the year, prior year available read-only
```

**Key Decisions:**
- Rollover requires no user action; it runs automatically when the new school year becomes active
- A record is carried forward only when it lacks both an end date and (for positions) a cancellation reason — matching the existing "Employment End Date and Separation Reason Co-requirement" and "Position Status Cancelled Requires Reason" invariants
- Excluded (ended/cancelled) records remain visible in historical, read-only views scoped to their original collection year
- FTE health indicators and data-quality flags are recalculated immediately after rollover so districts see outstanding issues before they begin new-year data entry

**State Changes:**
- Collection: N/A > Open (new school year)
- PositionRoster/EmployeeRoster records: Carried forward unchanged, or excluded and archived to historical view

**Events Published:**
- `CollectionOpened` - Signals downstream systems (dashboards, reporting) that a new collection year is available

**Error Scenarios:**
- Unexpected error during rollover for a specific record > Logged and flagged for Staffing Data Admin review; does not block rollover for other records
- District logs in before rollover completes > Prior year's roster displayed as read-only until rollover finishes

---

## Create New Position

**What:** District user creates a new position in the organizational chart, defining job type, approved FTE, and position status.  
**When:** District approves new staffing allocation for upcoming school year or mid-year need arises  
**Who:** Staffing Authorized User (School District role)

```mermaid
---
title: Staffing - Create New Position
---
---
title: Staffing - Create New Position
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    participant EEMAPI
    participant CommAPI

    DistrictUser->>StaffingUI: Navigate to Positions > "Add Position"
    StaffingUI-->>DistrictUser: Display position creation form (title, job type, category, function, building, approved FTE)

    DistrictUser->>StaffingUI: Enter position details > Save
    StaffingUI->>StaffingAPI: POST /positions/create (entityCode, positionDetails)

    StaffingAPI->>StaffingAPI: Validate position fields (approved FTE > 0, job type/category/function alignment)

    alt Validation fails
        StaffingAPI-->>StaffingUI: 400 Bad Request (validation errors)
        StaffingUI-->>DistrictUser: Display errors
    else Validation passes
        StaffingAPI->>EEMAPI: GET /organizations/{entityCode}/buildings
        EEMAPI-->>StaffingAPI: 200 OK {buildings[]}
        StaffingAPI->>StaffingAPI: Validate building code exists in entity

        alt Building invalid
            StaffingAPI-->>StaffingUI: 400 Bad Request (invalid building code)
            StaffingUI-->>DistrictUser: Display error
        else Building valid
            StaffingAPI->>StaffingAPI: Generate position identifier (system-generated)
            StaffingAPI->>StaffingAPI: Create position (status: Active, filled FTE: 0, vacant FTE: 0, frozen FTE: 0)
            StaffingAPI--)CommAPI: Publish PositionCreated event
            StaffingAPI-->>StaffingUI: 201 Created {positionId, positionIdentifier}
            StaffingUI-->>DistrictUser: Success - redirect to position assignments page
        end
    end
```

**Key Decisions:**
- Position identifier is system-generated to ensure uniqueness
- Default status is Active (no employee assignment yet)
- Building validation against EEM ensures positions linked to valid entity hierarchy
- FTE health indicator shows mismatch immediately (approved FTE > 0, filled FTE = 0)

**State Changes:**
- Position: N/A > Active

**Events Published:**
- `PositionCreated` - Triggers dashboard updates, notifications if configured

**Error Scenarios:**
- Approved FTE <= 0 > 400 Bad Request, must be positive
- Invalid job type/category/function combination > 400 Bad Request, user corrects
- Building code not in entity's EEM data > 400 Bad Request, user selects valid building
- Duplicate position title within entity > Warning only, creation allowed

---

## Update Position Status

**What:** District user changes position status (e.g., Active to Filled, Active to Frozen, Active to Cancelled), enforcing business rules for employee associations and required fields.  
**When:** Position is filled by employee, temporarily frozen due to budget, or permanently cancelled  
**Who:** Staffing Authorized User (School District role)

```mermaid
---
title: Staffing - Update Position Status
---
---
title: Staffing - Update Position Status
---
sequenceDiagram
    actor DistrictUser
    participant StaffingUI
    participant StaffingAPI
    participant CommAPI

    DistrictUser->>StaffingUI: Navigate to Position Details > "Update Status"
    StaffingUI->>StaffingAPI: GET /positions/{positionId}
    StaffingAPI-->>StaffingUI: 200 OK {position, currentStatus, currentAssignments[]}

    StaffingUI-->>DistrictUser: Display status update form (new status, status date, cancellation reason?)
    DistrictUser->>StaffingUI: Select new status + enter required fields > Save
    StaffingUI->>StaffingAPI: PATCH /positions/{positionId}/status (newStatus, statusDate, cancelReason?)

    StaffingAPI->>StaffingAPI: Validate status transition rules

    alt Status = Filled
        alt No employee assigned
            StaffingAPI-->>StaffingUI: 400 Bad Request (Filled status requires employee assignment)
            StaffingUI-->>DistrictUser: Display error - assign employee first
        else Employee assigned
            StaffingAPI->>StaffingAPI: Update status to Filled
        end

    else Status = Approved
        alt Expected start date not in future
            StaffingAPI-->>StaffingUI: 400 Bad Request (Approved status requires future start date)
            StaffingUI-->>DistrictUser: Display error
        else Expected start date valid
            alt Employee currently assigned
                StaffingAPI-->>StaffingUI: 400 Bad Request (Approved status cannot have active employee)
                StaffingUI-->>DistrictUser: Display error - remove employee assignment first
            else No employee assigned
                StaffingAPI->>StaffingAPI: Update status to Approved
            end
        end

    else Status = Active
        alt Employee assigned after effective date
            StaffingAPI->>StaffingAPI: Update status to Active, end employee assignment at effective date
        else No employee assigned
            StaffingAPI->>StaffingAPI: Update status to Active
        end

    else Status = Frozen
        alt Employee assigned
            StaffingAPI->>StaffingAPI: Update status to Frozen, end employee assignment at effective date
        else No employee assigned
            StaffingAPI->>StaffingAPI: Update status to Frozen
        end

    else Status = Cancelled
        alt Cancellation reason not provided
            StaffingAPI-->>StaffingUI: 400 Bad Request (Cancelled status requires reason)
            StaffingUI-->>DistrictUser: Display error
        else Cancellation reason provided
            alt Employee assigned after effective date
                StaffingAPI->>StaffingAPI: Update status to Cancelled, end employee assignment at effective date
            else No employee assigned
                StaffingAPI->>StaffingAPI: Update status to Cancelled
            end
        end
    end

    StaffingAPI->>StaffingAPI: Record status change in audit history
    StaffingAPI--)CommAPI: Publish PositionStatusChanged event
    StaffingAPI-->>StaffingUI: 200 OK {updatedStatus, statusDate}
    StaffingUI-->>DistrictUser: Success - status updated
```

**Key Decisions:**
- Status transition rules enforce data integrity (e.g., Filled requires employee, Frozen prohibits employee)
- Employee assignments automatically end when position becomes Frozen or Cancelled
- Cancellation reason required to document why position eliminated
- Status changes tracked in audit history for compliance reporting

**State Changes:**
- Position.status: OldStatus > NewStatus
- Assignment.endDate: Set to status effective date if employee assigned and status is Frozen/Cancelled

**Events Published:**
- `PositionStatusChanged` - Triggers notifications if employee assignment ended

**Error Scenarios:**
- Filled status without employee > 400 Bad Request, user assigns employee first
- Cancelled status without reason > 400 Bad Request, user provides reason
- Approved status with current employee > 400 Bad Request, user removes assignment first
- Status date in future > Warning only, status change allowed
