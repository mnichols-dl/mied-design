
---

## Data Storage Strategy

### IAM Database (Azure SQL)

**Database:** `iam-db`

**Key Tables:**
- `users` - User accounts and authentication metadata
- `authorizations` - Active and inactive authorizations
- `authorization_requests` - Pending/approved/denied requests
- `approval_actions` - Approval/denial records
- `roles` - Role definitions
- `permissions` - Permission catalog
- `user_groups` - User group definitions
- `removal_requests` - External removal requests
- `inactivity_policies` - Deactivation policy configuration
- `impersonation_sessions` - Impersonation audit log
- `audit_log` - Authorization change audit trail

---

### Communications Database (Azure SQL)

**Database:** `communications-db`

**Key Tables:**
- `email_templates` - Template metadata
- `template_versions` - Versioned template content
- `email_instances` - Metadata for sent emails (subject, recipient, status, sent_at) - NO email body/content
- `email_delivery_events` - SendGrid status updates (Delivered, Bounced, etc.)
- `alerts` - Dashboard alerts
- `comments` - Comments on process items
- `consolidation_buffer` - Temporary event buffering for consolidation

---

### Email Content Store (Azure Cosmos DB)

**Database:** `email-content-store`

**Purpose:** Hot storage for rendered email content (subject + body HTML/text) for resend capability and help desk support

**Key Collections:**
- `email_content` - Rendered email body and subject, TTL-based auto-deletion

**Retention:** Configurable per category (default 90 days), after which content purged but metadata remains in SQL

**Note:** After content retention period expires, email content is permanently deleted from Cosmos DB. Only metadata (who, when, status) remains in SQL for audit compliance. No long-term cold storage for email content.

---

### Credentialing Database (Azure SQL)

**Database:** `credentialing-db`

**Key Tables:**
- `credential_applications` - Application submissions and processing state
- `issued_credentials` - Active and historical credentials
- `issued_endorsements` - Endorsements attached to certificates
- `credential_definitions` - Credential type configurations with versioning
- `endorsement_definitions` - Endorsement type configurations with versioning
- `assessment_definitions` - Assessment type configurations with versioning
- `assessment_results` - Educator competency assessment scores
- `definition_versions` - Historical versions of all definitions

---

### Documents Metadata Database (Azure SQL)

**Database:** `documents-metadata-db`

**Key Tables:**
- `documents` - Document metadata (filename, blob path, status, uploader, timestamps)
- `document_categories` - Category definitions with retention policies
- `document_versions` - Version history for replaced documents
- `bulk_operations` - Bulk upload/delete tracking
- `override_requests` - Retention override workflows
- `document_access_log` - Audit trail of all document interactions

---

### Document Storage (Azure Blob Storage)

**Primary Containers:** `documents-{category-slug}` (e.g., `documents-credential-supporting-docs`)

**Blob Path Structure:** `{year}/{month}/{attachment-type}/{attachment-id}/{document-id}_{sanitized-filename}`

**Lifecycle Policies:** Azure-managed tier transitions (Hot -> Cool -> Archive based on age)

---

### Payments Database (Azure SQL)

**Database:** `payments-db`

**Key Tables:**
- `payment_transactions` - Core payment lifecycle tracking (status, amount, confirmation number, timestamps)
- `bulk_payment_transactions` - Parent record for multi-application payments
- `refund_requests` - Refund workflow tracking
- `reconciliation_batches` - Daily posting file processing tracking
- `reconciliation_discrepancies` - Unmatched posting file records requiring manual review

**Key Design Decisions:**
- **Amount Immutability:** Payment amount calculated once at transaction creation and cannot be modified
- **Hash Validation:** All CEPAS confirmations require SHA-1 hash validation before status update
- **Posting File as Source of Truth:** Reconciliation file is authoritative; auto-updates missing payments
- **Temporal Tables:** System-versioned tables enable "what was the status at timestamp X?" queries
- **Optimistic Concurrency:** Row versioning prevents lost updates during concurrent operations

---

### Professional Practice Review Database (Azure SQL)

**Database:** `profpractice-db`

