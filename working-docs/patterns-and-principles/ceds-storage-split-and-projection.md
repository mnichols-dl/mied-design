# CEDS Storage Split and Projection Review

**Status:** Draft working notes, 2026-10-06. Exploratory. Nothing here is a decision; the outbox approach in particular is a candidate, not a commitment. Open questions are tracked in `hub/tracking/open-questions.md` (rows dated 2026-10-06), not here.

Companion to [ceds-jsonld-integration.md](ceds-jsonld-integration.md), which sets the principle (CEDS JSON-LD is an integration concern, Azure SQL stays authoritative). This note tests that principle against the CEDS ontology and the client's extension material, then says what to project, into which collection, and what to keep out.

## Sources reviewed

| Source | Used for |
|---|---|
| `client-inputs/ceds-ontology/CEDS-Ontology.rdf` | Baseline. Version 14.0.0.0. About 400 entity classes, 967 option sets. The ontology labels classes by CEDS element code (for example `C200079` is Credential Award), so mapping needs label lookups. |
| `CEPI Extension Workbook - LM.xlsx` | Michigan-only and pending extensions, each tagged with the Cosmos collection it loads into. This is the best available statement of what CEPI expects in Cosmos. |
| `Credentials Workbook - Updated for Ontology.xlsx` | Credential definition and award mappings, status proposals, record versioning convention. |
| `An Intro to CEDS in Transactional Systems` (deck, marked not final) | Record management fields, identifier model, JSON-LD conventions. Only the first part was read. |
| Solution graph (`design/graph/MiEdWorkforce.ttl`) and working docs | Aggregates, events, databases. Read at entity and event level, not field level. |

## Selection rule

Project to CEDS when the thing is a **fact about a person, organization or credential that other state systems would consume**. Keep internal when it is **workflow, money, access control, content, or sensitive adjudication**. Project outcomes, never process state.

## Tiers

| Tier | Meaning | Examples |
|---|---|---|
| A | Azure SQL authoritative, projected to the CEDS Cosmos store | Credential definitions and awards, employment and assignments, PD activity and attendance, EPP enrollment |
| B | Azure SQL (or Cosmos/Blob) only, never projected | Applications, payments, authorization, documents, communications, disclosures |
| C | Inbound CEDS | Organizations from CEPI |

## Tier A: collections and recommendations

Collection names are the CEPI `Cosmos Database Collection` values in the extension workbook. "Ext" means the property is a Michigan or pending extension, so it needs an approved row in the workbook before it can be emitted.

### CredentialDefinition

| | |
|---|---|
| SQL sources | `credential_definitions`, `endorsement_definitions`, `definition_versions` (CredentialDefinition, EndorsementDefinition aggregates, CredentialCategory). From EPP: `approved_certificate_categories`, `approved_endorsements`. |
| CEDS classes | Credential Definition, Credential Definition Category (System plus Type), Credential Definition Identification, Grade Level Range Detail, Credential Offered, Credential Definition Assessment (ext class), Credential Definition Course Code (ext class) |
| Extensions in play | Professional Credential Type, Credential Recommendation Start/End Date, Enrollment Start/End Date, Teacher Preparation Program Type, Credential Definition Assessment and Course Code start/end dates, Credential Definition Date Inactive |
| Trigger | Definition created, updated, deprecated or retired events (six exist for credentials and endorsements) |
| Keep in SQL | Fee schedules, requirement rules, validity and renewal logic. They drive processing and need referential integrity. |
| Recommendation | Project a slice of each definition. Do **not** store these natively as JSON-LD, despite Pattern 3's example list. Use Credential Definition Category System values such as `Credential Profession` to carry the credential categories (Teacher, CTE, Administrator, Counselor, Psychologist, Nurse). EPP approval windows map to Credential Offered plus the recommendation and enrollment date extensions. |

### CredentialAward

| | |
|---|---|
| SQL sources | `issued_credentials`, `issued_endorsements`, `issuance_history` (IssuedCredential, IssuedEndorsement aggregates) |
| CEDS classes | Credential Award, Credential Issuer, Credential Award Relationship Detail (endorsement to certificate), Credential Award Credit |
| Extensions in play | Credential Award Identifier and Identification System (approved), Credential Award Status and Status Date (proposal 1033, pending), Jurisdiction Organization, Recommending Organization (the EPP), Out of State College, Professional Certificate or License Number |
| Trigger | CredentialIssued, Renewed, Suspended, Revoked, Expired, EndorsementAdded, EndorsementNullified |
| Keep in SQL | Applications, application status, processor notes, verification, payment status, manual review state |
| Recommendation | Key by PIC. The Credentials workbook says the grant team only loads awards for people with a PIC and a real name. Issuer is an IRI to the State of Michigan organization. A status mapping from the credentialing `CredentialStatus` value object to the pending Credential Award Status option set is needed; the options are not yet approved. |

### AssessmentRegistration

