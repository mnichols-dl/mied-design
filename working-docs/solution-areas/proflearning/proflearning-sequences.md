# Professional Learning - Workflows & Sequences

This document contains sequence diagrams for all workflows in the Professional Learning domain.

**Conventions:**
- Solid arrows (`->>`) = Synchronous calls
- Dashed arrows (`-->>`) = Responses
- Dotted arrows (`--)`) = Async/fire-and-forget
- **actor** = Human only
- **participant** = Every non-human: UI, services, Event Bus, external systems
- Request arrows start with an API kind tag: `APP` (application API), `SVC` (service API), `EXT` (external API), `OUT` (outbound call to an external system)
- Every application API call is authorized by the owning service through the cached IAM permission check (Service API). It is not drawn unless noted.

---

## Program Application Submission and Approval

**What:** Sponsor submits new/modified program for approval
**When:** Sponsor wants to offer new program or modify existing
**Who:** Coordinator or Assistant Coordinator. Permission: proflearning.program.create (scoped to the coordinator's sponsor); the admin step uses proflearning.program.view and proflearning.program.approve (system-wide)

```mermaid
---
title: Professional Learning - Program Application Submission
---
sequenceDiagram
    actor Coordinator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant DocumentsApi as Documents API
        participant EventBus as Event Bus
    end

    Coordinator->>UI: Complete program application form

    Coordinator->>UI: Upload program agenda
    UI->>ProfLearningApi: APP Request agenda upload
    ProfLearningApi->>DocumentsApi: SVC POST /documents/upload/request
    ProfLearningApi-->>UI: Document reference
    UI-->>Coordinator: Agenda uploaded

    UI->>ProfLearningApi: APP POST /program-applications
    Note over ProfLearningApi: Program details, agenda ref, sponsor ID
    ProfLearningApi->>ProfLearningApi: Create ProgramApplication (PendingApproval)
    ProfLearningApi--)EventBus: ProgramApplicationSubmitted
    Note over EventBus: Notifies communications; application is now visible in the admin's pending-review list
    ProfLearningApi-->>UI: Application ID
    UI-->>Coordinator: Confirmation message
```

```mermaid
---
title: Professional Learning - Admin Approves Program Application
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant DocumentsApi as Documents API
        participant EventBus as Event Bus
    end

    Note over UI: Admin views the filtered, sortable program-applications list
    Admin->>UI: View pending program applications
    UI->>ProfLearningApi: APP GET /program-applications (status PendingApproval)
    ProfLearningApi-->>UI: Pending applications

    Admin->>UI: Select application, review details
    UI->>ProfLearningApi: APP GET /program-applications/{applicationId}
    ProfLearningApi->>DocumentsApi: SVC GET /documents/{documentId}/download
    DocumentsApi-->>ProfLearningApi: Agenda content
    ProfLearningApi-->>UI: Application details with agenda

    alt Approve
        Admin->>UI: Add comments, approve
        UI->>ProfLearningApi: APP POST /program-applications/{applicationId}/approve
        ProfLearningApi->>ProfLearningApi: Update Program (Approved)
        ProfLearningApi--)EventBus: ProgramApproved
        Note over EventBus: Consumed by Communications (notifies the coordinator of the decision)
    else Reject
        Admin->>UI: Add comments, reject
        UI->>ProfLearningApi: APP POST /program-applications/{applicationId}/reject
        ProfLearningApi->>ProfLearningApi: Update Application (Rejected)
        ProfLearningApi--)EventBus: ProgramRejected
        Note over EventBus: Consumed by Communications (notifies the coordinator of the decision)
    else Request More Info
        Admin->>UI: Add comments, request info
        UI->>ProfLearningApi: APP POST /program-applications/{applicationId}/request-info
        ProfLearningApi->>ProfLearningApi: Update Application (RequiresInfo)
        ProfLearningApi--)EventBus: InfoRequested
        Note over EventBus: Consumed by Communications (notifies the coordinator of the decision)
    end

    ProfLearningApi-->>UI: Updated status
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
**Who:** Coordinator. Permission: proflearning.attendance.add (scoped to the session's program)

```mermaid
---
title: Professional Learning - Add Attendees
---
sequenceDiagram
    actor Coordinator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant IamApi as IAM API
    end

    Coordinator->>UI: Navigate to program session

    alt Online Entry
        Coordinator->>UI: Search for educator by name/ID
        UI->>ProfLearningApi: APP GET /educators/search
        ProfLearningApi->>IamApi: SVC GET /users/search
        IamApi-->>ProfLearningApi: Matching educators with Unique IDs
        ProfLearningApi-->>UI: Search results

        Coordinator->>UI: Select educator, add to roster
        UI->>ProfLearningApi: APP POST /sessions/{sessionId}/attendees
        ProfLearningApi->>IamApi: SVC GET /users/by-unique-id/{uniqueId}
        IamApi-->>ProfLearningApi: User confirmed
        Note over ProfLearningApi: Default awarded SCECHs to program max
        ProfLearningApi->>ProfLearningApi: Add Attendee to roster
    else File Upload
        Coordinator->>UI: Upload attendee CSV
        UI->>ProfLearningApi: APP POST /sessions/{sessionId}/attendees/bulk-upload
        ProfLearningApi->>ProfLearningApi: Parse CSV, extract Unique IDs
        loop For each Unique ID
            ProfLearningApi->>IamApi: SVC GET /users/by-unique-id/{uniqueId}
            IamApi-->>ProfLearningApi: User confirmed or not found
        end
        Note over ProfLearningApi: Default awarded SCECHs to program max
        ProfLearningApi->>ProfLearningApi: Add valid Attendees to roster
        ProfLearningApi-->>UI: Success with validation report
    end

    UI-->>Coordinator: Attendees added (with any errors)
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
**Who:** Coordinator. Permission: proflearning.attendance.view, proflearning.attendance.adjust, proflearning.attendance.certify (scoped to the session's program)

```mermaid
---
title: Professional Learning - Adjust SCECHs and Certify
---
sequenceDiagram
    actor Coordinator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant CredApi as Credentialing API
        participant EventBus as Event Bus
    end

    Coordinator->>UI: View attendance roster for session
    UI->>ProfLearningApi: APP GET /sessions/{sessionId}/attendees
    ProfLearningApi-->>UI: Roster with attendees

    opt Adjust for partial attendance
        Coordinator->>UI: Decrease awarded SCECHs for attendee
        UI->>ProfLearningApi: APP PATCH /session-attendees/{attendeeId}/scech-award
        Note over ProfLearningApi: Validate not exceeding program max
        ProfLearningApi->>ProfLearningApi: Update SCECHAward entity
        ProfLearningApi-->>UI: Updated award
    end

    Coordinator->>UI: Review roster, agree to attestations
    Coordinator->>UI: Certify attendance
    UI->>ProfLearningApi: APP POST /sessions/{sessionId}/certify

    alt Evaluation Required
        ProfLearningApi->>ProfLearningApi: Check if all attendees submitted evaluation
        alt Missing evaluations
            ProfLearningApi-->>UI: 400 Pending evaluations
        end
    end

    opt School Counselor SCECH validation
        ProfLearningApi->>CredApi: SVC GET /credentials/{educatorId}/type
        CredApi-->>ProfLearningApi: Credential type
        Note over ProfLearningApi: Filter College/Career/Military SCECHs for non-counselors
    end

    ProfLearningApi->>ProfLearningApi: Mark roster as Certified
    Note over ProfLearningApi: After certification, direct edits blocked
    ProfLearningApi--)EventBus: AttendanceCertified
    Note over EventBus: Consumed by Communications (sends SCECH award notifications to attendees)
    ProfLearningApi-->>UI: Certification result
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
**Who:** Coordinator. Permission: proflearning.attendance.view, proflearning.correction.submit (scoped to the session's program)

```mermaid
---
title: Professional Learning - SCECH Correction Request
---
sequenceDiagram
    actor Coordinator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant EventBus as Event Bus
    end

    Coordinator->>UI: Navigate to certified attendance roster
    UI->>ProfLearningApi: APP GET /sessions/{sessionId}/attendees
    ProfLearningApi-->>UI: Certified roster (locked)

    Coordinator->>UI: Select attendee, request correction
    UI->>UI: Display correction request form
    Note over UI: Shows current value, requests new value + justification

    Coordinator->>UI: Enter new SCECH value and justification
    UI->>ProfLearningApi: APP POST /correction-requests
    Note over ProfLearningApi: { attendeeId, sessionId, oldValue, newValue, justification }

    ProfLearningApi->>ProfLearningApi: Validate new value within program max
    ProfLearningApi->>ProfLearningApi: Create SCECHCorrectionRequest (Submitted)
    ProfLearningApi--)EventBus: SCECHCorrectionRequested
    Note over EventBus: Request becomes visible in the admin's correction-request review list
    ProfLearningApi-->>UI: Correction request ID
    UI-->>Coordinator: Request submitted, awaiting admin review
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
**Who:** Professional Learning Admin. Permission: proflearning.correction.view, proflearning.correction.approve (system-wide)

```mermaid
---
title: Professional Learning - Admin Reviews Correction
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant EventBus as Event Bus
    end

    Note over UI: Admin views the filtered, sortable correction-requests list
    Admin->>UI: View correction requests queue
    UI->>ProfLearningApi: APP GET /correction-requests (status Submitted)
    ProfLearningApi-->>UI: Pending correction requests

    Admin->>UI: Select request, review details
    UI->>ProfLearningApi: APP GET /correction-requests/{correctionRequestId}
    ProfLearningApi-->>UI: Request details
    Note over UI: Shows: attendee, program, session, old value, new value, justification

    alt Approve
        Admin->>UI: Add notes, approve correction
        UI->>ProfLearningApi: APP POST /correction-requests/{correctionRequestId}/approve
        ProfLearningApi->>ProfLearningApi: Update CorrectionRequest (Approved)
        ProfLearningApi->>ProfLearningApi: Update ProgramAttendance with new SCECH value
        ProfLearningApi--)EventBus: SCECHCorrectionApproved
        Note over EventBus: Consumed by Credentialing to update SCECH balance, and by Communications (notifies the coordinator of the decision)
    else Deny
        Admin->>UI: Add notes explaining denial, deny
        UI->>ProfLearningApi: APP POST /correction-requests/{correctionRequestId}/deny
        ProfLearningApi->>ProfLearningApi: Update CorrectionRequest (Denied)
        ProfLearningApi--)EventBus: SCECHCorrectionDenied
        Note over EventBus: Consumed by Communications (notifies the coordinator of the decision)
    end

    ProfLearningApi-->>UI: Updated status
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
**Who:** Professional Learning Admin. Permission: proflearning.admin.evaluationtemplates (system-wide)

```mermaid
---
title: Professional Learning - Configure Evaluation Template
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant QuestionSetApi as Question Set API
    end

    Admin->>UI: Navigate to evaluation template management
    UI->>ProfLearningApi: APP GET /evaluation-templates
    ProfLearningApi-->>UI: Existing templates

    Admin->>UI: Create new template or edit existing
    UI->>UI: Display template editor

    Admin->>UI: Enter template name, description
    Admin->>UI: Select applicable program categories

    Admin->>UI: Add questions to template
    UI->>QuestionSetApi: APP GET /question-sets
    QuestionSetApi-->>UI: Available question sets
    Note over UI: Question Set capability manages question content

    Admin->>UI: Select question set IDs to include
    Admin->>UI: Save template

    UI->>ProfLearningApi: APP POST /evaluation-templates
    Note over ProfLearningApi: { name, description, categories[], questionSetRefs[] }
    ProfLearningApi->>ProfLearningApi: Create EvaluationTemplate (Active)
    ProfLearningApi-->>UI: Template ID
    UI-->>Admin: Template saved successfully
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
**Who:** Professional Learning Admin. Permission: proflearning.admin.categories (system-wide)

```mermaid
---
title: Professional Learning - Manage Program Categories
---
sequenceDiagram
    actor Admin
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
    end

    Admin->>UI: Navigate to category management
    UI->>ProfLearningApi: APP GET /program-categories
    ProfLearningApi-->>UI: Categories and subcategories

    alt Create Category
        Admin->>UI: Create new category
        UI->>UI: Display category form
        Admin->>UI: Enter category name, description
        UI->>ProfLearningApi: APP POST /program-categories
        ProfLearningApi->>ProfLearningApi: Create ProgramCategory (Active)
        ProfLearningApi-->>UI: Category ID
    else Create Subcategory
        Admin->>UI: Create subcategory under existing category
        UI->>UI: Display subcategory form
        Admin->>UI: Enter subcategory name, select parent category
        UI->>ProfLearningApi: APP POST /program-categories/{categoryId}/subcategories
        ProfLearningApi->>ProfLearningApi: Create Subcategory under parent
        ProfLearningApi-->>UI: Subcategory ID
    else Deactivate Category
        Admin->>UI: Select category, deactivate
        UI->>ProfLearningApi: APP POST /program-categories/{categoryId}/deactivate
        ProfLearningApi->>ProfLearningApi: Check for active programs using category
        alt Has active programs
            ProfLearningApi-->>UI: 400 Cannot deactivate
            Note over UI: Must reassign programs first
        else No active programs
            ProfLearningApi->>ProfLearningApi: Update category status (Inactive)
        end
    end

    UI-->>Admin: Operation complete
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
**Who:** Educator (Citizen User). Permission: proflearning.collegecourse.apply (self only)

```mermaid
---
title: Professional Learning - Apply College Course
---
sequenceDiagram
    actor Educator
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
        participant CredApi as Credentialing API
        participant EventBus as Event Bus
    end

    Educator->>UI: Navigate to college course credit page
    UI->>ProfLearningApi: APP GET /my/college-courses/eligible
    ProfLearningApi->>CredApi: SVC GET /credentials/{educatorId}/last-issuance
    CredApi-->>ProfLearningApi: Last issuance date
    ProfLearningApi->>ProfLearningApi: Filter STARR courses completed after issuance
    ProfLearningApi->>ProfLearningApi: Exclude already-applied courses
    ProfLearningApi-->>UI: List of eligible courses

    Educator->>UI: Select course(s) to apply
    UI->>ProfLearningApi: APP POST /my/college-courses
    ProfLearningApi->>ProfLearningApi: Create CollegeCourseApplication
    ProfLearningApi->>ProfLearningApi: Convert college credits to SCECH hours (1 credit = 15 SCECHs)
    ProfLearningApi->>ProfLearningApi: Mark course as used
    ProfLearningApi--)EventBus: CollegeCourseApplied
    Note over EventBus: Consumed by Communications (sends confirmation to the educator)
    ProfLearningApi-->>UI: SCECH credits awarded
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
**Who:** Public (unauthenticated) or Educator (authenticated for bookmarking). Permission: proflearning.catalog.search (unauthenticated); proflearning.catalog.bookmark (authenticated); proflearning.program.view and proflearning.sponsor.view for details