**Key Tables:**
- `disclosures` - Criminal history and disciplinary action reports
- `professional_practice_responses` - Annual PPR compliance submissions
- `educator_ppr_status` - Account markers and compliance tracking
- `external_background_checks` - Rap Back/NASDTEC integration data (metadata only, no rap sheet content)
- `ppr_worklists` - Disclosure review queue configurations
- `nasdtec_records` - Local storage of interstate disciplinary records

**Key Design Decisions:**
- **Assessment Calculation:** PPRClearanceAssessment and RosterEligibilityAssessment are calculated on-demand, never stored
- **Account Markers:** Boolean flags on EducatorPPRStatus trigger different clearance logic paths
- **Annual PPR Compliance:** Computed property (not stored) based on LastPPRResponseDate vs June 30 threshold
- **Rap Sheet Display-Only:** Full rap sheets retrieved via SOAP API are displayed transiently, NEVER persisted
- **NASDTEC Local Storage:** Interstate records stored locally for search/reference with ClearinghouseUrl link to external site
- **Temporal Tables:** System-versioned tables track all disclosure status changes and marker modifications

---

### Professional Learning Database (Azure SQL)

**Database:** `proflearning-db`

**Purpose:** Manages professional learning programs, attendance tracking, SCECH credit awarding, and college course credit applications

**Key Tables:**
- `sponsors` - Professional learning sponsor metadata (references EEM organizations)
- `coordinator_assignments` - User-to-sponsor coordinator role mappings
- `programs` - Approved professional learning offerings
- `program_applications` - Application lifecycle tracking (new, modify, renew)
- `program_sessions` - Specific dated offerings of programs
- `program_categories` - Classification taxonomy (categories and subcategories)
- `attendance_rosters` - Session-specific attendee collections
- `attendees` - Educator enrollments in sessions
- `scech_awards` - Awarded credits by type (General, College, Career, Military)
- `scech_correction_requests` - Post-certification adjustment workflows
- `evaluation_templates` - Question set references for program feedback
- `evaluation_submissions` - Attendee evaluation responses
- `college_course_applications` - Educator applications of STARR courses for SCECH credit
- `starr_course_records` - Synced college course completion data
- `bookmarks` - User-saved programs and sponsors

**Key Design Decisions:**
- **SCECH Immutability:** Once attendance certified, SCECH awards locked; changes require correction request workflow
- **Conversion Rate:** 1 semester credit = 15 SCECH hours (configurable)
- **College Course Eligibility:** Computed based on last credential issuance date from Credentialing domain
- **Credential Type Filtering:** School Counselor-specific SCECH types validated via Credentialing API
- **Temporal Tables:** System-versioned tables track program changes, correction requests, and award adjustments
- **External Registration:** Program fees displayed but collected via sponsor's external systems (not in MiEdWorkforce)

---

### EPP Database (Azure SQL)

**Database:** `epp-db`

**Purpose:** Manages educator preparation provider program approvals, candidate enrollment tracking, and EPP application review workflows

**Key Tables:**
- `educator_preparation_providers` - EPP designation and metadata (links to EEM organizations via read-only reference)
- `epp_contacts` - EPP-specific coordinator contact information
- `approved_certificate_categories` - Certificate type/pathway approvals with MDE dates and close dates
- `approved_endorsements` - Specific endorsements EPP can recommend by certificate type
- `candidate_enrollments` - Candidate journey through EPP programs (enrollment lifecycle tracking)
- `enrollment_programs` - Program assignments for each candidate enrollment (program level, grade band)
- `enrollment_status_history` - Temporal tracking of candidate status changes
- `credential_application_reviews` - EPP review records for credential applications
- `recommended_endorsements` - Endorsements recommended by EPP for specific applications
- `application_remarks` - External and internal remarks on application reviews
- `approval_application_reviews` - EPP review records for alternative route approval applications
- `epp_worklists` - Configured worklist definitions for application filtering
- `bulk_upload_jobs` - Candidate tracking bulk upload job metadata

