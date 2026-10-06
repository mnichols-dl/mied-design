# API specs in the graph: round trip, kinds of API, and events

## Round trip

All 11 `<area>-api.yml` specs were read into the graph (`reconcile-api.ts --apply`, spec text wins) and written back (`export-docs.ts --docs api`). `compare-api.ts` parses both and compares them key by key, ignoring key order, layout and comments. All 11 are identical in meaning.

What is not identical, by design:

- Comments in the source (the section dividers) are not reproduced.
- Layout and quoting follow the `yaml` package, not the original authors.
- One source defect is repaired in the comparison and the output: a path listed twice under `paths:` (invalid YAML, where a lenient parser silently keeps only the later one). The two entries are merged into one path item.
- Operation tags keep their authored order through `sd:tagsJson`; `sd:tags` holds the same values for querying. A list-form `x-access` (staffing) keeps its form through `sd:accessTypeJson`.
- An operation with no `x-access` in the spec has none in the graph. Earlier graph values for those eight organizations operations were removed (the spec wins), and the graph had also marked them internal-service, which the spec never said.

## The model

`sd:ApiSpec` holds the header, servers, security and the non-schema components as authored JSON. `sd:ApiOperation` holds the operation text and `sd:audience` (derived from `x-access`: internal-user is application, internal-service is service, external-public and external-client are external). `sd:ApiSchema` and `sd:ApiSchemaProperty` hold each component schema and its properties, with `sd:projectsFrom` linking a schema to the aggregate, entity or value object it is a view of. Everything the model does not give its own property stays as authored JSON on the owning node, so nothing in a spec is dropped.

The three kinds of API in `solution-architecture.md` are not recorded as a separate field in the specs. They are derived from `x-access`, and the legacy names are kept as written so the spec still round-trips. If the specs move to the new names, change the mapping in `audienceOf` (`scripts/api-doc.ts`).

## 1. Operations by kind of API

Kind comes from `x-access`: internal-user is an application API, internal-service a service API, external-public and external-client an external API. An operation with no `x-access` is unclassified.

| Area | Operations | Application only | Service only | External only | Application + service | External mixed with another kind | Unclassified |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| communications | 22 | 21 | 1 | 0 | 0 | 0 | 0 |
| credentialing | 34 | 31 | 0 | 2 | 1 | 0 | 0 |
| documents | 25 | 15 | 10 | 0 | 0 | 0 | 0 |
| epp | 44 | 38 | 4 | 2 | 0 | 0 | 0 |
| iam | 38 | 28 | 8 | 2 | 0 | 0 | 0 |
| organizations | 8 | 0 | 0 | 0 | 0 | 0 | 8 |
| payments | 10 | 9 | 0 | 1 | 0 | 0 | 0 |
| proflearning | 33 | 30 | 0 | 1 | 0 | 2 | 0 |
| profpractices | 30 | 26 | 3 | 1 | 0 | 0 | 0 |
| reporting | 9 | 9 | 0 | 0 | 0 | 0 | 0 |
| staffing | 39 | 25 | 1 | 0 | 0 | 13 | 0 |
| **Total** | 292 | 232 | 27 | 9 | 1 | 15 | 8 |

### Operations that mix an external API with another kind (15)

These cannot be split cleanly if the external API is to be its own service with no code shared with the application API. Each needs a decision: external only, application only, or two operations.