| | |
|---|---|
| SQL sources | `assessment_results` (AssessmentResult aggregate, from the Pearson MTTC file) |
| CEDS classes | Assessment Registration, Assessment Result, Teacher Education Credential Exam |
| Extensions in play | Report Institution 1 to 3, OtherState, High and Low Grade Level, Assessment Score Metric Type |
| Trigger | AssessmentResultImported |
| Keep in SQL | Assessment definitions and passing criteria, import file handling, unmatched-record resolution. Only the Credential Definition Assessment link is projected. |
| Recommendation | Project results once matched to a Unique ID and PIC. Never carry SSN. |

### K12Staff

| | |
|---|---|
| SQL sources | `employee_roster`, `employment_history`, `assignments`, `assignment_course_details`, `spots` |
| CEDS classes | Employment, K12 Staff Employment, K12 Staff Assignment, Assignment, Organization Program Type, Job Position (linked) |
| Extensions in play | Employment Start Date, Profile Status, K12 Staff Classification, Classroom Position Type, Hourly Wage, Migrant Education Program fields, Special Education Age Group Taught, Job Position and Staff Compensation links, K12 Staff Assignment Organization |
| Trigger | Decision needed (see open questions). Candidates: on every employment and assignment change, or only on CollectionCertified. Certified data is immutable, so a certification trigger removes churn and matches the data warehouse closeout. |
| Keep in SQL | Collections workflow, quality review results, audit work items and findings, document requests, exceptions, new-teacher monitoring and mentor assignments |
| Recommendation | Treat as the main candidate for certified-only projection. Project Staff Experience (years of prior teaching) only, from new-teacher monitoring. |

### Job

| | |
|---|---|
| SQL sources | `position_roster`, `positions` |
| CEDS classes | Job, Job Position, Job Detail, Job Position Status Detail |
| Extensions in play | Education Job Type, Evaluation Required Indicator |
| Trigger | PositionCreated, PositionStatusChanged |
| Recommendation | Project position definitions. FTE health indicators and approved FTE tracking stay in SQL. |

### StaffEvaluation

| | |
|---|---|
| SQL sources | `evaluation_outcomes` |
| CEDS classes | Staff Evaluation (Outcome, Score or Rating, System), Staff Evaluation Part |
| Extensions in play | Staff Evaluation Scale |
| Trigger | EvaluationOutcomeRecorded |
| Keep in SQL | Appeal tracking and deadlines |

### ProfessionalDevelopmentActivity

| | |
|---|---|
| SQL sources | `programs`, `program_sessions`, `program_categories`, `sponsors` (sponsor is an Organization from EEM) |
| CEDS classes | Professional Development Activity, Professional Development Session, Organization (sponsor link) |
| Extensions in play | Approval Start/End Date, Sponsoring Agency Name, Content Subcategory, District Provided PD Indicator, Learner Activity Prerequisite, Technical Requirements, Sponsor Agrees to College Conversion Program Rules |
| Trigger | ProgramApproved, ProgramModified |
| Keep in SQL | Program applications and review, coordinator assignments, evaluation templates and submissions, bookmarks, correction request workflow |
| Recommendation | Project approved programs only. Do not use Course Section, which is how the current EPP and PL notes describe the mapping. |

### StaffProfessionalDevelopment

| | |
|---|---|
| SQL sources | `attendees`, `scech_awards`, `college_course_applications` (PL). `candidate_enrollments`, `enrollment_programs`, `enrollment_status_history` (EPP). |
| CEDS classes | Staff Professional Development Activity, Program Participation Teacher Prep, Role Status (Teacher Preparation Program Enrollment Status), Teacher Education Credential Exam (link) |
| Extensions in play | Professional Development Used As Credential Basis, Alternative Route Placement Status, Program Exit Reason, Student Teaching Status, and about 20 other teacher-prep properties |
| Trigger | AttendanceCertified, SCECHAwardAdjusted, CandidateStatusChanged, CandidateExited |
| Keep in SQL | EPP application reviews, recommendations, remarks, worklists, bulk upload jobs, SCECH correction requests |
| Recommendation | Project awarded SCECH after certification (awards are immutable once certified). Candidate enrollment projects as teacher-prep participation, not as K12Student. |

### Person

| | |
|---|---|
| SQL sources | IAM identity link (Mi-Key unique ID and PIC, name, email), Staffing `education_history` |
| CEDS classes | Person, Person Name, Person Other Name, Person Identification, Person Email Address, Person Degree Or Certificate |
| Extensions in play | Highest Level of Education Completed and its Detail class, Staff Member Identification System, Location Address additions |
| Trigger | PersonRecordUpdated, IdentityResolutionCompleted |
| Recommendation | Mi-Key and CEPI are the person master. Project only the identity link and fields MiEd actually owns. Confirm whether MiEd writes this collection at all. Never project SSN. |

### Common

| | |
|---|---|
| CEDS classes | Record Status, Data Collection, Grade Level Range Detail |
| Extensions in play | Record Start/End Date Time, Data Collection Certification Open/Close Date, Data Corrections End Date |
| Recommendation | Staffing `collection_definitions` project to Data Collection; the extensions match the collection certification model almost field for field. Record Status carries the version window on every projected document. |

## Tier C: Organizations (inbound)

CEDS classes: Organization, Organization Identification, Organization Relationship, Organization Program Type, Location Address, with the Organization Type extension.

