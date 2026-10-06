# Professional Learning - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Professional Learning domain.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human or external system
- **participant** = Internal service/component

---

## Program Application Submission and Approval

**What:** Sponsor submits new/modified program for approval
**When:** Sponsor wants to offer new program or modify existing
**Who:** Coordinator or Assistant Coordinator

```mermaid
---
title: Professional Learning - Program Application Submission
---
sequenceDiagram
    actor Coordinator
    participant ProfLearningUI
    participant ProfLearningAPI
    participant IAM_API
    participant DocumentsAPI
    participant ServiceBus
    
    Coordinator->>ProfLearningUI: Complete program application form
    ProfLearningUI->>IAM_API: Check permission (proflearning.program.create)
    IAM_API-->>ProfLearningUI: Authorized for sponsor
    
    Coordinator->>DocumentsAPI: Upload program agenda
    DocumentsAPI-->>Coordinator: Document reference
    
    ProfLearningUI->>ProfLearningAPI: POST /program-applications
    Note over ProfLearningAPI: Program details, agenda ref, sponsor ID
    ProfLearningAPI->>ProfLearningAPI: Create ProgramApplication (PendingApproval)
    ProfLearningAPI--)ServiceBus: Publish ProgramApplicationSubmitted
    Note over ServiceBus: Notifies communications; application is now visible in the admin's pending-review list
    ProfLearningAPI-->>ProfLearningUI: Application ID
    ProfLearningUI-->>Coordinator: Confirmation message
```

```mermaid
---
title: Professional Learning - Admin Approves Program Application
---
sequenceDiagram
    actor Admin
    participant ProfLearningAdminUI
    participant ProfLearningAPI
    participant DocumentsAPI
    participant ServiceBus
    
    Note over ProfLearningAdminUI: Admin views the filtered, sortable program-applications list
    Admin->>ProfLearningAdminUI: View pending program applications
    ProfLearningAdminUI->>ProfLearningAPI: GET /program-applications?status=PendingApproval
    ProfLearningAPI-->>ProfLearningAdminUI: Pending applications
    
    Admin->>ProfLearningAdminUI: Select application, review details
    ProfLearningAdminUI->>ProfLearningAPI: GET /program-applications/{id}
    ProfLearningAPI-->>ProfLearningAdminUI: Application details
    ProfLearningAdminUI->>DocumentsAPI: Fetch program agenda
    DocumentsAPI-->>ProfLearningAdminUI: Agenda content
    
    alt Approve
        Admin->>ProfLearningAdminUI: Add comments, approve
        ProfLearningAdminUI->>ProfLearningAPI: POST /program-applications/{id}/approve
        ProfLearningAPI->>ProfLearningAPI: Update Program (Approved)
        ProfLearningAPI--)ServiceBus: Publish ProgramApproved
    else Reject
        Admin->>ProfLearningAdminUI: Add comments, reject
        ProfLearningAdminUI->>ProfLearningAPI: POST /program-applications/{id}/reject
        ProfLearningAPI->>ProfLearningAPI: Update Application (Rejected)
        ProfLearningAPI--)ServiceBus: Publish ProgramRejected
    else Request More Info
        Admin->>ProfLearningAdminUI: Add comments, request info
        ProfLearningAdminUI->>ProfLearningAPI: POST /program-applications/{id}/request-info
        ProfLearningAPI->>ProfLearningAPI: Update Application (RequiresInfo)
        ProfLearningAPI--)ServiceBus: Publish InfoRequested
    end
    
    ProfLearningAPI-->>ProfLearningAdminUI: Updated status
    
    ServiceBus--)CommunicationsWorker: Event received
    CommunicationsWorker->>CommunicationsAPI: Notify coordinator of decision
```

**Key Decisions:**
- Document reference stored in ProfLearning, blob in Documents domain
- Admin's pending-review list is the same `GET /program-applications` endpoint, filtered by status
- Admin UI accesses ProfLearning API endpoints directly for approve/reject/request-info

**State Changes:**
- ProgramApplication: Draft -> PendingApproval -> Approved/Rejected/RequiresInfo
- Program: null -> Approved -> Active (if approved)