- proflearning: GET /sponsors/{sponsorId} (getSponsor) is internal-user, external-public
- proflearning: GET /programs/{programId} (getProgram) is internal-user, external-public
- staffing: POST /positions/{positionId}/status (updatePositionStatus) is internal-user, external-client
- staffing: PATCH /positions/{positionId} (updatePosition) is internal-user, external-client
- staffing: PATCH /employee-roster/{employeeId}/employment-status (updateEmploymentStatus) is internal-user, external-client
- staffing: PATCH /employee-roster/{employeeId}/demographics (updateEmployeeDemographics) is internal-user, external-client
- staffing: PATCH /assignments/{assignmentId} (updateAssignment) is internal-user, external-client
- staffing: POST /employee-roster/resolve-near-match (resolveNearMatch) is internal-user, external-client
- staffing: POST /employee-roster/{employeeId}/demographics/resolve-near-match (resolveDemographicsNearMatch) is internal-user, external-client
- staffing: POST /employee-roster/request-new-id (requestNewId) is internal-user, external-client
- staffing: DELETE /assignments/{assignmentId} (endAssignment) is internal-user, external-client
- staffing: POST /positions (createPosition) is internal-user, external-client
- staffing: POST /assignments/create (createAssignment) is internal-user, external-client
- staffing: POST /employee-roster/{employeeId}/demographics/cancel-near-match (cancelDemographicsNearMatch) is internal-user, external-client
- staffing: POST /employee-roster/add-employee (addEmployee) is internal-user, external-client

### Operations with no x-access (8)

- organizations: POST /admin/jobs/sync (triggerOrgSync)
- organizations: GET /organizations/search (searchOrganizations)
- organizations: GET /organization-types (listOrganizationTypes)
- organizations: GET /admin/sync-log (getSyncLog)
- organizations: GET /organizations/by-lead-admin (getOrganizationsByLeadAdmin)
- organizations: GET /organizations/{organizationCode}/hierarchy (getOrganizationHierarchy)
- organizations: GET /organizations/{organizationCode} (getOrganization)
- organizations: POST /admin/jobs/sync/complete (completeSyncJob)

## 2. Hostnames and security in spec headers

| Area | Servers | Top-level security |
| --- | --- | --- |
| communications | http://communications-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/communications/api/v1 (External via APIM (limited endpoints)) | [{"MiLogin":["communications:read"]}] |
| credentialing | http://credentialing-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/credentialing/api/v1 (External via APIM) | [{"MiLogin":["credentialing:read"]}] |
| documents | http://documents-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/documents/api/v1 (External via APIM) | none |
| epp | http://epp-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/epp/api/v1 (External via APIM) | [{"MiLogin":["epp:read"]}] |
| iam | http://iam-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/iam/api/v1 (External via APIM) | [{"MiLogin":["iam:read"]}] |
| organizations | http://organizations-api.platform.svc.cluster.local/api/v1 (Internal AKS cluster (platform namespace)) | [{"MiLogin":["organizations:read"]}] |
| payments | http://payments-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/payments/api/v1 (External via APIM) | [{"MiLogin":["payments:read"]}] |
| proflearning | http://proflearning-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/proflearning/api/v1 (External via APIM) | [{"MiLogin":["proflearning:read"]}] |
| profpractices | http://ppr-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/profpractice/api/v1 (External via APIM) | [{"MiLogin":["profpractice:read"]}] |
| reporting | http://reporting-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/reporting/api/v1 (External via APIM (not expected to be used; all consumers are internal)) | [{"MiLogin":["reporting:read"]}] |
| staffing | http://staffing-api.default.svc.cluster.local/api/v1 (Internal AKS cluster); https://api.miedworkforce.mi.gov/staffing/api/v1 (External via APIM) | [{"MiLogin":["staffing:read"]}] |

## 3. API schemas against domain events

For each event, the fields in its `payloadHighlights` are looked for in the schemas that project from the emitting aggregate (or from anything the aggregate links to). Names are compared ignoring case and punctuation, so `process_item_id` matches `processItemId`. Payload fields listed as missing are carried by the event but not exposed by any API schema for that aggregate: either the API should expose them, or the event carries data consumers cannot also get by calling the API.

148 events: 4 payloads fully covered by the aggregate's API schemas, 134 with fields missing from the schemas, 3 whose emitting aggregate has no API schema projecting from it, 7 with no emitting aggregate in the graph, 0 with no payload listed.

Of the missing fields: 73 are timestamps or dates (when the change happened), 22 name who did it, 30 give a reason or a change list, 111 are identifiers, 124 are something else. The first three are facts about the transition, which an API describing current state would not normally carry, so a gap there is expected. Identifiers and the others are the ones worth reading: an identifier the event carries and the schema lacks (applicantId in the event where the schema says educatorId, for example) is usually a naming difference to settle.