The design currently describes this three ways: a Cosmos replica (pattern doc and tech standards), SQL `organizations-db` with a `ceds_metadata` JSON column (databases doc), and Synapse loading CEDS IDS tables (EEM integration notes). Recommendation: pick SQL. IAM needs sub-10ms transitive hierarchy lookups, which a materialized relational hierarchy serves directly, and the full payload in `ceds_metadata` preserves interoperability. A separate Cosmos replica adds nothing for a read-only reference set.

## Tier B: keep out of CEDS

| Area | Reason |
|---|---|
| Credential applications, processing notes, review state | Workflow. Only the resulting award is projected. |
| Payments, refunds, reconciliation | No CEDS fit. CEDS Financial Account models organization finance. |
| Professional practices (disclosures, Rap Back, NASDTEC, PPR, account markers) | CEDS Incident is student-discipline shaped. The outcome is already expressed through Credential Award Status. Criminal history should not sit in a shared state store. Rap sheets are never persisted anywhere. |
| IAM authorizations, roles, permissions, impersonation, inactivity policy | Hot path, sub-10ms, RBAC with scope. CEDS Authorization and Authentication exist but carry only an application role name and dates. |
| Communications, documents, reporting, question sets, worklists, audit logs | No CEDS equivalent, or operational content. |
| Staffing collection workflow, audit, exceptions, quality review | Process state. Only certified outputs and the collection definition project. |

## Candidate projection mechanism (not decided)

The review found the following gaps in the current design, so something like this is needed. The specifics are open for discussion.

1. **No outbox exists.** Publishing to Service Bus after the SQL commit is a dual write; a crash between the two loses the event.
2. **Current events are notifications, not snapshots.** For example `CredentialRevoked` carries `{credentialId, educatorId, revokedAt, reason}`. That cannot build a Credential Award document. Staffing and EPP events have no topic, and none lists the reconciliation worker as a consumer.
3. **Candidate approach:** a transactional outbox table in each projecting database, written in the same transaction as the state change, holding a versioned snapshot of the aggregate slice and the CEDS collection it targets. A relay publishes to Service Bus (CloudEvents). The reconciliation worker maps, validates with SHACL and upserts to Cosmos. This is separate from the internal domain events.
4. **Alternatives to weigh:** SQL change tracking or change event streaming feeding the worker (no outbox code, but couples the worker to table shape); polling temporal tables by period (simple, but delayed and weak on deletes).

Design points that hold under any option:

| Concern | Note |
|---|---|
| Idempotency | Key the upsert on aggregate id plus version so replays are safe. |
| Ordering | Preserve order per aggregate; partition the topic or session on the aggregate id. |
| Record versioning | CEDS uses `RecordStartDateTime` and `RecordEndDateTime`, with 9999-12-31 meaning current. The Credentials workbook uses the source ModifiedDate for start. SQL system-versioned period columns already use the same sentinel. |
| Deletes | CEDS has no soft-delete flag (issue 607 is a proposal). A tombstone convention is needed. |
| Rebuild | Temporal tables should make full re-projection possible after a mapping or CEDS version change. |
| Validation failures | Dead letter, never block the operational transaction (per the existing pattern doc). |
| Partition keys | PIC for person-centric collections (CredentialAward, K12Staff, StaffProfessionalDevelopment); definition id or organization code for definitions. Decide per collection. |

## Mapping register (needed, does not exist yet)

The graph carries `cedsAlignment` on one node (OrganizationAggregate). Extensions need CEPI manager approval before use. A register should hold, per projected field: SQL table and column, CEDS class and property code, extension status (baseline, approved ext, pending proposal), and the owning collection. It could live as data in the design repo and feed both the worker and the SHACL shapes.

## Corrections to existing CEDS alignment notes

These are recommendations; the source docs have not been edited.

| Doc | Current text | Better fit (CEDS 14.0.0.0 labels) |
|---|---|---|
| `solution-databases-WORKING.md`, EPP | Candidate data aligned with CEDS K12Student | Program Participation Teacher Prep with Role Status |
| EPP | Program categories map to CEDS CourseSection and CredentialType | Credential Definition Category System and Type |
| Staffing | Education history per PsStudentAcademicRecord | Person Degree Or Certificate; Highest Level of Education Completed is a CEPI extension on Person |
| Staffing | K12StaffEmployment, K12StaffAssignment | Correct concepts, but v14 labels are "K12 Staff Employment" and "K12 Staff Assignment"; Employment and Assignment are the parent classes |
| `ceds-jsonld-integration.md`, Pattern 3 | Credential type metadata and endorsement types as native JSON-LD candidates | Conflicts with the pattern's own poor-candidate rule (versioned, referenced by applications). Keep in SQL and project. |
| Organizations docs | Cosmos replica vs SQL with JSON column vs IDS tables | Choose one (recommend SQL) |

## What this review did not cover

Field-level mapping, FDD detail beyond what the graph captures, the later slides of the CEDS deck, and SHACL shapes. The ontology file is the base ontology; the extension workbook is dated 2026-01-28 and marked work in progress.