**Events Published:**
- `ProgramApplicationSubmitted` - Application becomes visible in the admin's pending-review list
- `ProgramApproved` - Triggers coordinator notification
- `ProgramRejected` - Triggers coordinator notification
- `InfoRequested` - Coordinator notified to provide additional details

**Error Scenarios:**
- Coordinator lacks permission for sponsor -> HTTP 403
- Required agenda document missing -> Validation error
- Missing required comments on reject/request-info -> Validation error

---

## Add Attendees to Program

**What:** Coordinator enrolls educators in program session
**When:** Before or during program delivery
**Who:** Coordinator

```mermaid
---
title: Professional Learning - Add Attendees
---
sequenceDiagram
    actor Coordinator
    participant ProfLearningUI
    participant ProfLearningAPI
    participant IAM_API
    
    Coordinator->>ProfLearningUI: Navigate to program session
    ProfLearningUI->>IAM_API: Check permission (proflearning.attendance.add)
    IAM_API-->>ProfLearningUI: Authorized
    
    alt Online Entry
        Coordinator->>ProfLearningUI: Search for educator by name/ID
        ProfLearningUI->>ProfLearningAPI: GET /educators/search?query={name}
        ProfLearningAPI->>IAM_API: Validate educators exist
        IAM_API-->>ProfLearningAPI: Matching educators with Unique IDs
        ProfLearningAPI-->>ProfLearningUI: Search results
        
        Coordinator->>ProfLearningUI: Select educator, add to roster
        ProfLearningUI->>ProfLearningAPI: POST /sessions/{id}/attendees
        ProfLearningAPI->>IAM_API: GET /users/{uniqueId}
        IAM_API-->>ProfLearningAPI: User confirmed
        Note over ProfLearningAPI: Default awarded SCECHs to program max
        ProfLearningAPI->>ProfLearningAPI: Add Attendee to roster
        ProfLearningAPI-->>ProfLearningUI: Success
    else File Upload
        Coordinator->>ProfLearningUI: Upload attendee CSV
        ProfLearningUI->>ProfLearningAPI: POST /sessions/{id}/attendees/bulk-upload
        ProfLearningAPI->>ProfLearningAPI: Parse CSV, extract Unique IDs
        loop For each Unique ID
            ProfLearningAPI->>IAM_API: GET /users/{uniqueId}
            IAM_API-->>ProfLearningAPI: User confirmed or not found
        end
        Note over ProfLearningAPI: Default awarded SCECHs to program max
        ProfLearningAPI->>ProfLearningAPI: Add valid Attendees to roster
        ProfLearningAPI-->>ProfLearningUI: Success with validation report
    end
    
    ProfLearningUI-->>Coordinator: Attendees added (with any errors)
```