**Key Design Decisions:**
- **EEM Dependency:** Core organization data (name, address, FICE code) is read-only from Organization Reference Data (synced from EEM)
- **EPP-Specific Data:** Only EPP designation, program approvals, and coordinator contacts stored locally
- **Candidate Multi-EPP:** Candidates can have enrollments at multiple EPPs simultaneously
- **Program Approval Dates:** MDE Approval Date, Enrollment Close Date (Pro Prep), Recommend Close Date (MiEdWorkforce) control visibility
- **MTTC Validation:** Assessment results consumed from Credentialing domain, not stored in EPP database
- **Application Reviews:** EPP domain stores review metadata and recommendations; Credentialing owns application lifecycle
- **State Machine:** Status transitions validated per EPP type (Traditional vs Alternative route) - rules defined in business logic
- **Temporal Tables:** System-versioned tables track enrollment status changes, program modifications, and EPP configuration updates
- **Bulk Upload Staging:** CSV files uploaded to Documents staging container, processed by Synapse, results stored in EPP database

**CEDS Alignment:**
- Organization references follow CEDS Organization model
- Candidate data aligned with CEDS K12Student where applicable
- Program categories map to CEDS CourseSection and CredentialType standards

---

### Staffing Database (Azure SQL)

**Database:** `staffing-db`

**Purpose:** Manages employment and position lifecycle for educators and staff within Michigan educational organizations, including roster definition, assignment tracking, and audit certification.

**Key Tables:**
- `employee_roster` - Employee records with employment status, start/end dates, separation reasons
- `employment_history` - Temporal tracking of employment status changes
- `position_roster` - Organizational chart of approved positions with FTE allocations
- `positions` - Position definitions with job type, category, function, approved FTE
- `spots` - Assignment allocations within positions (SCED course, grade levels, FTE, dates)
- `assignments` - Employee-to-position linkages with assignment details
- `assignment_course_details` - SCED codes, subject areas, grade spans for teaching assignments
- `education_history` - Post-secondary degrees and institutions for highest education calculation
- `evaluation_outcomes` - Annual performance assessment results with appeal tracking
- `new_teacher_monitoring` - Mentorship assignments and SCECH tracking for first 3 years
- `collections` - Collection definitions with school year dates, certification periods, statuses
- `collection_certification` - Attestation records with e-signatures and timestamps
- `quality_review_results` - Validation errors, warnings, justification-required conditions
- `audit_work_items` - ISD and SOM auditor review tracking with findings and documentation
- `audit_findings` - District/individual-level placement issues with justifications
- `document_requests` - Requests for additional evidence during audit review
- `collection_exceptions` - Approved requests for deadline extensions with justifications

**Key Design Decisions:**
- **Mi-Key Integration:** All employee records linked to Unique ID via synchronous API calls during creation
- **Credential Validation:** Placement appropriateness calculated on-demand via Credentialing domain API
- **PPR Eligibility:** Employment addition blocked if PPR flags restrict roster placement
- **Temporal Tables:** System-versioned tables track all employment status changes, position modifications, and assignment updates
- **Immutable Certification:** Certified collections become read-only except via formal reopen process (Collection Exception)
- **FTE Health Indicators:** Position FTE allocations monitored but not blocking (warning only)
- **Audit Workflow:** District certification triggers automatic ISD audit worklist creation via event subscription

**CEDS Alignment:**
- Employment data follows CEDS K12StaffEmployment model
- Position definitions aligned with CEDS K12StaffAssignment
- Course details mapped to SCED classification system
- Education history structured per CEDS PsStudentAcademicRecord

---

### Organizations Database (Azure SQL)

**Database:** `organizations-db`

**Key Tables:**
- `organizations` — Organization master data (code, name, type, status, lead admin email/name, grade band, CEDS JSON-LD metadata)
- `organization_hierarchy` — Materialized parent-child relationships with pre-computed ancestor chains
- `organization_types` — Active organization type catalog
- `organization_sync_log` — Audit trail of all sync pipeline executions

**Key Design Decisions:**
- **Read-Only Replica:** No write endpoints. All data flows in from CEPI via Synapse pipeline only.
- **Materialized Hierarchy:** Ancestor chains are pre-computed during sync and stored as a JSON array on each hierarchy record, enabling single-row lookups for IAM's transitive permission evaluation.
- **Inactive Retention:** Organizations deactivated in EEM are marked Inactive but never deleted; historical cross-domain references (authorizations, assignments) must remain resolvable.
- **CEDS JSON-LD:** Full CEPI payload stored in `ceds_metadata` (NVARCHAR MAX JSON column) for completeness and future interoperability; structured columns extract the fields actively used by consumers.
- **Sync Logging:** Every pipeline execution is logged with record-level counts and failure reasons to support operational monitoring and staleness tracking.