### Events with payload fields the API schemas do not carry

| Area | Event | Emitted by | Schemas checked | Fields missing |
| --- | --- | --- | --- | --- |
| communications | CommentCreated | Comment | CommentSummary | process_item_id |
| communications | DashboardAlertCreated | Alert | AlertSummary | user_id |
| communications | DashboardAlertResolved | Alert | AlertSummary | user_id, resolved_at |
| communications | EmailBounced | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | bounce_reason, substatus |
| communications | EmailContentArchived | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | archived_at |
| communications | EmailContentPurgedEvent | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | purged_at |
| communications | EmailDeferred | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | defer_reason |
| communications | EmailDelivered | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | delivered_at |
| communications | EmailDropped | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | drop_reason |
| communications | EmailResent | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | original_instance_id, new_recipient_email, resent_by |
| communications | EmailSendFailedEvent | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | template_id, version_id, event_id, missing, variable, names |
| communications | EmailSent | EmailInstance | EmailInstanceSummary, EmailInstanceDetail | template_id, recipients |
| communications | EmailTemplateActivated | EmailTemplate | TemplateVersionSummary, TemplateSummary, TemplateDetail | effective_date |
| credentialing | ApplicationApproved | CredentialApplication | RequiredDocument, RequiredAssessment, PaymentRequirement, ApplicationSummary, ApplicationStatus, ApplicationDetail | applicantId, approvedAt, approvedBy |
| credentialing | ApplicationDenied | CredentialApplication | RequiredDocument, RequiredAssessment, PaymentRequirement, ApplicationSummary, ApplicationStatus, ApplicationDetail | applicantId, deniedAt, deniedBy, reason |
| credentialing | ApplicationRequiresManualReview | CredentialApplication | RequiredDocument, RequiredAssessment, PaymentRequirement, ApplicationSummary, ApplicationStatus, ApplicationDetail | applicantId, reasons, enum, flaggedAt, context, missingDocuments, assessmentGaps, pprStatus |
| credentialing | AssessmentDefinitionCreated | AssessmentDefinition | AssessmentDefinitionSummary | effectiveFrom |
| credentialing | AssessmentDefinitionRetired | AssessmentDefinition | AssessmentDefinitionSummary | retiredAt, retiredBy, replacementAssessmentId |
| credentialing | AssessmentDefinitionUpdated | AssessmentDefinition | AssessmentDefinitionSummary | changes, effectiveFrom, updatedBy |
| credentialing | AssessmentResultsValidated | CredentialApplication | RequiredDocument, RequiredAssessment, PaymentRequirement, ApplicationSummary, ApplicationStatus, ApplicationDetail | educatorId, assessmentResults, assessmentId, score, passed, allRequirementsMet |
| credentialing | CredentialApplicationSubmitted | CredentialApplication | RequiredDocument, RequiredAssessment, PaymentRequirement, ApplicationSummary, ApplicationStatus, ApplicationDetail | applicantId |
| credentialing | CredentialDefinitionCreated | CredentialDefinition | CredentialDefinitionSummary | credentialType, effectiveFrom |
| credentialing | CredentialDefinitionDeprecated | CredentialDefinition | CredentialDefinitionSummary | credentialType, deprecatedAt, deprecatedBy |
| credentialing | CredentialDefinitionUpdated | CredentialDefinition | CredentialDefinitionSummary | credentialType, changes, effectiveFrom, updatedBy |
| credentialing | CredentialExpiredEvent | IssuedCredential | PublicEducatorSearchResult, PublicEducatorCredentialHistory, EducatorCredentialSummary, CredentialSummary, CredentialStatus, CredentialDetail | educatorId, expiredAt |
| credentialing | CredentialIssued | IssuedCredential | PublicEducatorSearchResult, PublicEducatorCredentialHistory, EducatorCredentialSummary, CredentialSummary, CredentialStatus, CredentialDetail | educatorId |
| credentialing | CredentialRenewed | IssuedCredential | PublicEducatorSearchResult, PublicEducatorCredentialHistory, EducatorCredentialSummary, CredentialSummary, CredentialStatus, CredentialDetail | educatorId, renewedAt, newExpiryDate |
| credentialing | CredentialRevoked | IssuedCredential | PublicEducatorSearchResult, PublicEducatorCredentialHistory, EducatorCredentialSummary, CredentialSummary, CredentialStatus, CredentialDetail | educatorId, revokedAt, reason |
| credentialing | CredentialSuspended | IssuedCredential | PublicEducatorSearchResult, PublicEducatorCredentialHistory, EducatorCredentialSummary, CredentialSummary, CredentialStatus, CredentialDetail | educatorId, suspendedAt, reason |
| credentialing | EndorsementAdded | IssuedEndorsement | GradeBand | endorsementId, credentialId, educatorId, endorsementCode, gradeBand |
| credentialing | EndorsementDefinitionCreated | EndorsementDefinition | EndorsementDefinitionSummary | effectiveFrom |
| credentialing | EndorsementDefinitionDeactivated | EndorsementDefinition | EndorsementDefinitionSummary | deactivatedAt, deactivatedBy |
| credentialing | EndorsementDefinitionUpdated | EndorsementDefinition | EndorsementDefinitionSummary | changes, effectiveFrom, updatedBy |
| credentialing | EndorsementNullified | IssuedEndorsement | GradeBand | endorsementId, credentialId, educatorId, nullifiedAt, reason |
| documents | BulkOperationCompleted | BulkOperation | BulkOperationDetail | type |
| documents | DocumentArchivedToBlob | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | archived_at, archive_tier |
| documents | DocumentCategoryCreated | DocumentCategory | DocumentCategorySummary, DocumentCategoryDetail | retention_years |
| documents | DocumentHardDeleted | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | deleted_at, deletion_reason |
| documents | DocumentMalwareDetected | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | threat_type, threat_name, scan_service |
| documents | DocumentReplaced | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | new_version_id, replaced_by_user_id, old_version_archived |
| documents | DocumentSoftDeleted | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | deleted_by_user_id |
| documents | DocumentViewed | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | viewed_by_user_id, viewed_at |
| documents | LegalHoldApplied | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | applied_by_user_id, justification |
| documents | LegalHoldReleased | Document | DocumentSummary, DocumentMetadata, AuditLogEntry | released_by_user_id, release_reason |
| documents | OverrideRequestApproved | OverrideRequest | OverrideRequestSummary, OverrideRequestDetail | document_ids, approved_by_user_id |
| documents | OverrideRequestCreatedEvent | OverrideRequest | OverrideRequestSummary, OverrideRequestDetail | requested_by, document_ids, approval_count_required |
| documents | OverrideRequestExecutedEvent | OverrideRequest | OverrideRequestSummary, OverrideRequestDetail | executed_at, executed_by |
| documents | RetentionPolicyUpdated | DocumentCategory | DocumentCategorySummary, DocumentCategoryDetail | old_retention_years, new_retention_years |
| epp | ApprovalApplicationDenied | ApprovalApplicationReview | ApprovalApplicationReviewSummary, ApprovalApplicationReviewDetail | approvalApplicationId, eppCode, denialDate, remarks |
| epp | ApprovalApplicationRecommended | ApprovalApplicationReview | ApprovalApplicationReviewSummary, ApprovalApplicationReviewDetail | approvalApplicationId, eppCode, recommendationDate, remarks |
| epp | BulkUploadCompletedEvent | CandidateEnrollment | EnrollmentStatus, EnrollmentProgram, CandidateEnrollmentSummary, CandidateEnrollmentDetail | bulkUploadJobId, processedCount, errorCount, errorReportUrl |
| epp | CandidateEnrollmentRejected | CandidateEnrollment | EnrollmentStatus, EnrollmentProgram, CandidateEnrollmentSummary, CandidateEnrollmentDetail | candidateId, rejectionDate |
| epp | CandidateEnrollmentVerified | CandidateEnrollment | EnrollmentStatus, EnrollmentProgram, CandidateEnrollmentSummary, CandidateEnrollmentDetail | candidateId |
| epp | CandidateExited | CandidateEnrollment | EnrollmentStatus, EnrollmentProgram, CandidateEnrollmentSummary, CandidateEnrollmentDetail | candidateId |
| epp | CandidateStatusChanged | CandidateEnrollment | EnrollmentStatus, EnrollmentProgram, CandidateEnrollmentSummary, CandidateEnrollmentDetail | candidateId, oldStatus, newStatus, effectiveDate |
| epp | CredentialApplicationDenied | CredentialApplicationReview | ApplicationReviewSummary, ApplicationReviewDetail | eppCode, denialDate, remarks |
| epp | CredentialApplicationOnHold | CredentialApplicationReview | ApplicationReviewSummary, ApplicationReviewDetail | eppCode, holdDate, remarks |
| epp | CredentialApplicationRecommended | CredentialApplicationReview | ApplicationReviewSummary, ApplicationReviewDetail | eppCode, recommendedEndorsements, recommendationDate |
| epp | EPPProgramApprovalChanged | EducatorPreparationProvider | PublicProviderSearchResult, PublicProviderDetail, ProPrepConfig, GradeBand, EppType, EppProviderSummary, EppProviderDetail | certificateType, endorsement, approvalDates |
| epp | EPPProviderCreated | EducatorPreparationProvider | PublicProviderSearchResult, PublicProviderDetail, ProPrepConfig, GradeBand, EppType, EppProviderSummary, EppProviderDetail | name, approvedPrograms |
| epp | EPPProviderModified | EducatorPreparationProvider | PublicProviderSearchResult, PublicProviderDetail, ProPrepConfig, GradeBand, EppType, EppProviderSummary, EppProviderDetail | modifiedFields |
| iam | AuthorizationApproved | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, approvedRoles, approverId, approvedAt |
| iam | AuthorizationDenied | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, deniedRoles, approverId, denialReason, deniedAt |
| iam | AuthorizationExpired | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, expiredAt |
| iam | AuthorizationRequested | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, miLoginId |
| iam | AuthorizationRevoked | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, revokedBy, revokedAt, reason |
| iam | AuthorizationUpdated | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, addedRoles, removedRoles, addedEntities, removedEntities |
| iam | AuthorizationWithdrawn | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, withdrawnAt |
| iam | IdentityRecordRetired | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | retiredUniqueId, replacementUniqueId, retiredAt |
| iam | IdentityRecordSplit | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | originalUniqueId, newUniqueIds, splitAt |
| iam | IdentityRequestCancelled | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | userId, employeeRecordId, cancelledAt, autoCancelled |
| iam | IdentityRequestDenied | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | employeeRecordId, denialReason, resolvedBy, resolvedAt |
| iam | IdentityResolutionCompleted | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | resolutionOutcome, userId, employeeRecordId, resolvedBy, resolvedAt |
| iam | IdentityResolutionRequiresReview | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | userId, employeeRecordId |
| iam | NewCitizenUserAuthorized | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, uniqueId, miLoginId, authorizedAt |
| iam | PersonRecordUpdated | IdentityResolutionRequest | UserDetail, IdentityResolutionRequestResponse, IdentityResolutionRequestDetail, Error | userId, employeeRecordId, uniqueId, updatedAt, affectsActiveEmployment |
| iam | UserAccountDeactivated | UserAuthorization | Scope, RequestStatus, PendingApprovalSummary, MiLoginType, AuthorizationStatus, AuthorizationRequestSummary, AuthorizationRequestResponse, AuthorizationDetail | userId, miLoginId, deactivatedAt, reason |
| organizations | LeadAdministratorChanged | Organization | OrganizationWithLeadAdmin, OrganizationDetail, GradeBand | organizationCode, previousEmail, newEmail, effectiveAt |
| organizations | OrganizationDeactivated | Organization | OrganizationWithLeadAdmin, OrganizationDetail, GradeBand | organizationCode, organizationType, deactivatedAt |
| organizations | OrganizationSyncCompleted | SyncLogAggregate | SyncLogEntry, SyncJobResult, Error | syncId, completedAt |
| payments | BulkPaymentCompleted | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | bulk_transaction_id, application_ids, item_count |
| payments | BulkRefundCompletedEvent | RefundRequest | RefundRequestSummary, RefundRequestResponse | refundIds, totalAmount |
| payments | DiscrepancyResolvedEvent | ReconciliationBatch | ReconciliationDiscrepancySummary | resolutionAction |
| payments | MissingPaymentDetected | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | transaction_id, posting_file_date, detected_at |
| payments | PaymentCompleted | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | transaction_id |
| payments | PaymentDiscrepancyDetected | ReconciliationBatch | ReconciliationDiscrepancySummary | transaction_id |
| payments | PaymentFailed | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | transaction_id, failed_at |
| payments | PaymentInitiated | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | transaction_id, initiated_by, initiated_at |
| payments | PaymentReconciled | PaymentTransaction | PaymentTransactionSummary, PaymentInitiationResponse, PaymentConfirmationResult, BulkPaymentInitiationResponse | transaction_id, reconciliation_batch_id, reconciled_at |
| payments | ReconciliationBatchCompleted | ReconciliationBatch | ReconciliationDiscrepancySummary | batch_id, file_date, total_records, matched_count, missing_count, discrepancy_count |
| payments | RefundApproved | RefundRequest | RefundRequestSummary, RefundRequestResponse | approved_by, approved_at |
| payments | RefundFailed | RefundRequest | RefundRequestSummary, RefundRequestResponse | error_message, failed_at |
| payments | RefundRequested | RefundRequest | RefundRequestSummary, RefundRequestResponse | transaction_id, reason |
| proflearning | AttendanceCertified | ProgramAttendance | CertificationStatus, AttendanceRoster | attendeeIds, awardedSCECHs |
| proflearning | CollegeCourseApplied | CollegeCourseCredit | EligibleCourse | educatorId, courseId, awardedSCECHs |
| proflearning | CoordinatorAssigned | ProfessionalLearningSponsor | SponsorDetail | userId, role |
| proflearning | EvaluationRequired | ProgramAttendance | CertificationStatus, AttendanceRoster | attendeeId, programId, dueDate |
| proflearning | InfoRequestedEvent | ProfessionalLearningProgram | SessionLocation, ScechAllocation, ProgramSummary, ProgramStatus, ProgramFormat, ProgramDetail, ProgramContact, ProgramApplicationSummary, ProgramApplicationDetail, ApplicationReviewStatus | infoRequest |
| proflearning | ProgramApplicationSubmitted | ProfessionalLearningProgram | SessionLocation, ScechAllocation, ProgramSummary, ProgramStatus, ProgramFormat, ProgramDetail, ProgramContact, ProgramApplicationSummary, ProgramApplicationDetail, ApplicationReviewStatus | applicationDetails |
| proflearning | ProgramApproved | ProfessionalLearningProgram | SessionLocation, ScechAllocation, ProgramSummary, ProgramStatus, ProgramFormat, ProgramDetail, ProgramContact, ProgramApplicationSummary, ProgramApplicationDetail, ApplicationReviewStatus | approvalPeriod, maxSCECHs |
| proflearning | ProgramModified | ProfessionalLearningProgram | SessionLocation, ScechAllocation, ProgramSummary, ProgramStatus, ProgramFormat, ProgramDetail, ProgramContact, ProgramApplicationSummary, ProgramApplicationDetail, ApplicationReviewStatus | changes, requiresReapproval |
| proflearning | ProgramRejected | ProfessionalLearningProgram | SessionLocation, ScechAllocation, ProgramSummary, ProgramStatus, ProgramFormat, ProgramDetail, ProgramContact, ProgramApplicationSummary, ProgramApplicationDetail, ApplicationReviewStatus | rejectionReason |
| proflearning | SCECHAwardAdjusted | ProgramAttendance | CertificationStatus, AttendanceRoster | attendeeId, oldAmount, newAmount, justification |
| proflearning | SCECHCorrectionApproved | SCECHCorrectionRequest | CorrectionRequestSummary, CorrectionRequestStatus | approvedBy, newSCECHValue |
| proflearning | SCECHCorrectionDenied | SCECHCorrectionRequest | CorrectionRequestSummary, CorrectionRequestStatus | deniedBy, reason |
| proflearning | SCECHCorrectionRequested | SCECHCorrectionRequest | CorrectionRequestSummary, CorrectionRequestStatus | attendeeId, requestedChange |
| profpractices | DisclosureReviewCompleted | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | reviewerId, finalStatus, reviewCompletedAt |
| profpractices | DisclosureReviewStarted | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | reviewerId, reviewStartedAt |
| profpractices | DisclosureRoutedToWorklist | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | routedAt, routingReason |
| profpractices | DisclosureStatusChanged | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | educatorId, previousStatus, newStatus, changedAt, changedBy |
| profpractices | DisclosureSubmitted | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | educatorId, submittedAt |
| profpractices | NASDTECRecordMatched | ExternalBackgroundCheck | ExternalBackgroundCheck | educatorId, jurisdiction, transactionDate, clearinghouseId, clearinghouseUrl, requiresReview |
| profpractices | NonSystemActionLogged | Disclosure | PaginationMetadata, DisclosureType, DisclosureSummary, DisclosureStatus, DisclosureDetail | actionType, NotifiedSchoolDistrict |
| profpractices | PPRAccountMarkerChanged | EducatorPPRStatus | RosterEligibilityAssessment, PprClearanceAssessment, EducatorPprStatus | educatorId, markerType, MandatoryHoldRequirement |
| profpractices | PPRClearanceAssessmentChanged | EducatorPPRStatus | RosterEligibilityAssessment, PprClearanceAssessment, EducatorPprStatus | educatorId, previousAssessment, newAssessment, triggeredBy, DisclosureStatusChange |
| profpractices | PPRReminderRequired | EducatorPPRStatus | RosterEligibilityAssessment, PprClearanceAssessment, EducatorPprStatus | educatorId, reminderType, dueDate, currentComplianceStatus |
| profpractices | RapBackNotificationReceived | ExternalBackgroundCheck | ExternalBackgroundCheck | educatorId, notificationDate, tcn, judicialMarker |
| staffing | AssignmentEndedEvent | Assignment | SpecializedFunding, AssignmentSummary, AssignmentStatus, AssignmentDetail | endReason |
| staffing | AuditReportFinalized | AuditWorkItem | DistrictAuditReport, AuditWorkItemSummary, AuditType, AuditStatus, AuditFinding | auditTimestamp, auditorUserId |
| staffing | CollectionCertified | Collection | QualityReviewResult, CollectionSummary, CollectionStatus, CertificationStatus | collectionType, certificationTimestamp, certifyingUserId |
| staffing | CollectionExceptionApprovedEvent | CollectionException | ExceptionStatus | exceptionRequestId, approvedExtensionDate |
| staffing | CollectionExceptionDeniedEvent | CollectionException | ExceptionStatus | exceptionRequestId, denialReason |
| staffing | CollectionExceptionRequestedEvent | CollectionException | ExceptionStatus | exceptionRequestId, collectionId, entityCode, requestedExtensionDate |
| staffing | CollectionOpened | Collection | QualityReviewResult, CollectionSummary, CollectionStatus, CertificationStatus | carriedForwardCount, excludedCount |
| staffing | CredentialErrorJustificationSubmitted | Assignment | SpecializedFunding, AssignmentSummary, AssignmentStatus, AssignmentDetail | entityCode, uniqueId, scedCode |
| staffing | DemographicDataUpdatedEvent | EmployeeRoster | NearMatchResolutionRequired, EmploymentStatus, EmployeeSummary, EmployeeRecordStatus, EmployeeDemographics, EducatorSearchResult | updatedFields, updateSource, DistrictUser, CitizenUser |
| staffing | DocumentRequestCreatedEvent | AuditWorkItem | DistrictAuditReport, AuditWorkItemSummary, AuditType, AuditStatus, AuditFinding | documentRequestId, documentType, details |
| staffing | EmployeeAddedToRoster | EmployeeRoster | NearMatchResolutionRequired, EmploymentStatus, EmployeeSummary, EmployeeRecordStatus, EmployeeDemographics, EducatorSearchResult | employmentStartDate |
| staffing | EmployeeAssignedToPosition | Assignment | SpecializedFunding, AssignmentSummary, AssignmentStatus, AssignmentDetail | entityCode, uniqueId, assignmentStartDate |
| staffing | EmploymentStatusChanged | EmployeeRoster | NearMatchResolutionRequired, EmploymentStatus, EmployeeSummary, EmployeeRecordStatus, EmployeeDemographics, EducatorSearchResult | oldStatus, newStatus, effectiveDate, separationReason |
| staffing | EvaluationOutcomeRecorded | EmployeeRoster | NearMatchResolutionRequired, EmploymentStatus, EmployeeSummary, EmployeeRecordStatus, EmployeeDemographics, EducatorSearchResult | schoolYear, evaluationScale, outcome |
| staffing | NewTeacherMentorAssigned | EmployeeRoster | NearMatchResolutionRequired, EmploymentStatus, EmployeeSummary, EmployeeRecordStatus, EmployeeDemographics, EducatorSearchResult | menteeUniqueId, mentorUniqueId, mentorName, schoolYear |
| staffing | PositionCreatedEvent | PositionRoster | PositionSummary, PositionStatus, PositionDetail, EducationJobType | entityCode |
| staffing | PositionStatusChangedEvent | PositionRoster | PositionSummary, PositionStatus, PositionDetail, EducationJobType | oldStatus, newStatus, statusDate |