**Key Decisions:**
- Global educator search (not scoped to coordinator's organization) per BRD 26.5.1
- Default awarded SCECHs to program maximum, per BR
- Unique ID validation via IAM API before adding to roster

**State Changes:**
- AttendanceRoster: Draft (or existing) with new Attendee entities

**Events Published:**
- None (attendance not certified yet)

**Error Scenarios:**
- Unique ID not found in IAM -> Display error, cannot add attendee
- Duplicate attendee -> Prevent addition with error message
- Bulk upload with invalid IDs -> Return validation report showing which IDs failed

---

## Adjust SCECH Awards and Certify Attendance

**What:** Coordinator adjusts awarded hours for partial attendance, then certifies roster
**When:** After program completion
**Who:** Coordinator

```mermaid
---
title: Professional Learning - Adjust SCECHs and Certify
---
sequenceDiagram
    actor Coordinator
    participant ProfLearningUI
    participant ProfLearningAPI
    participant CredentialingAPI
    participant ServiceBus
    
    Coordinator->>ProfLearningUI: View attendance roster for session
    ProfLearningUI->>ProfLearningAPI: GET /sessions/{id}/attendance
    ProfLearningAPI-->>ProfLearningUI: Roster with attendees
    
    opt Adjust for partial attendance
        Coordinator->>ProfLearningUI: Decrease awarded SCECHs for attendee
        ProfLearningUI->>ProfLearningAPI: PATCH /attendees/{id}/scech-award
        Note over ProfLearningAPI: Validate not exceeding program max
        ProfLearningAPI->>ProfLearningAPI: Update SCECHAward entity
        ProfLearningAPI-->>ProfLearningUI: Updated award
    end
    
    Coordinator->>ProfLearningUI: Review roster, agree to attestations
    Coordinator->>ProfLearningUI: Certify attendance
    ProfLearningUI->>ProfLearningAPI: POST /sessions/{id}/certify
    
    alt Evaluation Required
        ProfLearningAPI->>ProfLearningAPI: Check if all attendees submitted evaluation
        alt Missing evaluations
            ProfLearningAPI-->>ProfLearningUI: Error - pending evaluations
        end
    end
    
    opt School Counselor SCECH validation
        ProfLearningAPI->>CredentialingAPI: GET /credentials/{educatorId}/type
        CredentialingAPI-->>ProfLearningAPI: Credential type
        Note over ProfLearningAPI: Filter College/Career/Military SCECHs for non-counselors
    end
    
    ProfLearningAPI->>ProfLearningAPI: Mark roster as Certified
    Note over ProfLearningAPI: After certification, direct edits blocked
    ProfLearningAPI--)ServiceBus: Publish AttendanceCertified
    ProfLearningAPI-->>ProfLearningUI: Success
    
    ServiceBus--)CommunicationsWorker: AttendanceCertified event
    CommunicationsWorker->>CommunicationsAPI: Send SCECH award notifications to attendees
```

**Key Decisions:**
- Certification blocked if evaluations required but not submitted
- SCECH awards immutable after certification (changes require SCECHCorrectionRequest)
- School Counselor credential type validated before awarding specialized SCECH types

**State Changes:**
- AttendanceRoster: Draft -> Certified
- SCECHAward: Draft -> Finalized (immutable)

**Events Published:**
- `AttendanceCertified` - Triggers attendee notifications, updates Credentialing domain

**Error Scenarios:**
- Missing required evaluations -> Block certification
- Awarded SCECHs exceed program max -> Validation error

---

## Sponsor Requests SCECH Correction

**What:** Coordinator submits request to adjust SCECH awards after attendance has been certified
**When:** Error discovered post-certification (e.g., incorrect attendance, data entry mistake)
**Who:** Coordinator

```mermaid
---
title: Professional Learning - SCECH Correction Request
---
sequenceDiagram
    actor Coordinator
    participant ProfLearningUI
    participant ProfLearningAPI
    participant ServiceBus
    
    Coordinator->>ProfLearningUI: Navigate to certified attendance roster
    ProfLearningUI->>ProfLearningAPI: GET /sessions/{id}/attendance
    ProfLearningAPI-->>ProfLearningUI: Certified roster (locked)
    
    Coordinator->>ProfLearningUI: Select attendee, request correction
    ProfLearningUI->>ProfLearningUI: Display correction request form
    Note over ProfLearningUI: Shows current value, requests new value + justification
    
    Coordinator->>ProfLearningUI: Enter new SCECH value and justification
    ProfLearningUI->>ProfLearningAPI: POST /correction-requests
    Note over ProfLearningAPI: { attendeeId, sessionId, oldValue, newValue, justification }
    
    ProfLearningAPI->>ProfLearningAPI: Validate new value within program max
    ProfLearningAPI->>ProfLearningAPI: Create SCECHCorrectionRequest (Submitted)
    ProfLearningAPI--)ServiceBus: Publish SCECHCorrectionRequested
    Note over ServiceBus: Request becomes visible in the admin's correction-request review list
    ProfLearningAPI-->>ProfLearningUI: Correction request ID
    ProfLearningUI-->>Coordinator: Request submitted, awaiting admin review
```

**Key Decisions:**
- Correction requests required for any changes to certified attendance
- Justification is mandatory
- New SCECH value must not exceed program maximum

**State Changes:**
- SCECHCorrectionRequest: null -> Submitted

**Events Published:**
- `SCECHCorrectionRequested` - Request becomes visible in the admin's correction-request review list

**Error Scenarios:**
- New value exceeds program max -> Validation error
- Missing justification -> Validation error
- Duplicate pending request for same attendee -> Reject with error

---

## Admin Reviews Correction Request

**What:** Professional Learning Admin approves or denies SCECH correction request
**When:** Correction request appears in the admin's correction-request review list
**Who:** Professional Learning Admin

```mermaid
---
title: Professional Learning - Admin Reviews Correction
---
sequenceDiagram
    actor Admin
    participant ProfLearningAdminUI
    participant ProfLearningAPI
    participant ServiceBus
    
    Note over ProfLearningAdminUI: Admin views the filtered, sortable correction-requests list
    Admin->>ProfLearningAdminUI: View correction requests queue
    ProfLearningAdminUI->>ProfLearningAPI: GET /correction-requests?status=Submitted
    ProfLearningAPI-->>ProfLearningAdminUI: Pending correction requests
    
    Admin->>ProfLearningAdminUI: Select request, review details
    ProfLearningAdminUI->>ProfLearningAPI: GET /correction-requests/{id}
    ProfLearningAPI-->>ProfLearningAdminUI: Request details
    Note over ProfLearningAdminUI: Shows: attendee, program, session, old value, new value, justification
    
    alt Approve
        Admin->>ProfLearningAdminUI: Add notes, approve correction
        ProfLearningAdminUI->>ProfLearningAPI: POST /correction-requests/{id}/approve
        ProfLearningAPI->>ProfLearningAPI: Update CorrectionRequest (Approved)
        ProfLearningAPI->>ProfLearningAPI: Update ProgramAttendance with new SCECH value
        ProfLearningAPI--)ServiceBus: Publish SCECHCorrectionApproved
        Note over ServiceBus: Consumed by Credentialing to update SCECH balance
        ProfLearningAPI-->>ProfLearningAdminUI: Success
    else Deny
        Admin->>ProfLearningAdminUI: Add notes explaining denial, deny
        ProfLearningAdminUI->>ProfLearningAPI: POST /correction-requests/{id}/deny
        ProfLearningAPI->>ProfLearningAPI: Update CorrectionRequest (Denied)
        ProfLearningAPI--)ServiceBus: Publish SCECHCorrectionDenied
        ProfLearningAPI-->>ProfLearningAdminUI: Success
    end
    
    ServiceBus--)CommunicationsWorker: Event received
    CommunicationsWorker->>CommunicationsAPI: Notify coordinator of decision
```

**Key Decisions:**
- Admin can approve or deny, not modify the requested value
- Admin notes are required for denials
- Approval triggers update to both ProgramAttendance and Credentialing domain

**State Changes:**
- SCECHCorrectionRequest: Submitted -> Approved/Denied
- ProgramAttendance.SCECHAward: oldValue -> newValue (if approved)

**Events Published:**
- `SCECHCorrectionApproved` - Updates Credentialing domain, notifies coordinator
- `SCECHCorrectionDenied` - Notifies coordinator

**Error Scenarios:**
- Missing admin notes on denial -> Validation error

---

## Admin Configures Evaluation Questions

**What:** Admin creates or edits evaluation templates used for program feedback
**When:** New evaluation needed or existing template requires updates
**Who:** Professional Learning Admin

```mermaid
---
title: Professional Learning - Configure Evaluation Template
---
sequenceDiagram
    actor Admin
    participant AdminUI
    participant ProfLearningAPI
    participant QuestionSetAPI
    
    Admin->>AdminUI: Navigate to evaluation template management
    AdminUI->>ProfLearningAPI: GET /evaluation-templates
    ProfLearningAPI-->>AdminUI: Existing templates
    
    Admin->>AdminUI: Create new template or edit existing
    AdminUI->>AdminUI: Display template editor
    
    Admin->>AdminUI: Enter template name, description
    Admin->>AdminUI: Select applicable program categories
    
    Admin->>AdminUI: Add questions to template
    AdminUI->>QuestionSetAPI: GET /question-sets?domain=proflearning
    QuestionSetAPI-->>AdminUI: Available question sets
    Note over AdminUI: Question Set capability manages question content
    
    Admin->>AdminUI: Select question set IDs to include
    Admin->>AdminUI: Save template
    
    AdminUI->>ProfLearningAPI: POST /evaluation-templates
    Note over ProfLearningAPI: { name, description, categories[], questionSetRefs[] }
    ProfLearningAPI->>ProfLearningAPI: Create EvaluationTemplate (Active)
    ProfLearningAPI-->>AdminUI: Template ID
    AdminUI-->>Admin: Template saved successfully
```

**Key Decisions:**
- Evaluation templates reference Question Set IDs, not inline question content
- Question Set capability manages actual question versioning and structure
- Templates can be assigned to program categories (applies to all programs in category) or individual programs

**State Changes:**
- EvaluationTemplate: null -> Active

**Events Published:**
- None (configuration change, not a business event)

**Error Scenarios:**
- Invalid question set reference -> Validation error
- Duplicate template name -> Validation error

---

## Admin Manages Program Categories

**What:** Admin creates, edits, or deactivates program categories and subcategories
**When:** New classification needed or existing categories require updates
**Who:** Professional Learning Admin

```mermaid
---
title: Professional Learning - Manage Program Categories
---
sequenceDiagram
    actor Admin
    participant AdminUI
    participant ProfLearningAPI
    
    Admin->>AdminUI: Navigate to category management
    AdminUI->>ProfLearningAPI: GET /program-categories
    ProfLearningAPI-->>AdminUI: Categories and subcategories
    
    alt Create Category
        Admin->>AdminUI: Create new category
        AdminUI->>AdminUI: Display category form
        Admin->>AdminUI: Enter category name, description
        AdminUI->>ProfLearningAPI: POST /program-categories
        ProfLearningAPI->>ProfLearningAPI: Create ProgramCategory (Active)
        ProfLearningAPI-->>AdminUI: Category ID
    else Create Subcategory
        Admin->>AdminUI: Create subcategory under existing category
        AdminUI->>AdminUI: Display subcategory form
        Admin->>AdminUI: Enter subcategory name, select parent category
        AdminUI->>ProfLearningAPI: POST /program-categories/{parentId}/subcategories
        ProfLearningAPI->>ProfLearningAPI: Create Subcategory under parent
        ProfLearningAPI-->>AdminUI: Subcategory ID
    else Deactivate Category
        Admin->>AdminUI: Select category, deactivate
        AdminUI->>ProfLearningAPI: PATCH /program-categories/{id}/deactivate
        ProfLearningAPI->>ProfLearningAPI: Check for active programs using category
        alt Has active programs
            ProfLearningAPI-->>AdminUI: Error - cannot deactivate
            Note over AdminUI: Must reassign programs first
        else No active programs
            ProfLearningAPI->>ProfLearningAPI: Update category status (Inactive)
            ProfLearningAPI-->>AdminUI: Success
        end
    end
    
    AdminUI-->>Admin: Operation complete
```

**Key Decisions:**
- Categories cannot be deleted, only deactivated
- Deactivation blocked if active programs depend on the category
- Subcategories must belong to exactly one parent category

**State Changes:**
- ProgramCategory: null -> Active (create)
- ProgramCategory: Active -> Inactive (deactivate)

**Events Published:**
- None (configuration change, not a business event)

**Error Scenarios:**
- Duplicate category name -> Validation error
- Deactivate category with active programs -> Blocked with error
- Subcategory without parent -> Validation error

---

## Educator Applies College Course for SCECH Credit

**What:** Educator selects completed college courses to count toward SCECH requirements
**When:** Educator views available courses from STARR data
**Who:** Educator (Citizen User)

```mermaid
---
title: Professional Learning - Apply College Course
---
sequenceDiagram
    actor Educator
    participant ProfLearningUI
    participant ProfLearningAPI
    participant CredentialingAPI
    participant ServiceBus
    
    Educator->>ProfLearningUI: Navigate to college course credit page
    ProfLearningUI->>ProfLearningAPI: GET /college-courses/eligible
    ProfLearningAPI->>CredentialingAPI: GET /credentials/{educatorId}/last-issuance
    CredentialingAPI-->>ProfLearningAPI: Last issuance date
    ProfLearningAPI->>ProfLearningAPI: Filter STARR courses completed after issuance
    ProfLearningAPI->>ProfLearningAPI: Exclude already-applied courses
    ProfLearningAPI-->>ProfLearningUI: List of eligible courses
    
    Educator->>ProfLearningUI: Select course(s) to apply
    ProfLearningUI->>ProfLearningAPI: POST /college-courses/apply
    ProfLearningAPI->>ProfLearningAPI: Create CollegeCourseApplication
    ProfLearningAPI->>ProfLearningAPI: Convert college credits to SCECH hours (1 credit = 15 SCECHs)
    ProfLearningAPI->>ProfLearningAPI: Mark course as used
    ProfLearningAPI--)ServiceBus: Publish CollegeCourseApplied
    ProfLearningAPI-->>ProfLearningUI: SCECH credits awarded
    
    ServiceBus--)CommunicationsWorker: CollegeCourseApplied event
    CommunicationsWorker->>CommunicationsAPI: Send confirmation to educator
```

**Key Decisions:**
- Course eligibility determined by last credential issuance date from Credentialing domain
- STARR data synced via Synapse Pipeline (external dependency)
- Conversion rate: 1 semester credit = 15 SCECH hours

**State Changes:**
- CollegeCourseApplication: null -> Applied -> Credited
- STARRCourseRecord: Eligible -> Used

**Events Published:**
- `CollegeCourseApplied` - Updates Credentialing domain, triggers confirmation email

**Error Scenarios:**
- Course completed before last credential issuance -> Not displayed in eligible list
- Course already applied -> Not displayed in eligible list

---

## Public Catalog Search

**What:** Public searches for professional learning programs or sponsors
**When:** Anyone wants to discover learning opportunities
**Who:** Public (unauthenticated) or Educator (authenticated for bookmarking)

```mermaid
---
title: Professional Learning - Public Catalog Search
---
sequenceDiagram
    actor Public
    participant CatalogUI
    participant ProfLearningAPI
    
    Public->>CatalogUI: Navigate to Professional Learning Resources
    CatalogUI->>CatalogUI: Display basic search field
    
    Public->>CatalogUI: Enter search criteria (basic or advanced)
    CatalogUI->>ProfLearningAPI: GET /catalog/search?query={criteria}
    Note over ProfLearningAPI: Filter by Active status only
    ProfLearningAPI-->>CatalogUI: Programs and sponsors matching criteria
    
    CatalogUI-->>Public: Display search results grid
    
    alt View Program Details
        Public->>CatalogUI: Click Program Name hyperlink
        CatalogUI->>ProfLearningAPI: GET /programs/{id}/details
        ProfLearningAPI-->>CatalogUI: Program details (description, fees, contact, etc.)
        CatalogUI-->>Public: Display program detail page
    else View Sponsor Details
        Public->>CatalogUI: Click Sponsor Name hyperlink
        CatalogUI->>ProfLearningAPI: GET /sponsors/{id}/details
        ProfLearningAPI-->>CatalogUI: Sponsor details and active programs
        CatalogUI-->>Public: Display sponsor detail page
    end
    
    opt Bookmark (authenticated only)
        Public->>CatalogUI: Click bookmark icon
        CatalogUI->>ProfLearningAPI: POST /bookmarks
        ProfLearningAPI-->>CatalogUI: Bookmark saved
    end
```

**Key Decisions:**
- No authentication required for search, authentication required for bookmarking
- Only Active programs/sponsors returned in search results

**State Changes:**
- None (read-only operation)

**Events Published:**
- None

**Error Scenarios:**
- No results found -> Display "No results found" message
- Special characters in search -> Ignored per BR
- 