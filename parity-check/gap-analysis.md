# Gap analysis

Computed from the graph: 51 aggregates, 60 entities, 193 value objects, 148 events, 105 business rules, 217 requirements, 305 permissions, 292 API operations, 179 sequences and batch jobs, 84 dependencies. Each finding is a prompt for a human decision, not necessarily an error; the heuristics are named where they matter.

## 1. Requirements

217 requirements; 91 have no satisfiedBy link (communications: 1, credentialing: 36, dashboards: 10, dataquality: 1, epp: 10, fdd32: 2, iam: 5, proflearning: 11, profpractices: 2, reporting: 4, staffing: 9).

- staffing: 24.4.1 Reject bulk-upload records lacking a valid Unique ID
- staffing: 22.5 Feature 22.5: Email/Comm (pointer, no own stories)
- staffing: 22.4 Feature 22.4: Data Input via API and File Upload (pointer, no own stories)
- staffing: 21.6 Email/Comm configuration scope pointer
- staffing: 10.BizSpec.EmploymentWorklist Configure the Employment Data Collection Worklist
- staffing: 10.BizSpec.DataQualityAdmin Configure Data Quality Review and Processing for staffing collections
- staffing: 10.6 Configure Staffing tiles and reports on role-based dashboards
- staffing: 10.16 Historical data connections/conversions — folded into other features' ACs
- staffing: 10.14 Queue management for API/file uploads — declined as an admin capability
- reporting: 2.BizSpec.4 Source report data from combined MDE/CEPI, historical, and integrated-system sources
- reporting: 2.BizSpec.3 Save, print, and download standardized and ad-hoc reports
- reporting: 2.BizSpec.2 Per-report admin controls: export tool selection, availability window, aggregation level
- reporting: 2.BizSpec.1 Report versioning on admin update
- profpractices: 30.BizSpec.SearchFilter Search and filter Rapback records
- profpractices: 30.BizSpec.MclMatching Match Rapback records against the MDE-defined MCL code list
- proflearning: 31.5.9 Hover-icon tooltips on Advanced Search fields, admin-editable text
- proflearning: 31.5.8 Reset button clears all search state
- proflearning: 31.5.6 'No results found' message clears on corrected search or Reset
- proflearning: 31.5.5 'No results found' validation message
- proflearning: 31.5.1 Professional Learning Opportunities Search hyperlink on landing page, no login
- proflearning: 28.2 Publishing the Professional Learning Catalog
- proflearning: 27-BizSpec-ProgramRegistrationDetailsStorage Store registration details to report on potential attendees (REJECTED / superseded)
- proflearning: 26.4 Professional Learning Catalog
- proflearning: 14.6 Document upload (scope pointer to 26.3/28.3; file-queue management excluded)
- proflearning: 14.5 Dashboard (scope deferral to Business Specification #3)
- proflearning: 14.4 Email/Comm (scope deferral to Business Specification #6)
- iam: 15.7 Historical Records — withdrawn, no longer necessary
- iam: 15.15 Citizen File Upload — withdrawn, no longer necessary
- iam: 15.14 ID Migration & Conversion — withdrawn, no longer necessary
- iam: 15.13 Identity-specific email/comms configuration — out of scope for iam, owned by Communications + Business Rule Management
- iam: 1.7 User Authorization reports — deferred to Feature 2.1
- fdd32: 32.3 Mark data inactive/'delete' per audit and retention schedules (soft-delete, historical reconstruction, admin change review)
- fdd32: 32.12 Vendor exit plan, environment compatibility, and pre-launch security/integration validation
- epp: 31.4.9 'No results found' validation message
- epp: 31.4.6 Program Name links to Available Providers (reverse lookup)
- epp: 31.4.5 Program results table: Name, Code, Type, Classification
- epp: 31.4.2 Providers vs. Programs search-mode toggle
- epp: 31.4.1 Educator Preparation Providers Search hyperlink on landing page, no login
- epp: 31.4.10 'No results found' message clears on corrected search or Clear
- epp: 13.7 Identity Management Integration for EPP (feature withdrawn)
- epp: 13.6 EPP Admin file-queue/upload-configuration management (declined as an admin feature)
- epp: 13.5 Dashboard configuration for EPP (pointer to Business Specification #3)
- epp: 13.4 Email/Comm configuration for EPP (pointer to Business Specification #6)
- dataquality: 19.5 MiEdWorkforce ADA Compliance (Feature 19.5)
- dashboards: 3.BizSpec.ReportsSummary Dashboard summary of upcoming reports with preview data
- dashboards: 3.BizSpec.PresetAdmin System Administrator authors preset role/function dashboards
- dashboards: 3.BizSpec.PersonnelSearchTile Personnel Search dashboard tile (SOM/Business users only)
- dashboards: 3.7 Dashboard - Staffing admin
- dashboards: 3.6 Dashboard - Credential admin
- dashboards: 3.5 Dashboard - Professional Learning Coordinator
- dashboards: 3.4 Dashboard - Educational Preparation Providers
- dashboards: 3.3 Dashboard - ISD/Auditor
- dashboards: 3.2 Dashboard - Educational Staff
- dashboards: 3.1 Dashboard - School District
- credentialing: 31.3 Legacy Reports (pre-2014 Special Education Approvals) — feature not carried into MiEdWorkforce
- credentialing: 31.2.9 'Learn about Credential Types' external link
- credentialing: 31.2.8 'Back to Educator Credentials' navigation button
- credentialing: 31.1.6 'No results found' validation message
- credentialing: 31.1.5 Credential Type filter: multiple selections or default All
- credentialing: 29 (Business Specifications, unnumbered — 'view PD hours by source and total') View Professional Development hours earned by source and total
- credentialing: 29 (Business Specifications, unnumbered — 'Complete SCECH Evaluations') View and complete SCECH evaluations for attended programs
- credentialing: 29 (Business Specifications, unnumbered — 'Add DPPD') Add District Provided Professional Development (DPPD) hours
- credentialing: 29 (Business Specifications, unnumbered — 'Add College Credits') Add college credits toward certificate requirements
- credentialing: 29.4.2 Generate downloadable cover letters
- credentialing: 29.2.9 Process/assign application by in-state vs. out-of-state
- credentialing: 29.2.7 Determine application status from Needs Responses flag
- credentialing: 29.1.8 Persistent certificate dashboard state
- credentialing: 29.1.10 Display system alerts related to credentialing
- credentialing: 17.FYB.2 MCL-waiver branch bypasses test-score/subject-area checks and forces Pending Evaluation
- credentialing: 17.2 Temporary Credentials User Feedback (feature-level pointer)
- credentialing: 17.1.5 Block duplicate endorsement/grade combination submissions
- credentialing: 16.3 Email/Communication (feature-level pointer)
- credentialing: 16.1.4 Search and view credential worklists
- credentialing: 11.9 Identity Management Integration (feature-level pointer, pending)
- credentialing: 11.8.3 View detailed MTTC test history within the application review screen
- credentialing: 11.7 Dashboard (feature-level pointer)
- credentialing: 11.6 Email/Communication (feature-level pointer)
- credentialing: 5.6 Reports (feature-level pointer)
- credentialing: 5.5.2 View a user's/role's assigned report access
- credentialing: 5.4.4 Capture audit details for manual batch job execution
- ... and 11 more

## 2. Permissions

### 2.1 Specs require a permission that the permissions docs do not define (7)

- `communications.credentials-templates.edit` (used by communications:updateTemplate, communications:createTemplateVersion)
- `communications.credentials-emails.view-history` (used by communications:searchEmailHistory)
- `communications.credentials-emails.resend` (used by communications:resendEmail)
- `communications.credentials-templates.view` (used by communications:listTemplates, communications:listTemplateVersions, communications:getTemplate)
- `communications.credentials-templates.delete` (used by communications:deleteTemplate)
- `communications.credentials-templates.create` (used by communications:createTemplate)
- `communications.credentials-templates.activate` (used by communications:activateTemplateVersion)

### 2.2 Permissions that no API operation requires (116: communications: 35, credentialing: 9, documents: 6, epp: 2, iam: 9, payments: 9, proflearning: 12, profpractices: 12, reporting: 6, staffing: 16)

Some permissions legitimately gate screens or data, not an operation, so read these as candidates.

- staffing: `staffing.new-teacher-monitoring.view` View new teacher monitoring data
- staffing: `staffing.new-teacher-monitoring.update` Update new teacher monitoring data
- staffing: `staffing.evaluation-outcome.view` View evaluation outcomes for employee
- staffing: `staffing.evaluation-outcome.submit` Submit evaluation outcome for employee
- staffing: `staffing.evaluation-outcome.approve-appeal` Approve or deny citizen appeal request
- staffing: `staffing.evaluation-outcome.appeal` Appeal evaluation outcome
- staffing: `staffing.employee-roster.bulk-upload` Upload bulk file to update employee data
- staffing: `staffing.education-history.view` View education history for employee
- staffing: `staffing.education-history.update` Add/update education history records
- staffing: `staffing.audit.decertify` De-certify audit report during allowable window
- staffing: `staffing.admin.view-file-queues` View and manage file upload queues
- staffing: `staffing.admin.reset-file-processing` Reset or delete files in processing queue
- staffing: `staffing.admin.manage-validations` Define and manage business rule validations
- staffing: `staffing.admin.manage-reports` Define and manage report availability and access
- staffing: `staffing.admin.manage-data-elements` Define and manage data element definitions
- staffing: `staffing.admin.manage-collections` Define and manage collection configurations
- reporting: `reporting.staffing-reports.manage` Define and manage report availability and access for Staffing
- reporting: `reporting.proflearning-reports.view` View and export all professional learning reports and analytics
- reporting: `reporting.payments-reports.view` View all payment reports (transactions, refunds, reconciliation, revenue, failed payments)
- reporting: `reporting.epp-reports.view` View EPP-specific reports (enrollment metrics, application statistics)
- reporting: `reporting.documents-reports.view` View document lifecycle and compliance reports
- reporting: `reporting.credentialing-reports.view` View application status and volume reports
- profpractices: `profpractice.worklist.assign` Add or remove reviewers from worklists
- profpractices: `profpractice.response.view` View PPR response history
- profpractices: `profpractice.questions.manage` Create, edit, or activate PPR question configurations
- profpractices: `profpractice.notifications.view` View PPR-related notifications for district users
- profpractices: `profpractice.nasdtec.access` Access external NASDTEC clearinghouse site for full details
- profpractices: `profpractice.disclosure.search` Search and filter disclosures across worklists
- profpractices: `profpractice.disclosure.revert` Revert finalized disclosure back to "Under PPR Review"
- profpractices: `profpractice.disclosure.delete` Remove disclosure from view but retain in audit (soft delete)
- profpractices: `profpractice.audit.view` View disclosure and marker audit logs
- profpractices: `profpractice.attachments.view` View supporting documents attached to disclosures
- profpractices: `profpractice.attachments.upload` Upload supporting documents to disclosure
- profpractices: `profpractice.attachments.delete` Remove document from view but retain in audit (soft delete)
- proflearning: `proflearning.sponsor.edit` Edit sponsor contact information and metadata
- proflearning: `proflearning.sponsor.delete` Delete sponsor applications or approved sponsors
- proflearning: `proflearning.sponsor.approve` Approve new sponsor applications
- proflearning: `proflearning.session.edit` Edit session dates and locations
- proflearning: `proflearning.session.create` Add program sessions (dates/locations)
- proflearning: `proflearning.report.view` View and export all professional learning reports and analytics
- proflearning: `proflearning.program.edit` Edit program details (triggers re-approval if approved)
- proflearning: `proflearning.evaluation.view` View submitted evaluations
- proflearning: `proflearning.evaluation.submit` Submit program evaluation
- proflearning: `proflearning.catalog.search` Search public professional learning catalog
- proflearning: `proflearning.attendance.override` Adjust SCECH hours after certification
- proflearning: `proflearning.admin.questions` Manage evaluation question sets
- payments: `payments.transaction.retry` Retry a failed payment transaction
- payments: `payments.report.view` View all payment reports (transactions, refunds, reconciliation, revenue, failed payments)
- payments: `payments.refund.view` View refund requests and history
- payments: `payments.reconciliation.view` View reconciliation batch results and daily posting file processing
- payments: `payments.reconciliation.run` Manually trigger reconciliation batch job
- payments: `payments.reconciliation.export` Export reconciliation data for financial audit
- payments: `payments.audit.view` View and export payment audit logs with advanced filtering
- payments: `payments.admin.force-status` Manually override a payment transaction status
- payments: `payments.admin.circuit-breaker-reset` Reset CEPAS circuit breaker after outage
- iam: `iam.user.view-own` View own user profile
- iam: `iam.user.impersonate` View system as another user (read-only)
- iam: `iam.user.edit` Edit user account details
- iam: `iam.removal-request.review` Review public authorization removal requests
- iam: `iam.inactivity.configure` Set auto-deactivation rules
- iam: `iam.identity-admin.manage-configuration` Manage identity-specific system configuration: validation rules, data element definitions, training materials, and processing queue monitoring
- iam: `iam.authorization.view` View authorization requests
- iam: `iam.audit.view` View authorization audit logs
- iam: `iam.approval-link.configure` Configure approval link expiration
- epp: `epp.reports.view` View EPP-specific reports
- epp: `epp.admin.system` Full administrative access to EPP domain
- documents: `documents.reports.view` View all document reports (storage usage, upload activity, retention compliance)
- documents: `documents.legal-hold.view` View documents with active legal holds
- documents: `documents.admin.view-quarantine` View documents flagged by malware scanner
- documents: `documents.admin.view-all` View and download any document regardless of attachment context
- documents: `documents.admin.override-scan` Mark a quarantined document as a false positive
- documents: `documents.admin.force-hard-delete` Immediately hard-delete a document outside the normal retention cycle
- credentialing: `credentialing.reports.view` View application status and volume reports
- credentialing: `credentialing.guidance.view` View credential issuance guidance documentation
- credentialing: `credentialing.guidance.manage` Upload and manage guidance documents
- credentialing: `credentialing.endorsement.view` View endorsements on credentials
- credentialing: `credentialing.credential.reinstate` Reinstate suspended credentials
- credentialing: `credentialing.credential.issue` Manually issue a credential record, correcting for an administrative error (e.g. an approved application that failed to generate its credential)
- credentialing: `credentialing.audit.view` View credential and application audit logs
- credentialing: `credentialing.assessment-result.view` View subject area assessment results for educators
- ... and 36 more

### 2.3 Application operations that require no permission (5)

- organizations: GET /organizations/search (searchOrganizations)
- organizations: GET /organization-types (listOrganizationTypes)
- organizations: GET /organizations/by-lead-admin (getOrganizationsByLeadAdmin)
- organizations: GET /organizations/{organizationCode}/hierarchy (getOrganizationHierarchy)
- organizations: GET /organizations/{organizationCode} (getOrganization)

## 3. API operations

### 3.1 Aggregates that no API schema projects from (4 of 51)

An aggregate with no API view is either internal-only, reached only by events, or missing an API.

- profpractices: ProfessionalPracticeResponse
- iam: InactivityPolicy
- documents: StagingFileAggregate
- credentialing: AssessmentResult

## 4. Sequences against the API and events

169 diagrams read. 394 of 396 arrows that name a verb and path carry an API-kind tag (APP, SVC, EXT, OUT).

### 4.1 API operations that no sequence calls (82 of 292: communications: 12, credentialing: 6, documents: 10, epp: 7, iam: 12, organizations: 3, proflearning: 11, profpractices: 5, reporting: 4, staffing: 12)

Matched on method and path (path parameters ignored). Plain CRUD and admin operations are often not in a sequence on purpose; the ones worth a look are state-changing operations with business meaning.

- staffing: PATCH /positions/{positionId} (updatePosition)
- staffing: PATCH /employee-roster/{employeeId}/employment-status (updateEmploymentStatus)
- staffing: PATCH /assignments/{assignmentId} (updateAssignment)
- staffing: POST /employee-roster/resolve-near-match (resolveNearMatch)
- staffing: POST /employee-roster/request-new-id (requestNewId)
- staffing: DELETE /assignments/{assignmentId} (endAssignment)
- staffing: DELETE /positions/{positionId} (cancelPosition)
- reporting: PUT /admin/reports/{reportDefinitionId} (adminUpdateReport)
- profpractices: POST /educators/{educatorId}/markers/rereview (setReReview)
- profpractices: POST /educators/{educatorId}/markers/enhanced-monitoring (setEnhancedMonitoring)
- profpractices: DELETE /educators/{educatorId}/markers/rereview (clearReReview)
- profpractices: DELETE /educators/{educatorId}/markers/mandatory-hold (clearMandatoryHold)
- profpractices: DELETE /educators/{educatorId}/markers/enhanced-monitoring (clearEnhancedMonitoring)
- proflearning: POST /programs/{programId}/withdraw (withdrawProgram)
- proflearning: PATCH /program-categories/{categoryId} (updateProgramCategory)
- proflearning: PUT /evaluation-templates/{templateId} (updateEvaluationTemplate)
- proflearning: POST /program-applications/{applicationId}/request-info (requestProgramApplicationInfo)
- proflearning: POST /program-applications/{applicationId}/reject (rejectProgramApplication)
- proflearning: DELETE /my/bookmarks (deleteBookmark)
- proflearning: POST /program-applications/{applicationId}/approve (approveProgramApplication)
- organizations: POST /admin/jobs/sync (triggerOrgSync)
- iam: POST /admin/jobs/generate-reports (triggerGenerateReports)
- iam: POST /authorizations/{authorizationId}/revoke (revokeAuthorization)
- iam: PATCH /identity-resolution/person-records/{userId} (adminUpdatePersonRecord)
- epp: PUT /epp-providers/{eppCode}/endorsements (updateEndorsement)
- epp: PUT /epp-providers/{eppCode}/certificate-categories (updateCertificateCategory)
- epp: PUT /candidate-enrollments/{enrollmentId}/programs (updateCandidateProgram)
- epp: DELETE /epp-providers/{eppCode} (deactivateEppProvider)
- documents: PATCH /documents/categories/{categorySlug} (updateDocumentCategory)
- documents: POST /documents/audit/logs/export (exportAuditLogs)
- documents: POST /documents/categories/{categorySlug}/deprecate (deprecateDocumentCategory)
- documents: POST /documents/categories (createDocumentCategory)
- communications: PUT /templates/{templateId} (updateTemplate)
- communications: PUT /comments/{commentId} (updateComment)
- communications: DELETE /templates/{templateId} (deleteTemplate)
- communications: DELETE /comments/{commentId} (deleteComment)
- communications: POST /templates (createTemplate)
- communications: POST /comments (createComment)
- communications: POST /alerts (createAlert)

### 4.2 Calls drawn in a sequence that match no operation in any spec (88)

- staffing: "Update Employee Demographics - Resolve Near Match" calls POST /identity-resolution/requests/{requestId}/cancel
- staffing: "ISD Auditor Review District Submission - Finalize ISD Audit" calls GET /audit/{auditItemId}/certification-status
- staffing: "Create New Position" calls GET /organizations/{entityCode}/buildings
- staffing: "Assign Employee to Position - Validate Placement" calls GET /employee-roster/search-active
- reporting: "Generate Embed Token and View Report - Happy Path" calls POST /GenerateToken
- profpractices: "Submit Self-Disclosure" calls POST /upload
- profpractices: "Submit Professional Practice Review Response" calls POST /upload
- profpractices: "Review NASDTEC Disciplinary Record" calls GET /worklists/{worklistId}/external-checks
- profpractices: "Review NASDTEC Disciplinary Record" calls PUT /external-checks/{checkId}/status
- profpractices: "Receive Rap Back Notification" calls POST /match-person
- profpractices: "Process NASDTEC Nightly Batch" calls GET /v1/clearinghouse/people
- profpractices: "Process NASDTEC Nightly Batch" calls POST /match-person
- profpractices: "Configure PPR Worklist" calls GET /worklists/{worklistId}
- proflearning: "Educator Applies College Course for SCECH Credit" calls GET /credentials/{educatorId}/last-issuance
- proflearning: "Admin Reviews Correction Request" calls GET /correction-requests/{correctionRequestId}
- proflearning: "Admin Configures Evaluation Questions" calls GET /question-sets
- proflearning: "Adjust SCECH Awards and Certify Attendance" calls GET /credentials/{educatorId}/type
- payments: "Refund Request and Approval" calls POST /api/v1/payment/cancel<br/>Headers
- payments: "Refund Request and Approval" calls POST /api/v1/payment/cancel<br/>Body
- payments: "Reconciliation Discrepancy Investigation" calls GET /reconciliation/discrepancies/{discrepancyId}
- payments: "Bulk Refund Processing" calls POST /api/v1/payment/cancel<br/>ConfirmationNumber
- payments: "Bulk Payment Processing" calls GET /applications/pending-payment
- organizations: "CEPI Nightly Sync" calls GET /organizations
- iam: "System Admin - Reviews External Authorization Removal Request" calls GET /removal-requests
- iam: "System Admin - Reviews External Authorization Removal Request" calls POST /removal-requests/{referenceId}/approve
- iam: "System Admin - Reviews External Authorization Removal Request" calls POST /removal-requests/{referenceId}/deny
- iam: "System Admin - Manual Authorization Grant" calls GET /organizations/search?query={query}
- iam: "System Admin - Manual Authorization Grant" calls GET /organizations/search?query={query}
- iam: "Identity Resolution - Split/Retire ID (Mi-Key-Direct)" calls POST /identity-resolution/callbacks
- iam: "Scope Approver - Rejects Authorization Request" calls GET /authorization-approvals/{token}
- iam: "Scope Approver - Approves Authorization Request" calls GET /authorization-approvals/{token}
- iam: "Integration Flows" calls GET /organizations/by-lead-admin?email={userEmail}
- iam: "Lead Administrator Bootstrap (First Login)" calls GET /organizations/by-lead-admin?email={userEmail}
- iam: "Identity Administrator - Resolve Identity Request" calls GET /identity-resolution/requests?status=RequiresResolution
- iam: "Identity Administrator - Resolve Link ID Request" calls GET /identity-resolution/requests?status=RequiresResolution
- iam: "Business User - Request Link ID" calls POST /employee-roster/{employeeId}/link-id-requests
- iam: "Business User - Request Authorization Update" calls GET /organizations/search?query={query}&status=Active
- iam: "Business User - Request Authorization Update" calls GET /organizations/search?query={query}&status=Active
- iam: "Business User - Initial Authorization Request" calls GET /organizations/search?query={query}&status=Active
- iam: "Business User - Initial Authorization Request" calls GET /organizations/search?query={query}&status=Active
- epp: "View Credential Application Detail for EPP Review" calls GET /applications/{applicationId}/mttc-results
- epp: "View Approval Application Detail for EPP Review" calls GET /applications/approvals/{applicationId}
- epp: "Update Candidate Enrollment Status" calls GET /candidate-enrollments/{enrollmentId}/available-transitions
- epp: "Search Approval Applications for Review" calls GET /applications/approvals
- epp: "Recommend Candidate for Credential" calls GET /applications/{applicationId}/mttc-results
- epp: "Load Recommendation Context" calls GET /applications/{applicationId}/requested-endorsements
- epp: "Load Recommendation Context" calls GET /applications/{applicationId}/mttc-results
- epp: "Load Recommendation Context" calls GET /epp-providers/{eppCode}/approved-endorsements
- documents: "Stage and Scan Import File" calls POST /{domain}/{staging-upload-route}
- documents: "Soft Delete Document" calls DELETE /{domain}/{document-route}
- documents: "Integration Pattern: Domain API Calling Documents" calls POST /{domain}/{upload-route}
- documents: "Request Upload and Upload File" calls POST /{domain}/{upload-route}
- documents: "Document Replacement and Versioning" calls POST /{domain}/{replace-route}
- documents: "Document Download with SAS Token" calls GET /{domain}/{download-route}
- documents: "Bulk Document Upload" calls POST /{domain}/{bulk-upload-route}
- credentialing: "View Definition History" calls GET /credential-definitions/{definitionId}/versions/{versionId}
- credentialing: "Submit Permit Application - Check Eligibility" calls GET /educators/{educatorId}/search
- credentialing: "Submit Permit Application - Check Eligibility" calls GET /permit-types
- credentialing: "Submit Permit Application - Check Eligibility" calls GET /permit-questions
- credentialing: "Submit New Certificate Application - Submit Application" calls POST /upload
- credentialing: "Submit New Certificate Application - Check Eligibility" calls GET /certificate-types
- credentialing: "Submit New Certificate Application - Check Eligibility" calls GET /eligibility-questions
- credentialing: "Submit New Certificate Application - Check Eligibility" calls POST /validate-eligibility
- credentialing: "Renew Credential - Check Eligibility" calls GET /credentials/renewable
- credentialing: "Renew Credential - Check Eligibility" calls GET /renewal-requirements
- credentialing: "Renew Credential - Check Eligibility" calls GET /scech-hours
- credentialing: "Nullify Endorsement" calls GET /endorsements/{endorsementId}/details
- credentialing: "Issue Temporary Permit (Exceptional) - Submit Application" calls POST /applications/permits/admin-issue/exceptional
- credentialing: "Issue Temporary Permit (Exceptional) - Check Eligibility" calls GET /permit-types
- credentialing: "Import Assessment Results" calls POST /upload
- credentialing: "Configure Endorsement Definition" calls GET /endorsement-definitions/{definitionId}
- credentialing: "Configure Credential Definition" calls GET /credential-categories
- credentialing: "Configure Credential Definition" calls GET /credential-definitions/{definitionId}
- credentialing: "Configure Assessment Definition" calls GET /assessment-definitions/{definitionId}
- credentialing: "Application Manual Review and Approval" calls POST /applications/{applicationId}/on-hold
- credentialing: "Add Endorsement to Certificate" calls GET /endorsements/eligible
- credentialing: "Add Endorsement to Certificate" calls GET /assessment-requirements
- communications: "Mass Email with Consolidation" calls GET /issues?ids=101
- communications: "Mass Email with Consolidation" calls POST /v3/mail/send
- communications: "Manual Email Send with Template Customization" calls GET /templates?functionalArea=Credentialing
- communications: "Manual Email Send with Template Customization" calls POST /v3/mail/send
- communications: "Event-Triggered Email Send - Send and Record Email" calls POST /v3/mail/send
- communications: "Email Resend with Recipient Override - Load Resend Context" calls GET /email-history/{instanceId}/resend-context
- communications: "Email Resend with Recipient Override - Resend Email" calls POST /v3/mail/send
- communications: "Email History Search and Details View" calls GET /email-history?functionalArea=Credentialing
- communications: "Email History Search and Details View" calls GET /email-history?recipientEmail=jane@example.com&status=Bounced
- communications: "Dashboard Alert Creation and Display - Display Alerts" calls GET /alerts?status=Active
- organizations: "CEPI Nightly Sync" calls GET /organizations

### 4.3 Events that no sequence publishes (46 of 148: communications: 13, credentialing: 6, documents: 3, iam: 3, payments: 4, proflearning: 7, profpractices: 4, staffing: 6)

- staffing: StaffingRecordSubmittedEvent
- staffing: NewTeacherMentorAssigned
- staffing: EvaluationOutcomeRecorded
- staffing: EmploymentStatusChanged
- staffing: EmployeeRosterAdditionAttemptedEvent
- staffing: AssignmentEndedEvent
- profpractices: PPRReminderRequired
- profpractices: PPRAccountMarkerChanged
- profpractices: DisclosureReviewStarted
- profpractices: DisclosureReviewCompleted
- proflearning: SCECHAwardAdjusted
- proflearning: ProgramRejected
- proflearning: ProgramModified
- proflearning: ProgramApproved
- proflearning: InfoRequestedEvent
- proflearning: EvaluationRequired
- proflearning: CoordinatorAssigned
- payments: RefundFailed
- payments: PaymentReconciled
- payments: PaymentInitiated
- payments: PaymentDueEvent
- iam: IdentityRecordSplit
- iam: IdentityRecordRetired
- iam: AuthorizationUpdated
- documents: RetentionPolicyUpdated
- documents: DocumentCategoryCreated
- documents: DocumentArchivedToBlob
- credentialing: EndorsementDefinitionDeactivated
- credentialing: CredentialRenewed
- credentialing: CredentialExpiredEvent
- credentialing: CredentialDefinitionDeprecated
- credentialing: AssessmentResultsValidated
- credentialing: AssessmentDefinitionRetired
- communications: EmailTemplateCreated
- communications: EmailTemplateActivated
- communications: EmailSent
- communications: EmailResent
- communications: EmailDropped
- communications: EmailDelivered
- communications: EmailDeferred
- communications: EmailContentPurgedEvent
- communications: EmailContentArchived
- communications: EmailBounced
- communications: DashboardAlertResolved
- communications: DashboardAlertCreated
- communications: CommentCreated

### 4.4 User-initiated sequences with no Permission on the Who line (7)

- payments: Individual Payment Flow (Payment Failed)
- iam: Integration Flows
- iam: Lead Administrator Bootstrap (First Login)
- documents: Staging File Cleanup Job
- documents: Hard Delete Expired Documents (Nightly Job)
- documents: Process Staged File in Synapse
- documents: Process Malware Scan Result

## 5. Domain model and events

### 5.1 Events with no emitting aggregate (7)

- staffing: StaffingRecordSubmittedEvent (Stub — placeholder for staffing's own real event, not yet converted here. See staffing-domain.md (no)
- staffing: EmployeeRosterAdditionAttemptedEvent (Stub — see staffing-domain.md (converted separately in this same batch, may not match exactly). prof)
- payments: PaymentDueEvent (Stub — distinct from any credentialing-side payment-completion event (a different purpose). See paym)
- iam: AuthorizationRemovalRequestedEvent (Emitted per iam-sequences.md but has no home in iam-domain.md's Core Aggregates section (only an ERD)
- external: UniqueIdAssignedEvent (Emitted by Mi-Key/the external identity-resolution capability, not by an Aggregate modeled in this f)
- external: BulkFileProcessedEvent (Published by Azure Synapse (ext:AzureSynapseSystem), not by Documents. Documents subscribes to updat)
- dataquality: DataQualityIssueDetectedEvent (Stub — dataquality is a stub-only registry context (no source docs anywhere); see dq:DataQualityCont)

### 5.2 Events with no consumers listed and no subscriber (20)

- staffing: StaffingRecordSubmittedEvent
- staffing: PositionStatusChangedEvent
- staffing: PositionCreatedEvent
- staffing: EmployeeRosterAdditionAttemptedEvent
- staffing: CollectionExceptionRequestedEvent
- staffing: AssignmentEndedEvent
- payments: PaymentDueEvent
- payments: DiscrepancyResolvedEvent
- payments: BulkRefundCompletedEvent
- iam: AuthorizationRemovalRequestedEvent
- external: UniqueIdAssignedEvent
- epp: BulkUploadCompletedEvent
- documents: OverrideRequestExecutedEvent
- documents: OverrideRequestCreatedEvent
- documents: BulkFileUploadedEvent
- dataquality: DataQualityIssueDetectedEvent
- credentialing: AssessmentResultImportedEvent
- communications: EmailTemplateVersionCreatedEvent
- communications: EmailSendFailedEvent
- communications: EmailContentPurgedEvent

### 5.3 Aggregates with states but no events (9)

- reporting: ReportDefinition (2 states)
- reporting: EmbedTokenRequest (2 states)
- profpractices: PPRWorklist (2 states)
- profpractices: PPRQuestionConfiguration (3 states)
- proflearning: ProgramCategory (2 states)
- proflearning: EvaluationTemplate (2 states)
- organizations: OrganizationTypeAggregate (2 states)
- iam: RoleDefinition (2 states)
- iam: InactivityPolicy (3 states)

### 5.4 Business rules tied to nothing (0 of 105)

Neither an aggregate they apply to nor anything that enforces them.

- None.

### 5.5 Rules enforced by nothing and mentioned in no sequence (4)

- reporting: Report Definitions Are Owned Outside the Application
- reporting: Not Every Dashboard Is a Power BI Report
- iam: Scope Context in Requests
- iam: External Removal Request Validation

## 6. Dependencies

### 6.1 Declared dependencies that name no operation or event (21 of 84)

- Documents to Credentialing (rest-api)
- Communications to SendGridSystem (rest-api)
- Iam to MiLoginSystem (rest-api)
- Documents to ProfessionalLearning (rest-api)
- ProfessionalPractices to MiKeySystem (rest-api)
- Reporting to PowerBISystem (rest-api)
- Iam to MiKeySystem (rest-api)
- Credentialing to Staffing (rest-api)
- Communications to CEPISystem (rest-api)
- Payments to CEPASSystem (rest-api)
- Payments to Iam (rest-api)
- Documents to Staffing (rest-api)
- ProfessionalLearning to Credentialing (rest-api)
- Epp to Credentialing (rest-api)
- Credentialing to ProfessionalLearning (rest-api)
- ProfessionalLearning to Credentialing (rest-api)
- Organizations to EEMSystem (rest-api)
- Communications to EEMSystem (rest-api)
- Documents to Epp (rest-api)
- Documents to AzureDefenderForStorageSystem (event-subscription)
- Documents to AzureBlobStorageSystem (rest-api)

### 6.2 Cross-area calls drawn in sequences with no declared dependency, as a node or in the dependency table (7 pairs)

- epp calls organizations (5 calls in sequences)
- communications calls iam (5 calls in sequences)
- epp calls profpractices (2 calls in sequences)
- communications calls credentialing (2 calls in sequences)
- staffing calls profpractices (1 call in sequences)
- proflearning calls staffing (1 call in sequences)
- payments calls credentialing (1 call in sequences)

## 7. Similar or overlapping things

Names are split into words (camel case, underscores), reduced to a rough stem, and compared as word sets across everything the design names: aggregates, entities, value objects, events, rules, permissions, API operations and schemas, glossary terms. A pair is listed when their word sets overlap strongly (Jaccard 0.6 or more, with at least two shared words, or identical sets in a different order). Descriptions are compared by shared meaningful words (cosine on word counts). The point is to surface the same idea under slightly different words, and the same name for different ideas.

### 7.1 Names that read as the same thing (40 pairs; top 40)

| A | B | Why |
| --- | --- | --- |
| staffing aggregate: Assignment | staffing entity: AssignmentDetail | same words |
| iam entity: AuthorizationRequest | iam entity: Authorization | same words |
| documents aggregate: BulkOperation | documents value object: BulkOperationSummary | same words |
| documents aggregate: Document | staffing entity: DocumentRequest | same words |
| profpractices value object: EducatorId | credentialing value object: EducatorId | same words |
| epp term: Endorsement | credentialing term: Endorsement | same words |
| organizations value object: GradeBand | credentialing value object: GradeBand | same words |
| profpractices term: Hard Delete | documents term: Hard Delete | same words |
| profpractices value object: InternalRemarks | epp value object: InternalRemark | same words |
| organizations term: Organization | iam term: Organization | same words |
| organizations term: Organization Hierarchy | iam term: Organization Hierarchy | same words |
| proflearning aggregate: ProgramCategory | proflearning value object: ProgramCategory | same words |
| profpractices value object: ResponseType | iam value object: RequestType | same words |
| profpractices value object: ResponseType | iam value object: RequestType | same words |
| iam entity: Role | iam value object: Role | same words |
| profpractices term: Soft Delete | documents term: Soft Delete | same words |
| communications event: EmailTemplateVersionCreatedEvent | communications event: EmailTemplateCreated | shares 3 of 4 words |
| proflearning value object: ApplicationReviewStatus | credentialing value object: ApplicationStatus | shares 2 of 3 words |
| epp event: ApprovalApplicationDenied | credentialing event: ApplicationDenied | shares 2 of 3 words |
| iam event: AuthorizationRequested | iam event: AuthorizationRemovalRequestedEvent | shares 2 of 3 words |
| documents aggregate: BulkOperation | documents entity: BulkOperationItem | shares 2 of 3 words |
| documents entity: BulkOperationItem | documents value object: BulkOperationSummary | shares 2 of 3 words |
| credentialing aggregate: CredentialApplication | epp event: CredentialApplicationRecommended | shares 2 of 3 words |
| credentialing aggregate: CredentialApplication | epp event: CredentialApplicationOnHold | shares 2 of 3 words |
| credentialing aggregate: CredentialApplication | epp event: CredentialApplicationDenied | shares 2 of 3 words |
| epp event: CredentialApplicationDenied | credentialing event: ApplicationDenied | shares 2 of 3 words |
| epp aggregate: CredentialApplicationReview | credentialing aggregate: CredentialApplication | shares 2 of 3 words |
| staffing event: DocumentRequestCreatedEvent | documents event: DocumentCategoryCreated | shares 2 of 3 words |
| profpractices value object: EffectiveDate | credentialing value object: EffectiveDateRange | shares 2 of 3 words |
| profpractices value object: EffectiveDate | credentialing value object: EffectiveDateRange | shares 2 of 3 words |
| organizations value object: GradeBand | credentialing value object: GradeBandRange | shares 2 of 3 words |
| credentialing value object: GradeBand | credentialing value object: GradeBandRange | shares 2 of 3 words |
| organizations value object: LeadAdministrator | iam rule: Lead Administrator Bootstrap | shares 2 of 3 words |
| payments event: PaymentCompleted | payments event: BulkPaymentCompleted | shares 2 of 3 words |
| profpractices term: Professional Practice Response | credentialing term: Professional Practice Status | shares 2 of 3 words |
| profpractices term: Professional Practice Response | credentialing term: Professional Practice Hold | shares 2 of 3 words |
| profpractices aggregate: ProfessionalPracticeResponse | epp value object: ProfessionalPracticeAnswers | shares 2 of 3 words |
| profpractices aggregate: ProfessionalPracticeResponse | credentialing value object: ProfessionalPracticeStatus | shares 2 of 3 words |
| profpractices aggregate: ProfessionalPracticeResponse | credentialing rule: Professional Practice Clearance | shares 2 of 3 words |
| payments event: RefundCompleted | payments event: BulkRefundCompletedEvent | shares 2 of 3 words |

### 7.2 Descriptions that overlap across areas (4 pairs; top 4)

| A | B | Similarity |
| --- | --- | ---: |
| profpractices entity: ReviewHistory | credentialing entity: IssuanceHistory | 0.63 |
| iam event: IdentityResolutionCompleted | external event: UniqueIdAssignedEvent | 0.60 |
| documents aggregate: StagingFileAggregate | external event: BulkFileProcessedEvent | 0.57 |
| profpractices rule: Reviewed - Felony Conditional Processing Rule | credentialing rule: Professional Practice Clearance | 0.56 |

### 7.3 Glossary terms defined in more than one area (5)

Each is either a shared concept defined twice (pick one owner) or one word meaning two things.

- "soft delete" in profpractices, documents
- "hard delete" in profpractices, documents
- "organization hierarchy" in organizations, iam
- "organization" in organizations, iam
- "endorsement" in epp, credentialing

### 7.4 Operations in different areas with nearly the same name (6)

- payments listPendingRefundApprovals (GET /refunds/pending-approval) and iam listPendingApprovals (GET /authorization-requests/pending-approvals)
- organizations searchOrganizations (GET /organizations/search) and iam searchOrganizations (GET /organizations/search)
- organizations getOrganizationHierarchy (GET /organizations/{organizationCode}/hierarchy) and iam getOrganizationHierarchy (GET /organizations/{organizationCode}/hierarchy)
- organizations getOrganization (GET /organizations/{organizationCode}) and iam getOrganization (GET /organizations/{organizationCode})
- epp initiateBulkUpload (POST /candidate-enrollments/bulk-upload/initiate) and documents initiateBulkUpload (POST /documents/bulk-upload/request)
- epp denyApplication (POST /application-reviews/{applicationId}/deny) and credentialing denyApplication (POST /applications/{applicationId}/deny)

### 7.5 Schemas with the same name in several specs (8)

Each is either a common shape that belongs in one shared definition (ErrorResponse, pagination) or a name that means different things in different areas.

- PaginationMetadata: communications, credentialing, documents, epp, iam, payments, proflearning, profpractices, reporting, staffing
- GradeBand: credentialing, epp, organizations, staffing
- ErrorResponse: payments, proflearning, reporting, staffing
- CertificationStatus: proflearning, staffing
- OrganizationType: iam, organizations
- OrganizationSummary: iam, organizations
- OrganizationDetail: iam, organizations
- Error: iam, organizations