### Events whose aggregate has no API schema

- credentialing: AssessmentResultImportedEvent (emitted by AssessmentResult), payload resultId, educatorId, assessmentIdentifier, scoreValue
- documents: BulkFileUploadedEvent (emitted by StagingFileAggregate), payload upload_id, functional_area, import_type, blob_path
- profpractices: PPRResponseSubmitted (emitted by ProfessionalPracticeResponse), payload responseId, educatorId, responseDate, newDisclosuresReported, createdDisclosureIds, responseStatus


## What this suggests

1. **External APIs mix with application APIs in 15 operations**, 13 in staffing and 2 in proflearning. Where the external API is its own service with no code shared with the application API, each of these becomes two operations: the application one (delegated user token, under the UI host's `/api`) and an external one (client credentials, on the external hostname), or just one of the two. Staffing's 13 are all writes that the external client can also perform; that is the first thing to settle with the client.
2. **Organizations has no `x-access` on any operation.** Its eight operations include admin and sync endpoints and two the other areas call to look up organizations, which are service API candidates (in-mesh, never through APIM). That needs an author's decision, not a guess.
3. **Only 9 operations are external-only**, across 6 areas. A separate external service would be small today: credentialing 2, epp 2, iam 2, payments 1, proflearning 1 (plus 2 mixed), profpractices 1.
4. **Spec headers**: see section 2. Hard-coded hostnames in `servers` belong to one place per kind of API (application, external) rather than in each area's spec.
5. **Event payloads and API schemas are different views of an aggregate**, as intended. The gaps that are worth reading are the identifier and "something else" fields in section 3, not the timestamps, actors and reasons. Several events name the same person differently from the schema (applicantId in the event, educatorId in the schema), and some events carry fields the aggregate's API never exposes (for example the effective-date fields on the definition events).
6. **Ambiguous projections**: shared schemas (PaginationMetadata, ErrorResponse, DefinitionVersion) have several candidate sources, and `projectsFrom` is only set where it is unambiguous. They are listed in each reconcile run's report.

## How to rerun

```
node scripts/reconcile-api.ts <area> [--apply]
node scripts/export-docs.ts <area> --docs api
node scripts/compare-api.ts <area> --generated <file>
node scripts/api-report.ts > report.md
```