```mermaid
---
title: Professional Learning - Public Catalog Search
---
sequenceDiagram
    actor Public
    box Browser
        participant UI as UI
    end
    box MiEdWorkforce (AKS)
        participant ProfLearningApi as Professional Learning API
    end

    Public->>UI: Navigate to Professional Learning Resources
    UI->>UI: Display basic search field

    Public->>UI: Enter search criteria (basic or advanced)
    UI->>ProfLearningApi: EXT public GET /catalog/search
    Note over ProfLearningApi: Filter by Active status only
    ProfLearningApi-->>UI: Programs and sponsors matching criteria

    UI-->>Public: Display search results grid

    alt View Program Details
        Public->>UI: Click Program Name hyperlink
        UI->>ProfLearningApi: EXT public GET /programs/{programId}
        ProfLearningApi-->>UI: Program details (description, fees, contact, etc.)
        UI-->>Public: Display program detail page
    else View Sponsor Details
        Public->>UI: Click Sponsor Name hyperlink
        UI->>ProfLearningApi: EXT public GET /sponsors/{sponsorId}
        ProfLearningApi-->>UI: Sponsor details and active programs
        UI-->>Public: Display sponsor detail page
    end

    opt Bookmark (authenticated only)
        Public->>UI: Click bookmark icon
        UI->>ProfLearningApi: APP POST /my/bookmarks
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