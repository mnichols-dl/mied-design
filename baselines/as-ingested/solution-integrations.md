# MiEdWorkforce External System Integrations

## 1. MiLogin (Worker, Business, Citizen)

**System Identity**
- **Purpose**: Primary authentication platform for Michigan citizens and state workers; provides identity verification only (no authorization/permissions)
- **Classification**: Inbound
- **Phase**: 1

**Integration Points**
- **Protocol**: OpenID Connect / OAuth 2.0
- **Integration Flow**: 
  - User initiates login -> Redirect to MiLogin -> Authentication -> Return to MiEdWorkforce with JWT token
  - Token validation and claims extraction
  - MiEdWorkforce performs all authorization internally based on identity
- **Data Exchanged**: 
  - Primary identifiers: `email`, `uniqueSecurityName`
  - Additional claims: `displayName`, `name`, `preferred_username`, `userType`, `email_verified`, `realmName`, `tenantId`
  - **Note**: MiLogin provides ONLY identity claims, NOT permissions or role assertions. Authorization is MiEdWorkforce's responsibility.
- **Frequency**: Real-time (per user session)

**Application Management**
- **Self-Service Portal**: Automated Application Onboarding Administrator Application
  - Access provisioned to application approvers (3-4 team members including project leads)
  - Enables self-management of app credentials, configuration, and settings
  - **[NEEDS INPUT: Specific documentation link for the Automated Application Onboarding Administrator Application]**
  - **[QUESTION: QA environment references "milogintpqa" - what does "TP" stand for?]**
- **User Approval Strategy**: Auto-approve all user types (citizen, business, worker) at the MiLogin layer
  - Authorization and access control performed entirely within MiEdWorkforce
  - **[OPEN QUESTION: Should MiEdWorkforce admins logging in via MiLogin Worker require manual approval?]**

**Environment Strategy**
- **Local/DEV/QA**: MiLogin QA environment (MiLogin Dev is not accessible outside the SOM network, making it unsuitable for external developers and local development workflows)
- **STAGE**: MiLogin QA environment (MiLogin has no staging environment)
- **PROD**: MiLogin PRD environment

**What We Need from MiLogin**
- **Access & Credentials**: 
  - OAuth 2.0 Client ID and Client Secret managed via Automated Application Onboarding Administrator Application
  - Credentials needed for:
    - Local/Dev/QA (all using MiLogin QA)
    - Staging (using MiLogin QA)
    - Prod (using MiLogin PRD)
  - **[NEEDS INPUT: When do we need credentials? Lead time from MiLogin team?]**
  - **[NEEDS INPUT: Are we reusing existing containers or creating new ones for MiEdWorkforce?]**
- **Management Portal Access**: 
  - Access to MiLogin management portal for test user creation and management
  - **[NEEDS INPUT: Who needs portal access? How to request?]**
- **Credential Management Strategy**: 
  - **[NEEDS INPUT: Credential rotation policy? Expiration periods?]**
  - **[NEEDS INPUT: Who manages credential updates? What's the process?]**
- **Documentation**: 
  - OpenID Connect discovery endpoint for each environment (DEV, QA, PRD)
  - JWT token structure and claims definitions
  - Token validation procedures and signature verification
  - Supported authentication flows (authorization code flow expected)
  - **Test user management documentation**: How to create/manage test accounts in MiLogin portal
  - **[NEEDS INPUT: MFA requirements - will MFA be required in lower environments (QA)? When logging in from personal devices?]**
- **Test Data**: 
  - DEV/Local: Process for creating fictitious test accounts with various userTypes (worker, business user, citizen)
  - QAT: Guidance on creating test accounts with realistic entity affiliations
  - **[NEEDS INPUT: Can we create arbitrary/fake email addresses in QA? Or must they be valid format?]**

**Token Claims Structure**
```json
{
  "email": "user@example.com",              // Primary identifier
  "uniqueSecurityName": "text",             // Secondary identifier
  "displayName": "John Doe",
  "name": "John Doe",
  "preferred_username": "johndoe",
  "userType": "citizen|worker|business",    // User classification
  "email_verified": true,
  "realmName": "text",
  "ext": {
    "tenantId": "text"
  },
  "iss": "https://milogin<dev/qat/prd>",
  "aud": "client_id",
  "sub": "unique_user_id",
  "iat": 1731254728,
  "exp": 1731261928
}
```

**Implementation Notes**
- **Dependencies**: None (foundational)
- **Blocks**: All other integrations (authentication required for system access)
- **Data Mapping**: 
  - `email` and/or `uniqueSecurityName` map to MiEdWorkforce user identity
  - `userType` informs MiEdWorkforce authorization logic
  - Authorization, entity affiliations, and permissions are determined by MiEdWorkforce based on identity

**Special Considerations**: 
  - MiLogin Dev environment not used due to SOM network accessibility restrictions
  - All lower environments (Local, Dev, QA, Staging) connect to MiLogin QA
  - MiLogin provides identity verification only; all authorization logic is internal to MiEdWorkforce
  - Auto-approval configured for all user types at MiLogin layer
  - MFA requirements handled by MiLogin for all user types
  - SP-initiated flow is primary authentication pattern (IdP-initiated also supported)
  - Underlying technology appears to be IBM ISV tooling
  - **[OPEN QUESTION: Should admin users require manual approval in MiLogin?]**

**Outstanding Questions**
1. **Admin User Approval**: Should MiEdWorkforce system administrators logging in via MiLogin Worker require manual approval rather than auto-approval?
2. **QA Environment Naming**: QA environment references "milogintpqa" - what does "TP" designation mean?
3. **User Management Portal**: Where specifically do we manage test users for MiLogin QA and PRD environments?
4. **Lower Environment Setup**: What are the specific steps to configure MiEdWorkforce in MiLogin QA for local/dev/qa/staging environments?
5. **Test Email Addresses**: Can we create arbitrary/fake emails in QA (e.g., test@fake.com), or must they be valid format?

**Reference Documentation**
- MiLogin Guide for Application Developers: https://dev.azure.com/SOM-EDCIM/EIAM-AS/_wiki/wikis/EIAM-AS.wiki/27/MiLogin-Guide-for-Application-Developers
- Automated Application Onboarding Administrator Application
- **[NEEDS INPUT: Documentation link for Automated Application Onboarding Administrator Application]**
- **[NEEDS INPUT: User management portal documentation and access procedures]**
- **[NEEDS INPUT: Step-by-step guide for configuring applications in lower environments]**

---

## 2. Mi-Key

**System Identity**
- **Purpose**: Centralized identity management system providing unique identifiers (Unique ID/PIC) through probabilistic matching; integrates eScholar's COTS Uniq-Id with SOM-developed Internal Services gateway
- **Classification**: Bidirectional
- **Phase**: 1

**Integration Points**
- **Architecture**: 
  - MiEdWorkforce > Internal Services (audit/event grid) > Uniq-Id matching engine
- **API Endpoints**: (See Mi-Key Swagger documentation)
  - **Match/Validate Identity**: Submit demographics, returns Match/No Match/Near Match with Unique ID or candidates
  - **Resolve Near Match**: Submit admin decision (confirm match, create new, cancel) 
    - **[CONFIRM: PUT /personid/transactions/{transactionId} with action payload?]**
  - **Update Demographics**: Update existing person record (triggers re-validation)
  - **Retrieve Person by Unique ID**: Query core demographics using Unique ID
  - **Search by Demographics**: Pre-search before adding new person (supports "Search then Add")
  - **Bulk/Async Processing**: Submit batches of person records, poll for status/results
  - **Query Transaction Status**: Check status of pending/expired near match transactions
    - **[CONFIRM: GET /personid/transactions/{transactionId} returns status + expiration?]**
- **Trigger**: 
  - Real-time: Citizen account creation, single employee adds, demographic updates, identity searches
  - Batch: District roster imports, EPP candidate uploads, professional learning attendance lists
- **Data Exchanged**: 
  - **Inbound to Mi-Key**: 
    - Demographics (name, DOB, SSN, sex, race/ethnicity, tribal affiliation, alternate names)
    - Resolution commands (action: CREATE_NEW/USE_EXISTING/CANCEL/LINK_IDS, justification, selectedUniqueId)
    - Submission context (requestType, submittedBy, organizationCode) **[CONFIRM: Required fields?]**
  - **Outbound from Mi-Key**: 
    - Unique ID, match result (Match/No Match/Near Match), potential match candidates
    - Batch/Transaction IDs
    - Transaction metadata (status, expiresAt, confidenceScores) **[CONFIRM: Available fields?]**
  - **Historical Data**: Name change history available for matching
- **Frequency**: Real-time per operation; Batch processing for bulk updates

**Environment Strategy**
- **DEV/Local**: Mocked - JSON responses simulating Match/No Match/Near Match outcomes
- **QA/STAGE**: Mi-Key STAGE environment
- **PROD**: Mi-Key Production

**What We Need from Mi-Key**
- **Access & Credentials**: 
  - API token (contact: GilmoreS1@michigan.gov)
  - APIM access via SOM VPN: https://mi-key.a7ak8.developer.apim.az.state.mi.us/
  - **[NEEDS INPUT: Lead time for aquiring tokens for subsequent environments?]**
- **Documentation**: 
  - Swagger (DEV: https://dev.mi-key.escholar.com/uid/docs/docs.html#, STAGE: https://stg.mi-key.escholar.com/uid/docs/docs.html#)
  - Online Guide: https://escholar-uniq-id-help.scrollhelp.site/uniqid/get-started
  - Matching Engine Logic: eScholarUniqID_PersonID_MatchingLogicOverview_v2023.docx
  - Integration Mapping: APIs by Functional Use Case.xlsx
  - Bulk API specs: batch limits, polling intervals, error handling
  - Rate limits (sync vs. async endpoints)
- **Test Data**: 
  - Sample Unique IDs (historical PICs, new UICs, name change scenarios)
  - Match outcome test cases (exact match, near matches, no match)
  - Bulk processing test files (100+ records with mixed outcomes)

**Key Integration Scenarios**

**Citizen Account Creation:**
- Submit demographics > Mi-Key returns Match/No Match/Near Match
- Near Match: Creates admin task, user cannot proceed until resolved
- Admin resolves > Re-submit decision > Unique ID returned

**District "Search then Add":**
- Search by demographics > Mi-Key returns potential matches
- User selects match OR creates new > Unique ID obtained for roster

**Bulk District Roster:**
- Upload file > Submit batch to async endpoint > Poll for results
- Near Matches route to admin queue

**Demographic Updates:**
- Update primary fields (name, SSN) > Submit to Mi-Key > Re-validation occurs
- Near Match possible if update creates ambiguity

**Identity Resolution for External Systems:**
- Rap Back: Receive demographics from MSP > Query Mi-Key for Unique ID match
- NASDTEC: Use Unique ID > Query Mi-Key for name/DOB > Search clearinghouse
- EPP/SCECH: Import rosters with Unique IDs > Validate against Mi-Key

**Implementation Notes**
- **Dependencies**: None (foundational)
- **Blocks**: All person-level integrations (Rap Back, NASDTEC, TSDL, CTEIS, EPP, SCECH)
- **Data Mapping**: eScholar model <--> CEDS alignment
- **Special Considerations**: 
  - Capture Batch/Transaction IDs for audit trail
  - Identity Admin worklist required for Near Match resolution
  - User notifications (on-screen + email) for account status
  - Cannot mock citizen authentication flow - must test with Mi-Key DEV
  - Bulk processing: handle async status polling and partial failures
  - Leverage historical name changes for long-term matching

**Outstanding Questions**
1. Separate API tokens for STAGE/PROD? Lead time?
2. CEDS <--> eScholar mapping fully documented?
3. Rate limits: sync endpoints? Batch size limits for async?
4. Status polling interval recommendation for bulk processing?
5. Test data available in STAGE: historical PICs/UICs, name change scenarios, bulk files?

**Reference Documentation**
- Mi-Key DevOps: https://dev.azure.com/SOM-MDECEPI/CEPI%20-%20Education%20Unique%20Person%20Identifier(EUPI)
- Swagger: DEV/STAGE environments linked above
- Integration Mapping: APIs by Functional Use Case.xlsx
- Data Migration Plan: Mi-Key Staffing Data Migration Plan.docx


NOTES:
Will get access to Internal Services API in Dev (will not access directly the eScholar APIs)
Dev will use this to test functionality of API, will use mock version locally and in MiEdWorkforce for development
Key Contact: Shirin Gilmore (API issues)
Internal Services is still being developed (2026-02-02) tickets go through Azure DevOps project
every other week (15-30 minutes)

---

## 3. Educational Entity Master (EEM)

**System Identity**
- **Purpose**: Official source of truth for Organization hierarchy (ISDs, LEAs, schools) and Lead Administrator assignments; used for authorization scope validation and approval routing
- **Classification**: Inbound
- **Phase**: 3

**Integration Points**
- **Data Format**: CEDS JSON-LD structure (recently agreed with client)
- **API Endpoints**: **[NEEDS INPUT: Specific endpoint names/paths]**
  - **Organization Search/Lookup**: Search entities by name, code, or type (typeahead/predictive search with debouncing)
  - **Organization Details by Code**: Retrieve full entity record including hierarchy and Lead Admin
  - **Organization Hierarchy Query**: Retrieve parent/child relationships for transitive authorization resolution
  - **Organization List by Type**: Filter entities by type (ISD/District/Building/Nonpublic/EPP/PLS)
  - **Lead Administrator Lookup**: Query which entities a user is Lead Admin for (by email/identity)
  - **Organization Validation**: Verify entity code exists and is active
  - **[NEEDS INPUT: Batch/bulk endpoint for ETL refresh?]**
- **Trigger**: 
  - Real-time: Authorization request submission, Organization search in UI, permission checks with transitive resolution
  - Batch: Daily ETL for local Organization cache refresh
- **Data Exchanged**: 
  - **Organization Core**: Organization Code, Organization Name, Entity Type (ISD/District/Building/Nonpublic School/EPP/Professional Learning Sponsor)
  - **Hierarchy**: Parent Organization Code(s), full ancestor path for transitive authorization
  - **Status**: Active/Inactive flag
  - **Contact**: Lead Administrator identification **[NEEDS INPUT: How is Lead Admin identified? Email? MiLogin ID? Name + contact info?]**
  - **Address**: Physical address (street, city, state, zip) **[NEEDS INPUT: Required for all entity types?]**
  - **[NEEDS INPUT: Other fields in CEDS JSON-LD structure we should consume?]**
- **Frequency**: 
  - Real-time: High-frequency for authorization checks, Organization search, scope validation
  - Daily ETL: **[NEEDS INPUT: Time of day? Overnight batch window?]**

**Environment Strategy**
- **DEV**: Mocked - Local JSON-LD files with sample entity hierarchy (e.g. 3 ISDs, 15 Districts, 50 Buildings, sample Lead Admins)
- **QA/STAGE**: EEM Staging (preferred over EEM QA due to ongoing changes in QA environment)
- **PROD**: EEM Production

**What We Need from EEM**
- **Access & Credentials**: 
  - Bearer tokens via Azure AD for Staging and Production
  - **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - **CEDS JSON-LD Schema Definition**: Complete schema with all fields, nesting structure, examples
  - **Lead Administrator Specification**: 
    - How Lead Admin is represented in JSON-LD structure **[NEEDS INPUT: Field name? Nested object?]**
    - What identifying information is provided **[NEEDS INPUT: Email required? Unique ID or PIC? All three?]**
    - Can one person be Lead Admin for multiple entities? **[NEEDS INPUT: Confirmed support?]**
    - Can an entity have multiple Lead Admins? **[NEEDS INPUT: Or always singular?]**
  - **Hierarchy Representation**: How parent-child relationships are expressed in JSON-LD
  - **Entity Type Enumeration**: Complete list of entity types and their codes
  - REST API specification (OpenAPI/Swagger preferred)
  - Sample request/response payloads for all endpoints
  - Error response formats and codes
  - Rate limits (if any) for real-time endpoints
- **Test Data**: 
  - Does EEM Staging contain realistic entity hierarchy data with Lead Admin assignments?
  - Sample CEDS JSON-LD entity records for DEV mocks (various entity types)
  - Test scenarios: Multi-level hierarchy, inactive entities, entities without Lead Admin

**Key Integration Scenarios**

**Authorization Request Routing:**
- User submits authorization request for District 42
- Query EEM for District 42 entity details > retrieve Lead Administrator contact
- Send approval request to Lead Admin email/identity
- **[NEEDS INPUT: If Lead Admin field is empty, what's the fallback? Route to parent entity? Error?]**

**Entity Search (UI Typeahead):**
- User types "Lincoln" in entity search field
- Debounced API call to EEM search endpoint > returns matching entities
- Display: Entity Name, Entity Code, Entity Type, Status
- User selects > Entity Code used for authorization scope

**Transitive Authorization Resolution:**
- User has "Staffing Administrator" at ISD 50
- Permission check for Building 123 in District 10 in ISD 50
- Query EEM for Building 123 hierarchy > returns ancestor path: [ISD-50, District-10, Building-123]
- EntityHierarchyResolver confirms ISD 50 in ancestor path > AUTHORIZED
- **[NEEDS INPUT: Does EEM API return full ancestor path, or must we walk parent references?]**

**Lead Administrator Bootstrap:**
- User authenticates via MiLogin Business (email: admin@district42.org)
- Post-auth workflow queries EEM: "Which entities have admin@district42.org as Lead Admin?"
- EEM returns: District 42, Building 5, Building 6
- System auto-grants "Entity Lead Administrator" role at those scopes
- **[NEEDS INPUT: Dedicated endpoint for this query, or filter on entity search?]**

**Daily Entity Cache Refresh:**
- Scheduled job triggers overnight
- Query EEM for all active entities (full dataset or delta since last refresh?)
- Map EEM CEDS JSON-LD > MiEdWorkforce CEDS-aligned tables
- Update local entity cache for authorization resolution
- **[NEEDS INPUT: Full refresh or incremental updates supported?]**

**Implementation Notes**
- **Dependencies**: Phase 1 (MiLogin for Lead Admin identity matching, Mi-Key for potential future entity-person linkages)
- **Blocks**: Authorization approval routing, transitive authorization resolution, CTEIS/NexSys/TSDL (entity validation for staffing data)
- **Data Mapping**: 
  - EEM provides CEDS JSON-LD structure
  - MiEdWorkforce ETL maps to internal CEDS-aligned relational tables
  - **[NEEDS INPUT: Detailed field mapping document available? Who owns this mapping?]**
- **Caching Strategy**:
  - Entity hierarchy cache: 24-hour TTL (invalidate on EEM update events if available)
  - Entity detail cache: 5-minute TTL for high-frequency lookups
  - Lead Admin assignments: **[NEEDS INPUT: Cache duration? How often do these change?]**
- **Special Considerations**: 
  - Entity search must use debouncing (300ms) to reduce API load during typeahead
  - Authorization checks with transitive resolution are high-frequency (sub-10ms target)
  - Daily ETL provides resilience if EEM temporarily unavailable (24-48 hour cache tolerance)
  - **[NEEDS INPUT: Does EEM emit events when entity data changes (Lead Admin reassignment, hierarchy changes)? If yes, can we subscribe for cache invalidation?]**

**Outstanding Questions**
1. **Lead Administrator Representation**: How is Lead Admin identified in CEDS JSON-LD? Field name? Data structure? Email, PIC, etc.?
2. **Lead Admin Cardinality**: Can one entity have multiple Lead Admins? Can one person be Lead Admin for multiple entities?
3. **Lead Admin Fallback**: If entity has no Lead Admin assigned, how should authorization requests be routed?
4. **Hierarchy Query**: Does API return full ancestor path for an entity, or must we recursively query parents?
5. **Lead Admin Lookup Endpoint**: Is there a dedicated endpoint to query "which entities is person X the Lead Admin for"?
6. **Change Events**: Does EEM publish events for entity data changes (hierarchy, Lead Admin)? Can we subscribe for cache invalidation?
7. **Incremental Updates**: Does daily ETL support delta/incremental refresh, or always full dataset?
8. **Entity Types**: Where in the CEDS JSON-LD structure is entity type specified? Is there a complete list of entity types in CEDS JSON-LD (ISD, District, Building, Nonpublic, EPP, PLS confirmed - others?)?
9.  **Address Requirement**: Is physical address required for all entity types, or only certain types?
10. **API Credentials**: When needed? Lead time for Staging and Production tokens?
11. **Rate Limits**: Are there rate limits for real-time entity search/lookup endpoints?
12. **Field Mapping**: Is there existing documentation for EEM CEDS JSON-LD > MiEdWorkforce CEDS table mapping? Who owns maintaining this?
13. **API Design Patterns**: Is there documentation on things like pagination approach (limit/offset or cursor or page), any common response envelope structure, error response format and HTTP codes?

**Reference Documentation**
- **[NEEDS INPUT: Link to EEM CEDS JSON-LD schema documentation]**
- **[NEEDS INPUT: Link to EEM API specification (Swagger/OpenAPI)]**
- **[NEEDS INPUT: Link to entity type enumeration/code list]**
- IAM Domain Design: Authorization approval routing, transitive authorization resolution, Lead Admin bootstrap

---

## 4. Michigan State Police (MSP) - Criminal History Records Information Subscription Service (CHRISS) Rap Back

**System Identity**
- **Purpose**: Criminal conviction notifications for educator safety screening (Professional Practices Review)
- **System Owners**: Michigan State Police (MSP)
- **Service**: CHRISS (Criminal History Records Information Subscription Service)
- **Classification**: Bidirectional
- **Phase**: 2

**Integration Points**

### A. Incoming Notifications (MSP -> MiEdWorkforce)

- **Protocol**: REST API (HTTPS POST)
- **Endpoint**: MiEdWorkforce exposes **[NEEDS INPUT: Path? e.g., `/api/rapback/notifications`]**
- **Authentication**: API Token **[NEEDS INPUT: Who generates? Who validates?]**
- **Trigger**: MSP-initiated (daily)
- **Payload**:
  ```json
  {
    "agencyName": "LANSE CREUSE PUBLIC SCHOOLS",  // Required - Requesting Agency from fingerprint form
    "applicantName": "DOE, JOHN",                  // Required
    "requestID": "123456",                         // Required
    "sid": "1111111A",                             // Optional - State Identification Number
    "applicantDOB": "01/01/1980",                  // Optional
    "tcn": "AA00000000A00",                        // Optional - Transaction Control Number
    "applicantSSN": "123456789",                   // Optional
    "tcnDate": "03/19/2025",                       // Optional
    "pic": "",                                     // Optional (often empty) - May be Unique ID if available
    "judicialFlag": "Y",                           // Optional
    "rapResponse": ""                              // Always empty in notifications
  }
  ```
- **Frequency**: Daily
- **Volume**: ~200kB/month
- **Network**: MSP Test/Prod IPs must be whitelisted; requires SOM Telecom coordination

### B. Rap Sheet Retrieval (MiEdWorkforce -> MSP)

- **Protocol**: SOAP 1.2 (HTTPS POST)
- **Endpoint**: 
  - QA: `https://chrissqa.state.mi.us/CHRISS_SUBSCRIPTION_PORTAL/SubscriptionService.asmx`
  - Prod: `https://chriss.state.mi.us/CHRISS_SUBSCRIPTION_PORTAL/SubscriptionService.asmx`
- **Operation**: `GET_RAPBACK`
- **Authentication**: 
  - `Token_Info`: clientCode, passCode, applicationName
  - User credentials: MDE_USER_ID, MDE_PASSWORD **[NEEDS INPUT: Shared or individual?]**
- **Trigger**: MiEdWorkforce-initiated (on-demand when admin reviews Rap Back record or PPR case)
- **Request**: Person identifiers (pic, SSN, DOB, name) **[NEEDS INPUT: Which are required, optional? Are any flexible?]**
- **Response**: `rapResponse` field contains unformatted rap sheet text + metadata (sid, tcn, tcnDate, judicialFlag)
- **Frequency**: On-demand (reactive - triggered by admin reviewing existing notification)
- **Volume**: ~20MB/month

**Current Scope: Reactive Workflow Only**
- **Phase 2 Implementation**: Receive notifications from MSP, retrieve rap sheets on-demand when reviewing those notifications
- **NOT in Scope**: Proactive criminal history searches during credential application submission
- **Future Consideration**: See "Potential Future Enhancements" section below

**Environment Strategy**
- **DEV**: Both integrations mocked
- **QA/STAGE**: 
  - A: Expose QA endpoint, MSP Test sends notifications **[NEEDS INPUT: Firewall for QA?]**
  - B: Connect to MSP Test
- **PROD**: 
  - A: Expose Prod endpoint, MSP Prod sends notifications **[NEEDS INPUT: Firewall for Prod?]**
  - B: Connect to MSP Prod

**What We Need from MSP**
- **Network Access**: MSP Test/Prod server IPs for firewall whitelisting (both QA and Prod)
- **Authentication**: 
  - Integration A: Token setup process **[NEEDS INPUT: Direction?]**
  - Integration B: clientCode, passCode, applicationName, MDE_USER_ID, MDE_PASSWORD **[NEEDS INPUT: When? Lead time?]**
- **Documentation**:
  - Field definitions: sid, tcn, tcnDate, judicialFlag (values/meanings)
  - When is `pic` populated vs empty? **[NEEDS INPUT: Confirm if `pic` is the Unique ID or a different identifier]**
  - What specific events trigger notifications? (arrest, arraignment, conviction, all three?)
  - Error handling/retry logic for Integration A
  - Response format expected from MiEdWorkforce for Integration A **[NEEDS INPUT: Can response indicate if notification matched an active educator?]**
  - Confirmation of monitoring workflow (fingerprint -> school employment flag -> hit -> notification)

**Monitoring & Notification Workflow**

**How MSP Knows Who to Monitor:**
1. **Fingerprint Submission**: When potential school employees are hired, they submit fingerprints on a "school employment form" to MSP
2. **School Employment Flag**: MSP flags these individuals in their database with a "school employment" indicator
3. **Criminal Activity Hit**: When a flagged individual has a "hit" (arrest, arraignment, or conviction) on their stored fingerprint
4. **Notification Sent**: MSP sends the relevant information to CHRISS/Rap Back for MDE to review
5. **MDE Review**: MDE determines if the individual is still employed or has credentials requiring action

**Key Points:**
- **No MiEdWorkforce registration needed**: The monitoring flag is set during the fingerprint/background check process required for school employment
- **Automatic monitoring**: Once flagged by MSP during fingerprinting, individuals are monitored continuously
- **Notification triggers**: **[NEEDS INPUT: Confirm which events trigger notifications - arrest, arraignment, conviction, or all?]**
- **Requesting Agency**: The `agencyName` field in notifications reflects the school district from the original fingerprint submission
- **Irrelevant Notifications**: System will receive notifications for individuals who may no longer be active educators (e.g., left profession, moved states, etc.)

**Key Workflows**

**Daily Notification Receipt:**
1. MSP sends POST to MiEdWorkforce endpoint (daily batch of notifications)
2. MiEdWorkforce receives notification with metadata
3. **Person Matching**:
   - If `pic` (Unique ID?) present -> use directly **[NEEDS INPUT: Confirm `pic` is Unique ID]**
   - If `pic` empty -> query Mi-Key with SSN/DOB/Name to find Unique ID
   - Store match status: "Matched on SSN", "Matched on FN/LN/DOB", or "No Match"
4. **Employment Check** (for matched records):
   - Query employment records for Unique ID
   - Set `Is Employed` flag if active employment found
   - Store each distinct operational district and termination date (if exists)
5. **Credential Check** (for matched records):
   - Query for certificates (any status), permits, career authorizations, or special ed approvals from previous or current school year
   - Set `Has Credential` flag if found
   - Record each cert type and status
6. **MCL Code Matching**:
   - Match notification against MDE-defined Michigan Compiled Law (MCL) codes
   - Ignore parentheses, spaces, special characters; case-insensitive
   - Store MCL match record if exact match found
7. **Create Rap Back Record**: Store notification metadata (requestID, tcn, sid, judicialFlag, etc.)
8. **Log Irrelevant Notifications**: 
   - If "No Match" or `Is Employed = false` AND `Has Credential = false`, flag as "Not Relevant to MDE"
   - Store in queryable log for periodic review/reporting to MSP
9. **Alerts**: Notify Rapback Admin of new records requiring review (matched + employed/credentialed)

**Admin Reviews Rap Back Record:**
1. Admin searches/filters Rap Back records (by TCN, SID, Name, Employment/Credential status, etc.)
2. Admin clicks TCN hyperlink to view record detail
3. Admin clicks "View Rap Sheet"
4. MiEdWorkforce calls SOAP `GET_RAPBACK` with person identifiers (reactive - triggered by notification review)
5. MSP returns `rapResponse` (unformatted rap sheet text)
6. Display `rapResponse` to admin in monospace font (NOT stored in MiEdWorkforce)
7. Admin reviews record, adds notes, maintains alias names
8. **Admin Takes Action**:
   - **Notify School District**: Select district(s), send email, update status to "Notified School District"
   - **Refer to Professional Practice Review**: Create PPR work item, update status to "Referred to Professional Practices Review"
   - **No Referral Required**: Update status, optionally flag for MSP removal notification (outside MiEdWorkforce)
   - **Mark as Not Relevant**: Update status to indicate notification does not pertain to active educator (for MSP feedback)
9. **Audit Trail**: All actions stored with date/time, user, hyperlink to email/PPR item (except No Referral Required)

**Admin Reviews Irrelevant Notifications:**
1. Admin navigates to "Irrelevant Notifications" report
2. System displays notifications where `Is Employed = false` AND `Has Credential = false` OR `Match Status = No Match`
3. Admin can:
   - Confirm irrelevance (mark for MSP feedback list)
   - Re-attempt match (if SSN/DOB/Name slightly off)
   - Create manual PPR disclosure if actually relevant
4. Periodic export of "Not Relevant" list for transmission to MSP (manual process initially)

**Data Stored in MiEdWorkforce:**
- **Notification Metadata** (from Integration A):
  - Requesting Agency (agencyName)
  - Applicant Name, SID, DOB, SSN, TCN, TCN Date
  - Unique ID (if matched), Request ID, Judicial Flag, Create Date
- **Matching Results**:
  - Match status: "Matched on SSN", "Matched on FN/LN/DOB", "No Match"
  - Date/time of SSN matching, personnel matching, employment matching, credential check
  - `Is Employed` flag, employing districts, termination dates
  - `Has Credential` flag, credential types and statuses
  - MCL code matches
  - **Relevance Flag**: "Relevant to MDE", "Not Relevant - No Match", "Not Relevant - No Employment/Credential"
- **Admin Actions**:
  - Notes and notes history (user, date/time)
  - Alias names and history
  - Action(s) taken, status updates, emails sent, PPR items created
  - Full audit trail
- **NOT Stored**: Rap sheet content (`rapResponse` from Integration B) - generated on-demand only

**Implementation Notes**
- **Dependencies**: 
  - Mi-Key (for SSN/Name/DOB to Unique ID matching when `pic` empty)
  - EEM (for Lead Administrator contact info in District Notification File exports)
  - PPR (for creating Professional Practice Review work items)
  - Email/Communications (for district notifications)
  - Firewall coordination with SOM Telecom
- **Blocks**: PPR workflows, credential issuance decisions
- **Storage**: Store notification metadata only; NEVER store `rapResponse` content
- **Special Considerations**:
  - MiEdWorkforce exposes public endpoint (firewall required)
  - `pic` often empty in notifications (requires Mi-Key lookup)
  - SOAP 1.2 requires SOAP client library
  - Rap sheet displayed in monospace font (unformatted text)
  - MCL code matching: ignore parentheses, spaces, special characters; case-insensitive
  - Person matching: ignore hyphens, spaces, apostrophes, special characters in name/DOB matching
  - Multiple actions possible per record (bulk operations supported)
  - Export capabilities: SID Review File (Excel), District Notification File (Excel), Irrelevant Notifications Report (Excel)
  - **Irrelevant notifications will be received** - individuals flagged during school employment may have left profession, moved states, or never been Michigan educators

**Search & Export Capabilities**

**Search Filters:**
- TCN, SID, Offense Type, Segment
- First Name, Last Name
- Employment/Credential checks
- Relevance Status: All, Relevant, Not Relevant
- Action(s) Taken: Ready to Process, Notified School District, Referred to Professional Practice Review, Notified MSP for removal/No Action Required, Marked as Not Relevant

**Search Result Columns:**
- TCN (hyperlink), SID, Last Name, First Name, DOB
- Unique ID, Matched on SSN? (Y/N), Received Date
- Has Credential? (Y/N), Is Employed? (Y/N)
- Requesting Agency, School District, Action(s) Taken
- Relevance Status

**Export Files:**
- **SID Review File** (Ready to Process records):
  - Excel format
  - Fields: SID, Last Name, First Name, Unique ID, Date of Termination (if applicable), Entity Name (if applicable)
  - For `Is Employed` = true: one row per employing school district
  - Each unique SID/Entity Name combination listed once
- **District Notification File** (Ready to Process + Is Employed records):
  - Excel format
  - Sorted by employing school districts
  - Fields: School District, Lead Administrator First/Last Name, Title, Email, SID 1 through SID 20
  - Lead Administrator data from EEM database
- **Irrelevant Notifications Report** (Not Relevant records):
  - [NEEDS INPUT: Format TBD - CSV export? Application logs? UI-only view?]
  - Fields: TCN, SID, Last Name, First Name, DOB, SSN, Received Date, Match Status, Is Employed, Has Credential, Reason for Irrelevance
  - For periodic review and potential transmission to MSP to update monitoring flags

### Potential Future Enhancements

**1. Automated Notification Response (Integration A Enhancement)**
- **Current**: MiEdWorkforce receives notification, returns basic HTTP 200 OK acknowledgment
- **Future**: Response payload indicates whether notification matched an active educator
  ```json
  {
    "acknowledgment": "received",
    "requestID": "123456",
    "tcn": "AA00000000A00",
    "matchStatus": "Matched - Active Educator",  // or "No Match", "Matched - Inactive"
    "uniqueID": "MI123456789",                   // if matched
    "isEmployed": true,
    "hasCredential": true
  }
  ```
- **Benefit**: Enables MSP to automate removal of irrelevant monitoring flags
- **Requires**: Agreement from both MSP and MDE teams on response structure and workflow changes

**2. Proactive MDE-Initiated Notification Correction**
- **Current**: MDE exports list of irrelevant notifications periodically, manually sends to MSP
- **Future**: MiEdWorkforce proactively sends removal request to MSP when notification determined irrelevant
  ```json
  POST https://msp.state.mi.us/api/rapback/remove-monitoring
  {
    "sid": "1111111A",
    "reason": "Individual no longer employed in Michigan education",
    "requestedBy": "MDE",
    "requestDate": "2025-01-29"
  }
  ```
- **Benefit**: Real-time cleanup of monitoring flags, reduces noise in daily notification batches
- **Requires**: New MSP API endpoint, authentication mechanism, agreement on removal criteria

**3. Proactive Criminal History Check at Application Submission**
- **Current**: MiEdWorkforce only receives notifications from MSP; relies on MSP to send all relevant criminal activity
- **Future**: When educator applies for credential, MiEdWorkforce proactively calls CHRISS `GET_RAPBACK` to search for criminal history
  - Use case: Catch criminal activity that occurred before educator got "school employment flag" or gaps in notification system
  - Workflow: Credentialing -> PPR clearance check -> SOAP `GET_RAPBACK` (SSN/DOB/Name) -> Create disclosure if criminal history found -> Hold application
  - Consideration: May require different CHRISS API operation (search vs retrieval) or clarification that `GET_RAPBACK` supports both reactive and proactive use cases
- **Benefit**: Ensures no criminal history is missed; provides additional layer of safety screening
- **Requires**: 
  - Confirmation from MSP that proactive searches are permitted and supported by `GET_RAPBACK` operation
  - Clarification of required vs optional identifiers for search (SSN alone? SSN + DOB + Name?)
  - Potential volume/rate limit considerations
  - Business decision on when to trigger proactive check (every application? only renewals? high-risk positions?)
- **Status**: Under exploration; NOT in Phase 2 scope; requires follow-up with MSP to determine technical feasibility

**Outstanding Questions**
1. **Firewall**: MSP Test/Prod IP addresses? Coordination timeline with SOM Telecom?
2. **API Token** (Integration A): Who generates/validates?
3. **Response Format** (Integration A): What should MiEdWorkforce return when receiving notification? Can response indicate match status? (See Future Enhancement #1)
4. **Field Definitions**: sid, tcn, tcnDate, judicialFlag meanings and possible values?
5. **PIC Field**: When populated vs empty? **Is `pic` the Unique ID, or a different MSP-specific identifier?** Can we rely on it when populated?
6. **Credentials** (Integration B): When needed? Lead time? Shared or individual?
7. **Endpoint Path**: What path for MiEdWorkforce notification endpoint?
8. **GET_RAPBACK Required Fields**: Which person identifiers are required vs optional for reactive retrieval? Is PIC alone sufficient? SSN alone? SSN + DOB + Name?
9. **Notification Triggers**: What specific events trigger notifications - arrest, arraignment, conviction, or all?
10. **Retry Logic**: If MiEdWorkforce endpoint is down, does MSP retry? What's the retry strategy?
11. **Proactive Search Feasibility**: Can `GET_RAPBACK` be used proactively (search for criminal history without prior notification)? If so, what identifiers are required? Is there a different operation for proactive searches? (See Future Enhancement #3)
12. **Irrelevant Notification Handling**: What is MSP's preferred method for receiving feedback on irrelevant notifications? Manual list? Automated removal requests? (See Future Enhancements #1 and #2)

**Reference Documentation**
- WSDL: https://chrissqa.state.mi.us/CHRISS_SUBSCRIPTION_PORTAL/SubscriptionService.asmx
- Functional Design: MiEdWorkforce 30.0 Interface/Integration - Rapbacks (30.1-30.8)
- MCL Codes: Rapbacks-MCLKeywords.xlsx
- Process Flow: MORE-Rapbacks Process.vsdx
- Contacts: Fernando Montenegro, Sean Strom

---

## 5. Pearson EdReports (MTTC)

**System Identity**
- **Purpose**: Provides MTTC (Michigan Tests for Teacher Certification) assessment results required for educator credentialing
- **Classification**: Inbound
- **Phase**: 2

**Integration Points**
- **Method**: Manual file upload (human-driven process)
- **Workflow**: 
  1. Staff logs into Pearson EdReports
  2. Generates export file
  3. Downloads file
  4. Uploads file to MiEdWorkforce
- **File Format**: Fixed-width text file (119-character records including trailing filler)
- **Frequency**: Monthly **[NEEDS INPUT: Specific day of month? Upload deadline?]**

**File Layout Specification**

| Start | End | Width | Field | Description/Values |
|-------|-----|-------|-------|-------------------|
| 1 | 17 | 17 | Last Name | |
| 18 | 27 | 10 | First Name | |
| 28 | 28 | 1 | Middle Initial | |
| 29 | 29 | 1 | (filler) | |
| 30 | 38 | 9 | SSN | |
| 39 | 39 | 1 | (filler) | |
| 40 | 45 | 6 | Birthdate | Format: mmddyy |
| 46 | 46 | 1 | (filler) | |
| 47 | 47 | 1 | Sex | 1=Male, 2=Female, 0=No Response |
| 48 | 48 | 1 | Ethnic | 1=AI/AN, 2=Black, 3=Hispanic, 4=Asian/PI, 5=White, 6=Other, 0=No Response |
| 49 | 49 | 1 | Grade Level | A=Elementary, B=Secondary |
| 50 | 50 | 1 | Cert Status | A-F or Blank |
| 51 | 52 | 2 | College/University Currently Attending | 00-99, Zero Filled |
| 53 | 53 | 1 | Academic Designation | A=Academic Major, B=Academic Minor |
| 54 | 60 | 7 | (filler) | |
| 61 | 62 | 2 | State Code | Out of State examinees only |
| 63 | 64 | 2 | Report Institution 1 | MI Examinees only - Zero Filled |
| 65 | 66 | 2 | Report Institution 2 | MI Examinees only - Zero Filled |
| 67 | 68 | 2 | Report Institution 3 | MI Examinees only - Zero Filled |
| 69 | 71 | 3 | Test Code | Zero Filled, 001-096 |
| 72 | 72 | 1 | (filler) | |
| 73 | 78 | 6 | Test Date | Format: mmddyy |
| 79 | 79 | 1 | (filler) | |
| 80 | 82 | 3 | Pass/Fail Status | Basic Skills: RMW filled; Content Tests: 1st char filled, last 2 blank. P=Pass, F=Fail, N=Not Taken |
| 83 | 91 | 9 | Scaled Scores | Filled only for failed tests. Basic Skills: RMW filled (3 chars each); Content Tests: 1st score filled, last 2 blank |
| 92 | 92 | 1 | Holistic Score | Basic Skills only. Values: B, 0, 2-4, 6-8 |
| 93 | 93 | 1 | (filler) | |
| 94 | 107 | 14 | Subarea Performance Indicators | Values 1-4 (see legend below). Blank for subareas not used |
| 108 | 108 | 1 | (filler) | |
| 109 | 115 | 7 | Analytic/Zero Designations | Basic Skills Failing Examinees only, Blank otherwise |
| 116 | 118 | 3 | Cumulative Basic Skills Pass/Fail | Basic Skills only: RMW filled. Content Tests: blank. P=Pass, F=Fail, N=Not Taken |
| 119 | 119 | 1 | (filler) | |

**Subarea Performance Indicators Legend:**
- 1 = Scaled Subarea Score of 100-179
- 2 = Scaled Subarea Score of 180-219
- 3 = Scaled Subarea Score of 220-259
- 4 = Scaled Subarea Score of 260-300

**Sample File Records**
```
Firstname41 LastName41     J 100000092 041749 25AA21B         21    096 060107 PPP2932942206 444444444 
Firstname41 LastName41     M 100000092 052580 24AC99A         40    083 061307 P  214        143223 
Firstname41 LastName41     M 100000092 052580 25AA21B         10    002 060607 P  744        243223 
Firstname41 LastName41     M 100000092 052580 24AC99A         23    089 060607 P  289        067567 
Firstname41 LastName41     M 100000092 052580 24AC99A         82    022 060707 P  179        346384 
```

**Environment Strategy**
- **DEV**: Mocked - Sample fixed-width files with various test scenarios (passes, fails, Basic Skills vs Content Tests)
- **QA/STAGE**: Sample Pearson export files **[NEEDS INPUT: Can Pearson provide QA test files?]**
- **PROD**: Production Pearson export files

**What We Need from Pearson**
- **Access & Credentials**: Staff login credentials for Pearson EdReports portal **[NEEDS INPUT: When needed? Who needs access?]**
- **Documentation**: 
  - Test code mappings (001-096: which codes correspond to which MTTC tests?)
  - Export generation instructions (how to generate file in Pearson portal)
  - Expected file naming convention
  - Character encoding (ASCII? UTF-8?)
  - Certification Status codes (A-F definitions)
  - College/University code mappings (00-99)
- **Test Data**: Additional sample export files with edge cases (all fails, mixed results, multiple test types)

**File Import Workflow**

**Staff Uploads File:**
- Staff generates export from Pearson EdReports portal
- Downloads fixed-width file to local machine
- Logs into MiEdWorkforce
- Uploads file via file import interface
- MiEdWorkforce parses fixed-width format (119 characters per record)
- System validates:
  - Record length = 119 characters
  - SSN format (9 digits)
  - Birthdate format (mmddyy, valid date)
  - Test Date format (mmddyy, valid date)
  - Test Code (001-096 range)
  - Pass/Fail status (P/F/N only)
  - Sex code (0/1/2)
  - Ethnic code (0-6)
- Matches person using SSN + Birthdate + Name (query Mi-Key for Unique ID)
- Links test results to educator's credential application
- Displays import summary (records processed, errors, unmatched)

**Matching Logic:**
- Match on SSN + Birthdate + Name to find Unique ID via Mi-Key
- If no match or multiple matches: flag for manual resolution
- Store test results with Unique ID for credential processing

**Data Interpretation Notes**
- **Basic Skills Tests** (Reading/Math/Writing):
  - Pass/Fail Status: All 3 characters filled (positions 80-82)
  - Scaled Scores: All 3 scores filled if failed (positions 83-91, 3 chars each)
  - Holistic Score: Populated (position 92)
  - Cumulative Pass/Fail: All 3 characters filled (positions 116-118)
- **Content Tests** (Subject-specific):
  - Pass/Fail Status: Only 1st character filled (position 80), positions 81-82 blank
  - Scaled Scores: Only 1st score filled if failed (positions 83-85), positions 86-91 blank
  - Holistic Score: Blank
  - Cumulative Pass/Fail: Blank
- **Subarea Performance Indicators**: 
  - 14-character field supports up to 14 subareas per test
  - Blank positions indicate subarea not applicable for that test

**Implementation Notes**
- **Dependencies**: Mi-Key (for SSN to Unique ID matching)
- **Blocks**: Credential application processing (MTTC results required for certification)
- **Data Mapping**: Parse fixed-width format to internal credential test result structure **[NEEDS INPUT: CEDS mapping required?]**
- **Special Considerations**:
  - Manual process: No API available from Pearson; staff-driven upload
  - SSN matching: Currently uses SSN for matching; transitioning to Unique ID as primary identifier
  - Manual resolution: Unmatched records require staff review and manual linking
  - Fixed-width format: Requires precise parsing (exact character positions 1-119)
  - Multiple test types: Logic must differentiate Basic Skills vs Content Tests
  - Multiple records per person: Same SSN may appear on multiple lines (different tests, different dates)
  - Monthly cadence: Allows time for manual intervention if import issues occur
  - File storage: **[NEEDS INPUT: Retain uploaded files? For how long? Audit purposes?]**
  - Zero-filled fields: Numeric fields use zero-padding (e.g., "096" not "96")
  - Date format: All dates use mmddyy format (6 characters)

**Outstanding Questions**
1. **Upload Timing**: Specific day of month for file upload? Deadline for processing?
2. **Pearson Access**: When do staff need Pearson portal credentials? Who requires access?
3. **Test Code Mappings**: Complete list of Test Codes (001-096) and their MTTC test names?
4. **File Naming**: Expected naming convention for uploaded files?
5. **Character Encoding**: ASCII or UTF-8?
6. **QA Test Files**: Can Pearson provide sample export files for QA testing?
7. **CEDS Mapping**: Is CEDS mapping required for test results?
8. **File Retention**: Should uploaded files be retained? For how long?
9. **Error Handling**: What happens if file is malformed or contains invalid Test Codes?
10. **Unmatched Records**: What's the process for resolving unmatched SSNs?
11. **Certification Status Codes**: What do codes A-F represent?
12. **College/University Codes**: Mapping of codes 00-99 to institutions?
13. **Multiple Tests**: How to handle multiple test results for same person on same upload?
14. **Report Institutions**: What are Report Institution codes 1-3 used for?

**Reference Documentation**
- Fixed-width file format specification (defined above)
- **[NEEDS INPUT: Link to Pearson EdReports portal documentation]**
- **[NEEDS INPUT: Technical contact at Pearson]**

---

## 6. NASDTEC Clearinghouse (Ed ID Clearinghouse)

**System Identity**
- **Purpose**: National database check for disciplinary actions against educators; ensures Michigan only credentials individuals in good standing (license/certificate annulled, denied, suspended, revoked, or invalidated)
- **Official Name**: Ed ID Clearinghouse by National Association of State Directors of Teacher Education and Certification (NASDTEC)
- **Common Name**: NASDTEC Clearinghouse
- **Classification**: Inbound
- **Phase**: 2

**Integration Points**
- **Base URL**: https://www.nasdtec.org/api
- **Protocol**: REST API (HTTPS GET)
- **API Endpoint**: `/v1/clearinghouse/people` (returns ALL flagged educators, not individual searches)
- **Authentication**: API Key (URL parameter: `apiKey=YOUR_KEY`)
- **Response Formats**: JSON (default), XML, CSV, Fixed (legacy - not recommended)
- **Parameters**:
  - `format`: json, xml, csv, fixed (default: json)
  - `startdate`: YYYYMMDD, "lastmonth", "currmonth" (optional - filters by TransactionDate)
  - `enddate`: YYYYMMDD, "lastmonth", "currmonth" (optional - filters by TransactionDate)
- **Trigger**: MiEdWorkforce-initiated (nightly automated pull to refresh local database)
- **Data Exchanged**:
  - Person: LastName, FirstName, MiddleName, SuffixName, BirthDate (YYYYMMDD)
  - Certification: CertificationID (SSN, Canadian Insurance ID, or other)
  - Action: Jurisdiction (2-letter state/province), TransactionDate (YYYYMMDD)
  - Reference: ClearinghouseId, ClearinghouseUrl (link to full details on NASDTEC site)
- **Frequency**: Nightly batch pull
- **Data Volume**: ~110kB per month

**Environment Strategy**
- **DEV**: Mocked - Sample JSON responses (clear, flagged, multiple flags)
- **QA/STAGE**: NASDTEC Testing environment **[NEEDS INPUT: Test API key available?]**
- **PROD**: NASDTEC Production (https://www.nasdtec.org/api)

**What We Need from NASDTEC**
- **Access & Credentials**: 
  - API Key for Testing and Production (contact: support@nasdtec.org)
  - Provided by MDE at implementation time **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - API specification (provided: NASDTEC_API_Documentation_v1.pdf)
  - Disciplinary action types represented in Clearinghouse
  - CertificationID interpretation (SSN vs other identifiers)
  - Rate limits or usage quotas
- **Test Data**: 
  - Test records with various disciplinary statuses
  - Edge cases (multiple jurisdictions, name variations)

**Key Workflows**

**Nightly Pull and Local Database Refresh:**
- Scheduled job calls `/v1/clearinghouse/people?format=json&startdate=currmonth&apiKey=KEY`
- Returns ALL flagged educators with TransactionDate in current month (bulk list, not individual search)
- MiEdWorkforce stores records locally:
  - LastName, FirstName, MiddleName, SuffixName, BirthDate
  - CertificationID (SSN or other identifier)
  - Jurisdiction, TransactionDate
  - ClearinghouseId, ClearinghouseUrl
- Match to Michigan educators:
  - Query Mi-Key using CertificationID (SSN) + BirthDate + Name for Unique ID
  - If match found, flag educator's record in MiEdWorkforce
  - Create manual review task for credential processor
- **[NEEDS INPUT: Incremental strategy - pull currmonth nightly, or pull lastmonth at month-end?]**

**Credential Processor Searches Local Database:**
- Processor reviews credential application
- System searches LOCAL MiEdWorkforce database (not live API call) for educator's name/SSN/DOB
- Displays any matching Clearinghouse records stored locally
- Processor clicks ClearinghouseUrl to view full details on NASDTEC site
- Manual decision: approve, deny, or request more information

**Implementation Notes**
- **Dependencies**: Mi-Key (for CertificationID/SSN to Unique ID matching)
- **Blocks**: Credential application approval (flags require manual review before issuance)
- **Data Mapping**: JSON response fields to internal disciplinary action table **[NEEDS INPUT: CEDS mapping required?]**
- **Special Considerations**:
  - New interface (didn't exist in legacy system)
  - API returns bulk list, NOT individual person search capability
  - Local database approach: Store Clearinghouse records in MiEdWorkforce for search/reference
  - Nightly pull keeps local database current without excessive API calls
  - ClearinghouseUrl provides link to full details (for manual review)
  - CertificationID may be SSN or other identifier (handle both)
  - TLS encryption required (handled by HTTPS)
  - API key specific to Michigan (do not share across organizations)
  - **[NEEDS INPUT: Check timing decision - search local database during application submission, review, or final approval?]**
  - **[NEEDS INPUT: Handling flags - manual review (current assumption) or auto-block?]**

**Outstanding Questions**
1. **API Key**: When can we get Testing and Production keys? Lead time from NASDTEC?
2. **Pull Strategy**: Pull `currmonth` nightly (incremental), or pull `lastmonth` at month-end (complete)?
3. **Historical Data**: Do we need to pull historical data at launch (e.g., `startdate=20200101`)? Or start fresh from go-live?
4. **Check Timing**: When to search local database - at application submission, during review, or final approval?
5. **Flag Handling**: Manual review (assumed) or auto-block applications with flags?
6. **CEDS Mapping**: Is CEDS mapping required for disciplinary records stored locally?
7. **Test API Key**: Is NASDTEC Testing environment available with test API key?
8. **Rate Limits**: Are there API rate limits or usage quotas?
9. **CertificationID Handling**: How to handle non-SSN CertificationIDs (Canadian Insurance ID, etc.)?
10. **Match Logic**: If multiple potential matches found via Mi-Key, how to resolve?
11. **Data Retention**: How long to retain Clearinghouse records in local database? Indefinitely? 7 years?
12. **Date Filter Behavior**: Does startdate/enddate filter by TransactionDate (when action occurred) or by when the record was added to the Clearinghouse database? This impacts our pull strategy.

**Reference Documentation**
- API Documentation: NASDTEC_API_Documentation_v1.pdf (can be found in PMO SharePoint, unclear if this is official documentation from NASDTEC or notes taken by a previous integrator)
- Base URL: https://www.nasdtec.org/api
- Support Contact: support@nasdtec.org, jimmy.adams@nasdtec.org 

---

## 7. STARR (Student Transcript and Academic Record Repository)

**System Identity**
- **Purpose**: Provides individual post-secondary awards and course data to pre-populate credential applications and verify professional learning
- **Classification**: Inbound
- **Phase**: 2

**Integration Points**
- **Protocol**: REST/Parquet
- **API Endpoints**: **[NEEDS INPUT: Specific endpoint names/paths]**
  - **[Query endpoint by Unique ID or SSN for person's transcripts]**
  - **[Retrieve specific award/degree details]** (if separate)
  - **[Course history endpoint]** (if separate)
- **Trigger**: MiEdWorkforce-initiated (query during credential application)
- **Data Exchanged**: Unique ID, Institution, Degree/Award Type, Major, Completion Date, Course History (for professional learning), **[NEEDS INPUT: GPA? Credits? Other fields?]**
- **Frequency**: Currently annual collection; **[NEEDS INPUT: Planning for real-time queries or more frequent batch updates?]**

**Environment Strategy**
- **DEV**: Mocked - Sample post-secondary transcripts for various degree types
- **QA**: **[NEEDS INPUT: STARR Staging or Production?]**
- **PROD**: STARR Production

**What We Need from STARR**
- **Access & Credentials**: 
  - **[NEEDS INPUT: Authentication method?]**
  - **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - API specification or file format spec
  - Award/degree type codes and mappings
  - Institution identifiers (IPEDS codes?)
  - Course code standards
  - Error handling
- **Test Data**: 
  - Sample transcripts for various scenarios (Bachelor's, Master's, multiple degrees, incomplete programs)
  - Professional learning course examples

**Implementation Notes**
- **Dependencies**: Phase 1 (Mi-Key) - Unique ID for person matching
- **Blocks**: None (enhances credential applications but not blocking)
- **Data Mapping**: **[NEEDS INPUT: STARR data already CEDS-compliant or needs mapping?]**
- **Special Considerations**: Currently annual collection; planning for future increased frequency. Non-blocking - if unavailable, applicants manually enter education history.

---

## 8. Michigan Student Data System (MSDS) - Teacher Student Data Link (TSDL)

**System Identity**
- **Purpose**: Connects student courses to teacher records using Unique IDs; validates active employment and course alignment
- **Classification**: Bidirectional
- **Phase**: 3

**Integration Points**
- **API Endpoints**: **[NEEDS INPUT: Specific endpoint names/paths]**
  - **[MiEdWorkforce validates teacher Unique ID active status]**
  - **[TSDL queries teacher course assignments from MiEdWorkforce]**
  - **[Student-teacher linkage validation endpoint]**
- **Trigger**: Bidirectional (MiEdWorkforce validates teacher Unique IDs; TSDL queries course assignments)
- **Data Exchanged**: Teacher Unique ID, Student Unique ID, Course ID, Course Section, School Year, Term, **[NEEDS INPUT: Other fields?]**
- **Frequency**: Daily ("Daily+")

**Environment Strategy**
- **DEV**: Mocked - Sample student-teacher linkage data
- **QA**: **[NEEDS INPUT: TSDL/MSDS Staging, QA, or Production?]**
- **PROD**: MSDS/TSDL Production

**What We Need from TSDL**
- **Access & Credentials**: 
  - **[NEEDS INPUT: Authentication method?]**
  - **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - API specification for bidirectional endpoints
  - Course code standards and mappings
  - Student-teacher linkage rules
  - Data validation requirements
  - Error codes and handling
- **Test Data**: 
  - Sample teacher records with various course assignments
  - Student-teacher linkages for validation testing

**Implementation Notes**
- **Dependencies**: Phase 1 (Mi-Key) - Unique ID required; Phase 3 (EEM) - entity validation
- **Blocks**: None directly, but critical for data quality and teacher validation
- **Data Mapping**: **[NEEDS INPUT: CEDS alignment needed?]**
- **Special Considerations**: High volume due to daily sync of intensive student-teacher records. Supports transition away from legacy PIC reliance.

---

## 9. CEPAS (Central Electronic Payment Authorized System)

**System Identity**
- **Purpose**: State's centralized electronic payment system for credit card and e-check transactions; handles credential application payments
- **Official Name**: Centralized Electronic Payment Authorization System (CEPAS)
- **Organization**: State of Michigan - Treasury
- **Classification**: Bidirectional
- **Phase**: 3

**Integration Points**

### A. Payment Initiation (User Payment Flow)

- **Protocol**: HTTP Redirect (HTTPS GET)
- **Direction**: MiEdWorkforce redirects TO CEPAS payment site
- **Trigger**: User-initiated (credential application payment)
- **Data Exchanged**:
  - **Outbound (encrypted with AES GCM)**:
    - `amount`: Payment amount
    - `ref`: Reference data (for matching in posting file)
    - `id`: Internal identifier (e.g., session ID)
    - `returnurl`: MiEdWorkforce callback URL for payment results
  - **Encrypted string format**: `aen=<key id>|<12B nonce><encrypted string>`
  - **Encryption**: AES GCM with 12-byte nonce, 16-byte tag, using CEPAS-provided key
  - **Inbound (query string to returnurl)**:
    - `c`: EpayReturnCode
    - `m`: EpayResultMessage
    - `o`: ConfirmationNumber
    - `t`: TotalAmount
    - `d`: SettlementSubmissionDate
    - `z`: AuthorizationCode
    - `i`: Original ID
    - `ct`: Card Type (VISA, MC, AMEX, DISC, STAR, Pulse, NYCE - not sent for eCheck)
    - `hash`: SHA-1 hash for validation (`SHA-1([ConfirmationNumber][Amount][SecurityKey])`)
- **Frequency**: Real-time (per payment)
- **Data Volume**: ~1kB per payment request

### B. Refunds/Reimbursements (Admin-Initiated)

- **Protocol**: REST API (HTTPS POST)
- **Direction**: MiEdWorkforce calls CEPAS API
- **Trigger**: Admin-initiated (manual refund request)
- **Authentication**: API Key (SecurityKey) + ApplicationID
- **Request Headers**:
  - `ApplicationID`: PayPoint account identifier
  - `PaymentChannel`: 1 (Web)
  - `SecurityKey`: API authentication token
- **Request Body** (CancelPayment operation):
  - `Header`: See authentication above
  - `ConfirmationNumber`: Original payment confirmation number
  - `RefundAmount`: Amount to refund (less than or equal to original)
- **Frequency**: As needed (daily)
- **Data Volume**: ~1kB per refund request

### C. Nightly Reconciliation (Posting File)

- **Protocol**: SFTP
- **Direction**: CEPAS posts file; MiEdWorkforce retrieves
- **Trigger**: Automated nightly (file posted by CEPAS)
- **File Name**: `<CEPAS application name>_YYMMDD.txt`
- **File Format**: Fixed-width text file
  - **Application Header (AH)**: Record Type, Site, Agency, Application (41 characters)
  - **Application Detail (AD)**: 776 characters per transaction record
    - Payment Method, Payment ID, Transaction Date, Payment/Command Codes
    - Payment Amount, Confirmation Number, Card Type, Account Number (last 4)
    - Payer info: Name, Email, Address
    - Custom Reference Data (254 characters)
  - **Application Footer (AF)**: Record Type, Site, Agency, Application, Total Records, Total Amount (67 characters)
- **Frequency**: Daily (nightly batch)
- **Data Volume**: <300kB per posting file
- **Purpose**: Reconcile previous day's transactions; match internal records to confirmed payments

**Environment Strategy**
- **DEV**: Mocked - Simulated payment redirect, refund API responses, sample posting files
- **QA/STAGE**: CEPAS Test/Sandbox environment **[NEEDS INPUT: Confirm sandbox environment available?]**
- **PROD**: CEPAS Production

**What We Need from CEPAS**
- **Access & Credentials**: 
  - **Payment Initiation**: AES GCM encryption key (ID + Key), SecurityKey (for hash validation), ApplicationID
  - **Refunds**: API SecurityKey, ApplicationID, PaymentChannel assignment
  - **Posting File**: SFTP credentials via File Transfer Service (FTS) **[NEEDS INPUT: FTS upgrade status? Timeline?]**
  - **[NEEDS INPUT: When needed per environment? Lead time?]**
- **Documentation**: 
  - Integration guide (referenced: Centralized Electronic Payment Authorization System (CEPAS) Resources)
  - Payment redirect URL structure and parameters
  - AES GCM encryption implementation details
  - Hash validation procedure (SHA-1 calculation)
  - Refund API specification (CancelPayment endpoint)
  - Posting file field definitions (provided above - confirm current)
  - Error codes and handling (EpayReturnCode values)
  - Card type codes
- **Test Data**: 
  - Test credit card numbers (success, decline, insufficient funds scenarios)
  - Sample posting files with various transaction types
  - Test refund scenarios

**Key Workflows**

**User Completes Payment:**
- User initiates credential application payment
- MiEdWorkforce encrypts payload (amount, ref, id, returnurl) using AES GCM with CEPAS key
- Redirects user to CEPAS payment site with encrypted string
- User enters payment details on CEPAS site (MiEdWorkforce never sees card numbers)
- CEPAS processes payment, redirects to MiEdWorkforce returnurl with results
- MiEdWorkforce validates hash (`SHA-1([ConfirmationNumber][Amount][SecurityKey])`)
- Stores ConfirmationNumber, Amount, Payment Status
- Updates credential application status

**Admin Initiates Refund:**
- Admin selects paid transaction for refund
- MiEdWorkforce calls CEPAS CancelPayment API
- Sends: Header (ApplicationID, PaymentChannel, SecurityKey), ConfirmationNumber, RefundAmount
- CEPAS processes refund, returns confirmation
- MiEdWorkforce updates transaction status

**Nightly Reconciliation:**
- CEPAS posts daily file to FTS SFTP mailbox
- Scheduled job retrieves file (`<app name>_YYMMDD.txt`)
- Parses fixed-width format (AH/AD/AF records)
- Matches AD records to internal transactions using Custom Reference Data field
- Updates pending payments to "confirmed" status
- Flags discrepancies for manual review

**Implementation Notes**
- **Dependencies**: FTS (File Transfer Service) upgrade must complete before Posting File integration **[NEEDS INPUT: FTS upgrade timeline?]**
- **Blocks**: Paid credential processing (no payment = no processing)
- **Data Mapping**: None (financial transaction data stored as-is)
- **Special Considerations**:
  - **PCI compliance**: MiEdWorkforce never handles credit card numbers; CEPAS handles all PCI-sensitive data
  - **Encryption**: AES GCM with CEPAS-provided key for payment redirect; TLS for all HTTPS communication
  - **Hash validation**: Critical security check for payment results using SHA-1 hash
  - **FTS upgrade**: Posting file integration delayed; current documentation may be outdated post-upgrade
  - **Custom Reference Data**: Use to match posting file records to internal transactions (254 character limit)
  - **Refund timing**: Refunds processed via API, not redirect flow
  - **Nightly batch**: Posting file provides confirmation of completed transactions for audit/reconciliation

**Outstanding Questions**
1. **CEPAS Sandbox**: Is Test/Sandbox environment available for QA/STAGE? Same credentials as Prod or separate?
2. **Credentials Timeline**: When can we get encryption keys, SecurityKey, ApplicationID for each environment? Lead time?
3. **FTS Upgrade**: What is timeline for File Transfer Service upgrade? Will posting file format change?
4. **SFTP Access**: When will SFTP credentials be available post-FTS upgrade?
5. **Refund API Endpoint**: What is the full URL for CancelPayment API?
6. **EpayReturnCode**: What are the possible values and their meanings?
7. **Card Type Handling**: Should we display card type to user? Store for reporting?
8. **Posting File Timing**: What time does posting file become available nightly?
9. **Discrepancy Handling**: What's the process for resolving posting file mismatches?
10. **Test Cards**: What test card numbers should we use in QA/STAGE for various scenarios?

**Reference Documentation**
- Integration Guide: Centralized Electronic Payment Authorization System (CEPAS) Resources
- Interface Docs: CEPAS Payments, CEPAS Posting File, CEPAS Reimbursements
- File Transfer Service: **[NEEDS INPUT: FTS documentation link]**
- Technical Contacts: Sean Strom (stroms@michigan.gov), Simon Wang (WangS@michigan.gov)
- Business Contact: Amy Kelso (kelsoa@michigan.gov)

---

## 10. CEPI Data Infrastructure

**System Identity**
- **Purpose**: Centralized data platform for cross-system analytics, longitudinal reporting, and historical data preservation; enables standardized data sharing across Michigan education systems
- **Classification**: Bidirectional
- **Phase**: 3

**System Components**

### A. CEDS Transactional Layer (Primary Integration Point)
- **Owner**: CEPI
- **Technology**: Cosmos DB
- **Purpose**: Centralized foundational data repository for cross-system consumption
- **Data Format**: CEDS JSON-LD documents
- **Status**: Active/Strategic (Future direction)

### B. CEPI CEDS Data Warehouse (Strategic Platform)
- **Owner**: CEPI
- **Technology**: Parquet files (Data Lake/Lakehouse architecture)
- **Purpose**: Modern analytics and longitudinal reporting data warehouse
- **Data Sources**: Fed by CEDS Transactional Layer
- **Status**: Active/Strategic (Future direction)

### C. MSLDS (Legacy/Transitional)
- **Owner**: CEPI
- **Technology**: Relational Database (SQL Server)
- **Purpose**: Legacy data warehouse for longitudinal student and workforce data
- **Data Scope**: Contains older historical data from MOECS/REP via Analysis Server
- **Status**: Legacy/Transitional - Expected to be replaced by Parquet-based warehouse

**Integration Points**

MiEdWorkforce publishes operational workforce data as CEDS-aligned events/snapshots to the **CEDS Transactional Layer**, which serves as the primary integration point for cross-system analytics. The Transactional Layer automatically feeds CEPI's downstream analytical systems, including the **CEPI CEDS Data Warehouse** (Parquet-based, modern) for longitudinal reporting.

During the transition period, MiEdWorkforce may also query the legacy **MSLDS** warehouse for older historical data until it is fully migrated to the CEPI CEDS DW. MiEdWorkforce does not write directly to Parquet files or manage ETL into CEPI warehouses.

**Publishing to CEDS Transactional Layer (Outbound):**
- **Trigger**: MiEdWorkforce-initiated (event-driven or batch)
- **Data Exchanged**: Credentials (issued, renewed, revoked), Employment (hired, assignments, terminations), Professional Learning (SCECH credits), Authorization changes
- **Frequency**: Real-time events, micro-batches, or daily snapshots
- **Method**: **[NEEDS INPUT: REST API? Message queue? Event Grid? Publishing mechanism TBD]**

**Querying Historical Data (Inbound - Transitional):**
- **Source**: MSLDS (during transition), then CEPI CEDS DW (post-transition)
- **Trigger**: MiEdWorkforce-initiated (on-demand when displaying historical context in UI)
- **Data Exchanged**: Historical credentials (from MOECS), historical staffing (from REP), historical professional learning
- **Frequency**: On-demand (as users view historical records)
- **Method**: **[NEEDS INPUT: Direct database query via Synapse? REST API? Access pattern TBD]**
- **Transition Timeline**: **[NEEDS INPUT: When will MSLDS data be fully migrated to CEPI CEDS DW?]**

**Environment Strategy**
- **DEV**: Mock endpoints/data stores for both publishing and querying
- **QA/STAGE**: CEDS Transactional Layer Staging (publish events); MSLDS QA/Staging (query historical data)
- **PROD**: CEDS Transactional Layer Production; MSLDS Production (transitional), then CEPI CEDS DW Production

**What We Need from CEPI**
- **Access & Credentials**: 
  - Credentials for publishing to CEDS Transactional Layer (per environment)
  - Access to MSLDS for historical queries (during transition)
  - **[NEEDS INPUT: Authentication methods? When needed? Lead time?]**
- **Documentation**: 
  - CEDS JSON-LD schema definitions for events MiEdWorkforce will publish
  - Publishing mechanism specification (API, message queue, etc.)
  - MSLDS schema/query access documentation
  - Historical data availability and date ranges in MSLDS
  - Transition timeline from MSLDS to CEPI CEDS DW
- **Test Data**: 
  - Sample CEDS JSON-LD event structures
  - Sample historical records in MSLDS QA/Staging for testing queries

**Implementation Notes**
- **Dependencies**: 
  - Phase 1 (Mi-Key): Unique ID required for person-level events
  - Phase 3 (EEM): Entity Code required for employment events
  - SLDS/CEDS grant team: Historical MOECS/REP data migration to MSLDS (external to MiEdWorkforce)
- **Blocks**: Cross-system analytics (without CEDS Transactional Layer publishing); Historical reporting (without MSLDS/CEPI CEDS DW access)
- **Data Mapping**: MiEdWorkforce transforms internal CEDS-aligned data to CEDS JSON-LD for publishing
- **Special Considerations**:
  - **Bounded Context**: MiEdWorkforce owns operational data; publishes events/snapshots to CEDS Transactional Layer asynchronously; never writes directly to external data stores
  - **Anti-Corruption Layer**: Abstract CEDS Transactional Layer integration behind internal interfaces to protect from external schema changes
  - **Data Responsibilities**: CEPI DW team migrates educational data (credentials, staffing) from MOECS/REP to MSLDS/CEPI CEDS DW; MiEdWorkforce migrates only workflow/audit data (application submissions, approval chains, payment records)
  - **Eventual Consistency**: Operational writes to MiEdWorkforce database are synchronous; events published to CEDS Transactional Layer are asynchronous
  - **Graceful Degradation**: If MSLDS/CEPI CEDS DW unavailable, display "Historical data temporarily unavailable" but allow operations to continue

**Outstanding Questions**
1. **Publishing Mechanism**: REST API? Message queue (Service Bus/Event Grid)? What is the preferred/planned integration method?
2. **CEDS JSON-LD Schemas**: Where are authoritative schema definitions documented? Who maintains them?
3. **Event Types**: Complete list of event types MiEdWorkforce should publish?
4. **Historical Query Method**: Direct database access via Synapse? Some other pattern? What access will be provided?
5. **MSLDS Schema**: Where is MSLDS schema documented for historical queries?
6. **MSLDS Transition**: What is the timeline for migrating MSLDS data to CEPI CEDS DW? When can MiEdWorkforce switch from querying MSLDS to CEPI CEDS DW?
7. **Authentication**: What authentication methods for CEDS Transactional Layer publishing and MSLDS querying? When can credentials be provisioned?
8. **Historical Data Scope**: What MOECS/REP data currently exists in MSLDS? Date ranges? Known gaps?
9. **Division of Labor**: Confirm CEPI DW team owns educational data migration; MiEdWorkforce owns workflow/audit data migration only?

**Reference Documentation**
- **[NEEDS INPUT: Link to CEDS Transactional Layer documentation/specifications]**
- **[NEEDS INPUT: Link to CEDS JSON-LD schema definitions]**
- **[NEEDS INPUT: Link to MSLDS schema documentation]**
- **[NEEDS INPUT: CEPI technical contacts for each system component]**
- MiEdWorkforce Data Systems Architecture document (this repository)

---

## 11. Michigan DataHub (MiDH)

**System Identity**
- **Purpose**: Centralized hub through which districts' SIS/HRMS vendors exchange staffing, assignment, professional learning, and identity data with MiEdWorkforce, translating between the Ed-Fi standard (MiDH-side) and CEDS (MiEdWorkforce-side). Covers both directions: MiDH submitting district data into MiEdWorkforce, and MiDH requesting MiEdWorkforce data back out to populate district systems.
- **Classification**: Bidirectional — see the two distinct mechanisms below; the client's own integration tracker (`MORE Integrations.md`) also classifies this as Bidirectional, not purely inbound
- **Phase**: 3

**A Note on Unified Data Platform (UDP) Overlap**

The state is separately pursuing a "Unified Data Platform" (UDP) initiative aimed at centralizing core state reference data — a single API/data layer for things like organization/entity metadata (districts, schools) that are static, have few update channels, and are consumed widely across many state systems. This is a distinct effort from MiDataHub, but it touches this integration because **some of the data these 20 use cases move may be a better fit for UDP than for MiEdWorkforce specifically** — particularly static code/reference vocabularies (CIP Codes, Career Cluster) versus MiEdWorkforce's own operational, per-educator records (employment, assignment, evaluation). Below, each use case is flagged with a **UDP Overlap** note. This is an open architectural question, not a resolved scope decision — flagged here so it surfaces early in scoping discussions with MiDataHub's team rather than being decided by default. Note also that nearly every use case below takes a District/Building Identifier as an input, which today resolves through EEM (Integration 3) — if UDP absorbs or wraps EEM's role as the organization/entity source of truth, that's a single structural change affecting all 20 use cases' inputs, not just the ones flagged individually below.

**Integration Points**
- **Protocol**: REST API (HTTPS POST/GET)
- **Two distinct mechanisms are documented in the source IDD** (`8.4 - Data Hub IDD.docx`), and should not be conflated:
  1. **CEPI Public Services `SendToHub`/`ReceiveFromHub`** — a generic batch JSON envelope (`BatchId`, `FeedSystem`, `DistrictId`, `DataType`, `JsonData`). `SendToHub` (MiEdWorkforce → CEPI → MiDH) exists today and is gated by per-district opt-in. `ReceiveFromHub` (MiDH → CEPI → MiEdWorkforce) does **not** exist yet — blocked on CEPI's own build, not something either MiDataHub or MiEdWorkforce controls unilaterally.
  2. **Ed-Fi Identities API** — MiDataHub-initiated calls directly against MiEdWorkforce services, covering both `Submit` (MiDataHub pushes district data in) and `Request` (MiDataHub pulls MiEdWorkforce data out to populate district systems) directions. This is the mechanism behind all 20 use cases below.
- **Direction**: MiDH/CEPI always initiates; MiEdWorkforce never calls out to MiDH's Ed-Fi ODS APIs directly (confirmed as the consensus direction — "Passive for MiEdWorkforce" — in the January 26, 2026 stakeholder meeting)
- **Trigger**: District-initiated via MiDH (SIS/HRMS vendor updates sync to MiEdWorkforce)
- **Frequency**: **[NEEDS INPUT: Real-time? Daily batch? District-driven?]** — individual-record vs. batch volume split is also still an open question as of the January 26, 2026 meeting

**API Endpoints / Use Cases** (from FDD `8.4 - Data Hub IDD`, 20 use cases; # matches the IDD's own numbering)

| # | Service | Action | Business Spec | UDP Overlap Note |
|---|---|---|---|---|
| 1 | Staff Identity Lookup (Unique ID) | Request | 24.12 | No — identity resolution is Mi-Key/MiEdWorkforce operational matching, not static reference data |
| 2 | Staff Employment Record | Submit | 21.3 / 21.9 | No — core MiEdWorkforce system-of-record data |
| 3 | Staff Assignment Record | Submit | 22.3 / 22.14 | No — core MiEdWorkforce system-of-record data |
| 4 | Staff Course Level Information | Submit | 22.16a | No for the assignment record itself; the underlying SCED/Course Code System code lists are plausibly shared reference vocabulary |
| 5 | Staff Special Education Information | Submit | 22.16b | No — operational assignment attribute |
| 6 | Staff Career Technical Education Information | Submit | 22.16c | **Yes, partially** — Career Cluster and CIP Code are state/national standardized reference vocabulary (the IDD itself notes CTEIS as a possible future source, and CIP-code/endorsement mappings are still pending from CEPI as of the August 2026 meeting); the staff-to-CTE-course assignment stays MiEdWorkforce's, but the code values themselves are a strong UDP candidate |
| 7 | Staff Specialized Funding Stream Information | Submit | 22.17 | Possibly, for the Migrant/Title I program category code lists; the staff/position funding assignment itself is operational |
| 8 | Staff Evaluation Information | Submit | 22.26 | No — MiEdWorkforce-specific operational data |
| 9 | Staff Basic Information ("Snack Pack") | Request | — | **Mixed, needs explicit scoping** — if UDP ends up hosting a canonical issued-credential reference feed, the Certifications portion of this response could eventually be UDP-sourced; PPR flags and any pending/in-process status should very likely stay MiEdWorkforce-only given sensitivity and currency requirements. This is also the endpoint with the still-open SSN-inclusion and PPR-scope questions (Caitlin/Beth reviewing as of January 2026) |
| 10 | Professional Learning Activity Event | Submit | — | No — operational, staff-specific |
| 11 | District Job Information Detail | Request | 18.3 / 18.9 | Possibly, for the underlying Local Job Category/Local Job Function/Education Job Type code lists — these already carry a separate Ed-Fi-deprecation risk (see Implementation Notes), and a shared UDP vocabulary could actually help resolve that rather than MiEdWorkforce/MiDataHub each recreating descriptor values; the position/slot records themselves remain staffing-operational |
| 12 | District Job Information Detail | Submit | 18.3 | No — operational (also independently flagged "may be on hold" per the February 9, 2026 meeting; MVP plan is UI-creates/API-maintains, not API-creates) |
| 13 | District Staff Employment Information | Request | 21.9 | No — read of #2 |
| 14 | District Staff Assignment Information | Request | 22.14 | No — read of #3 |
| 15 | District Staff Course Level Information | Request | 22.16a | Same code-list nuance as #4 |
| 16 | District Staff Special Education Information | Request | 22.16b | No — read of #5 |
| 17 | District Staff Career Technical Education Information | Request | 22.16c | Same CIP/Career Cluster nuance as #6 |
| 18 | District Staff Specialized Funding Stream Information | Request | 22.17 | Same nuance as #7 |
| 19 | District Professional Learning Activity Event | Request | — | No — read of #10 |
| 20 | Sponsor Professional Learning Activity Event | Request | — | No — unrelated to UDP; still carries its own open architecture question (should a Sponsor be treated as a district for this purpose, since Sponsors aren't an EEM-recognized organization type?) |

**Environment Strategy**
- **DEV**: MiEdWorkforce DEV exposes endpoints
- **QA/STAGE**: MiEdWorkforce QA exposes endpoints; MiDH QA/Staging calls **[NEEDS INPUT: MiDH environment names?]**
- **PROD**: MiEdWorkforce Prod exposes endpoints; MiDH Prod calls

**What We Need from MiDH**
- **Access & Credentials**: Agreement on OAuth 2.0 / Bearer Token (per source IDD), discussion of credential management workflow
- **Confirmation**: MiDH will initiate requests TO MiEdWorkforce CEDS APIs (push model); MiEdWorkforce will NOT pull from MiDH Ed-Fi APIs
  - **Rationale**: MiDH manages district vendor differences; districts prefer to push data out
- **Documentation**:
  - Sync patterns (real-time vs batch)
  - Error handling and retry logic
  - Sample CEDS JSON-LD payloads for all endpoints
  - Test scenarios (various district types, edge cases)
  - SHACL validation approach — whether validation happens at MiDataHub, at MiEdWorkforce, or both (open as of January 26, 2026 meeting)
- **Scope clarification vs. UDP**: which of the reference/code-list datasets flagged above (CIP Codes, Career Cluster, Local Job Category/Function, Education Job Type) MiDataHub intends to source itself vs. expects to eventually pull from UDP, so MiEdWorkforce isn't left maintaining a third copy of the same vocabulary

**Key Workflows**

**District Submits Staff:**
- District updates SIS
- MiDH translates Ed-Fi to CEDS JSON-LD
- MiDH POSTs to Staff Employment endpoint
- MiEdWorkforce validates Unique ID (Mi-Key), Entity ID (EEM), required fields
- Returns success/error

**District Submits Course Assignments:**
- District assigns teacher to course with SCED/CIP codes
- MiDH POSTs to Teaching Course Information endpoint
- Includes special education indicator if applicable

**Professional Learning Submission:**
- MiDH (or other systems) POST SCECH credits to shared endpoint
- Multiple agencies can submit to same endpoint

**Query Professional Practice Status:**
- MiDH (or public portal) queries Professional Practice Status endpoint
- Returns flags indicating arrest records, disciplinary actions, etc.
- Shared endpoint used across multiple contexts

**District Requests Data Back (Use Cases 11, 13–20):**
- District/vendor system needs to populate its own local copy of job, employment, assignment, course, or professional learning data
- MiDH calls the relevant Request endpoint with District Identifier (+ optional Unique ID / Building Identifier)
- MiEdWorkforce returns the current record(s) as a JSON payload; MiDH forwards to the calling system

**Implementation Notes**
- **Dependencies**: Mi-Key (Unique ID), EEM (Entity ID), standard MiEdWorkforce REST API patterns (auth, error handling, rate limiting)
- **Blocks**: CTEIS, NexSys (rely on staffing data from MiDH)
- **Data Mapping**: MiDH translates Ed-Fi to CEDS JSON-LD before calling MiEdWorkforce; it's possible this conversion occurs entirely on the MiDH side per the source IDD
- **Special Considerations**:
  - Push model: MiDH initiates; MiEdWorkforce does NOT pull from MiDH
  - Shared endpoints: SCECH and Professional Practice Status endpoints serve multiple consumers (not MiDH-specific)
  - Standard API design: Error handling, rate limiting, authentication consistent across all MiEdWorkforce endpoints
  - Identity prerequisite: Unique ID lookup (#1) supports both find-existing and create-on-no-match, so it's more accurate to say identity resolution happens as part of the flow rather than strictly gating it beforehand
  - Graceful degradation: Manual entry available if MiDH unavailable
  - CIP codes: MiDH submits CIP codes for course classifications — see UDP Overlap note on #6/#17
  - Ed-Fi standards risk: Local Job Category/Function map to `StaffClassificationDescriptor` values being deprecated in Ed-Fi post-DS-4.0; MiDataHub may need to recreate custom descriptor values to keep supporting these, which is a durability risk independent of the UDP question
  - Unresolved ownership: whether Unique ID matching (#1) resolves inside MiEdWorkforce or directly against Mi-Key is still an open question on both sides of this engagement, not just an internal SDD gap (raised at the January 26, 2026 meeting, "should we put the question on the action tracker?")

**Outstanding Questions**
1. **Push Model**: Confirm MiDH will initiate TO MiEdWorkforce (not expect pull FROM MiDH)
2. **CEDS Translation**: Can MiDH handle Ed-Fi to CEDS JSON-LD translation? Mapping complete for Phase 3?
3. **Endpoint Paths**: Specific paths for each of the 20 endpoints?
4. **Specialized Funding**: Part of staff position payload or separate endpoint?
5. **Sync Frequency**: Real-time? Batch? District-driven? Individual records vs. bulk — what's the expected split?
6. **Authentication**: Method? Credential timeline per environment?
7. **MiDH Environments**: Names? When can they test against MiEdWorkforce QA?
8. **Identity Ownership**: Does Unique ID matching for #1 occur in MiEdWorkforce or directly in Mi-Key?
9. **SHACL Validation**: Does validation happen at MiDataHub, at MiEdWorkforce, or both?
10. **Snack Pack Scope**: What PPR data should #9 include, and should any form of SSN be returned (full, last-4, or none)?
11. **UDP Boundary**: For CIP Codes, Career Cluster, and job-classification code lists (#6, #11, #17), does MiDataHub/UDP intend to become the canonical source, or should MiEdWorkforce continue to own and serve these values? Should be resolved before building persistent local copies of vocabulary that may migrate to UDP.
12. **Sponsor Modeling**: Does #20 require treating a Professional Learning Sponsor as a district-equivalent entity, or does it need its own modeling path?
13. **Job Creation Scope**: Confirm #12 (Submit Job Info) remains UI-creates/API-maintains only for MVP, not API-creates, per the February 9, 2026 meeting

**Reference Documentation**
- Source IDD: `functional-design-docs/08 - Interface and Integrations Setup/8.4 - Data Hub/8.4 - Data Hub IDD.docx`
- MiDataHub to MiEdWorkforce Mapping spreadsheet (CEDS/Ed-Fi field crosswalk, maintained externally — Google Sheet, not yet mirrored into this repo)
- **[NEEDS INPUT: MiEdWorkforce standard REST API docs (auth, errors, rate limits)]**
- **[NEEDS INPUT: MiDH integration guide]**
- Contacts: Joel Thiele (CEPI, business contact), Adam Nash / Bob Thayer (DTMB), Donald Winans (IDD author), Sean Strom (Technical Owner), Beth Dontje (Product Owner – MDE), Caitlin Groom (Product Owner – CEPI)

### Notes
What goes to Mi-Key and what goes to MEWF? e.g. Identities API
DataHub submits to Staff Employment (21.3) (API for get post staff positions, get roster of positions)
DataHub submits to Teaching Course info, e.g. SCED (for teacher positions)
DataHub submits additional info about whether a course is a spec ed
Get CIP code?
Specialized Funding (either working with Migrant or Title I)

Snack Pack - district may make updates? Updates of what? ("proactive and reactive") - Can all staff see these?
Expose endpoint for SCECH credit hours - tied to date or date range - may be non-CEDS, maybe other PD

**Engagement status**: As of the August 31, 2026 meeting, this engagement had just come off a "reset" ("Revised timelines - what has been established since the restart") — the pre-reset target dates (Mock Service 4/13/2026, QA/Stage 6/22/2026, Stage 11/30/2026, PROD 3/22/2027) should be treated as stale pending confirmation of what survived the reset. Vendor landscape identified as of January 2026: ~7 HRMS vendors + ~5 SIS vendors as the actual systems that would implement Ed-Fi push through MiDataHub.

---

## 12. CTEIS (Career Technical Education Information System)

**System Identity**
- **Purpose**: Tracks CTE (Career Technical Education) course enrollment and instructor assignments; validates CTE instructor identities and reduces duplicate staffing data submission
- **Classification**: Outbound (MiEdWorkforce exposes APIs for CTEIS to consume)
- **Phase**: 3

**Integration Points**
- **Protocol**: REST API (HTTPS)
- **Method**: POST/GET
- **Direction**: CTEIS queries MiEdWorkforce (MiEdWorkforce exposes endpoints based on existing REP API patterns)
- **API Endpoints** (MiEdWorkforce exposes these for CTEIS to call):
  - **GetPersonnelByDistrict**: Returns staff roster for a district (CTE instructors with assignments)
  - **GetPersonnelByPIC**: Returns individual staff record by Unique ID (note: may transition from "PIC" terminology to "UniqueId")
  - **GetPersonnelByCoreFields**: Search staff by demographics (FirstName, LastName, DOB, Gender)
  - **GetCompleteAssignmentCodeList**: Returns valid assignment codes (including CTE-specific codes)
- **Trigger**: CTEIS-initiated (queries MiEdWorkforce on-demand or daily for roster validation)
- **Data Exchanged**:
  - **GetPersonnelByDistrict** response:
    - District/School codes, Unique ID (PIC), Name, DateOfHire, DateOfTermination
    - HighestEducationalCode, AnnualSalary, FundedPositionStatusCode, TitleI flag
    - AssignmentListings: SchoolCode, AssignmentCode, FTE, AccountingCode
  - **GetPersonnelByPIC** response:
    - Unique ID (PIC), Name, Gender, DateOfBirth
    - EmployedDistricts[], EmployedBuildings[]
  - **GetPersonnelByCoreFields** response: Same as GetPersonnelByPIC
  - **GetCompleteAssignmentCodeList** response:
    - Code, Description, ClosedDate, IsEdEffectivenessRequired
- **Frequency**: 
  - Daily roster queries (estimated 1000-2000 requests/day)
  - On-demand validation queries

**Environment Strategy**
- **DEV**: MiEdWorkforce DEV exposes mock endpoints
- **QA/STAGE**: MiEdWorkforce QA exposes endpoints; CTEIS QA queries **[NEEDS INPUT: Does CTEIS have QA/Staging?]**
- **PROD**: MiEdWorkforce Prod exposes endpoints; CTEIS Prod queries

**What We Need from CTEIS**
- **Access & Credentials**: 
  - **[NEEDS INPUT: Authentication method? API token? Certificate?]**
  - **[NEEDS INPUT: When do they need access to each environment? Lead time?]**
- **Documentation**:
  - Confirmation that existing REP API format is acceptable (based on interface doc, appears compatible)
  - CTE assignment code list (which AssignmentCodes indicate CTE instructors?)
  - Query patterns and expected volumes
  - Error handling expectations
- **Test Data**: 
  - Sample district codes for testing GetPersonnelByDistrict
  - Sample Unique IDs for testing GetPersonnelByPIC
  - Test scenarios: CTE instructors, non-CTE staff, terminated employees

**Key Integration Scenarios**

**CTEIS Validates District CTE Roster:**
- CTEIS sends POST to `GetPersonnelByDistrict` with DistrictCode (e.g., "25000")
- MiEdWorkforce returns all staff for that district with assignment details
- CTEIS filters for CTE-specific AssignmentCodes
- CTEIS validates that CTE instructors in their system match MiEdWorkforce records

**CTEIS Validates Individual Instructor:**
- CTEIS has Unique ID (PIC) for instructor
- CTEIS sends POST to `GetPersonnelByPIC` with PIC value
- MiEdWorkforce returns individual staff record with employment status
- CTEIS confirms instructor is actively employed at expected district/building

**CTEIS Searches by Demographics:**
- District submits CTE data to CTEIS without Unique ID
- CTEIS sends POST to `GetPersonnelByCoreFields` with name/DOB/gender
- MiEdWorkforce returns matching staff records (may be multiple)
- CTEIS prompts district to select correct match and records Unique ID

**Implementation Notes**
- **Dependencies**: 
  - Phase 1 (Mi-Key): Unique ID assignment and validation
  - Phase 3 (EEM): Entity codes for district/building validation
  - Phase 3 (MiDH): Staffing roster data sourced from districts
- **Blocks**: CTE instructor validation for districts (without this, CTEIS cannot validate staff)
- **Data Mapping**: Response format is CEPI-specific (not CEDS); based on legacy REP API structure
- **Special Considerations**:
  - **PIC vs Unique ID terminology**: Endpoints use "PIC" but will return Unique ID (they're the same after migration)
  - **Shared endpoints**: These same API endpoints are also used by TSDL, NexSys, MEGS+, and potentially others
  - **Reduces duplicate submission**: Districts currently submit CTE instructor data separately to both REP and CTEIS; this integration eliminates that duplication
  - **TSDL vs CTEIS**: Students reported in CTEIS are NOT reported in TSDL (mutually exclusive populations)
  - **Legacy system replacement**: This is replacing functionality from REP "PIC Public Services"
  - **Coordination with Mi-Key team**: Must ensure identity validation services point to correct system (MiEdWorkforce or Mi-Key) and avoid service duplication
  - **Assignment code filtering**: CTEIS will need to filter GetPersonnelByDistrict results for CTE-specific assignment codes
  - **Data volume**: ~1000-2000 API requests per day (mix of district roster pulls and individual validations)

**Outstanding Questions**
1. **Authentication**: What method does CTEIS prefer? API token? Certificate-based? Azure AD?
2. **Environment Access**: When does CTEIS need access to DEV/QA/PROD endpoints? Lead time for credential provisioning?
3. **CTEIS Environments**: Does CTEIS have QA/Staging environment, or only Production?
4. **CTE Assignment Codes**: Which specific AssignmentCodes indicate CTE instructors? (for documentation/testing)
5. **Error Handling**: What should happen if district code is invalid? Staff not found? How should MiEdWorkforce communicate errors?
6. **Rate Limiting**: Are there concerns about 1000-2000 requests/day? Should we implement rate limiting or throttling?
7. **Response Format Changes**: Does CTEIS need any modifications to the existing REP API response format, or is it fully compatible?
8. **Credential Validation**: Does CTEIS need additional fields related to CTE credentials beyond what's in GetPersonnelByDistrict?
9. **Service Coordination**: Who is coordinating with Mi-Key team to ensure identity validation services aren't duplicated?

**Reference Documentation**
- Interface Doc: "CTEIS" (Donald Winans, 8/1/2025)
- Based on existing REP API "TeacherInformation" service endpoints
- Technical Contact: Donald Winans (winansd@michigan.gov), Andrew Marsh (marsha4@michigan.gov)

---

## 13. NexSys

**System Identity**
- **Purpose**: Supports Title I Part A Comparability reporting; receives staff lists and assignment details to fulfill federal requirements
- **Classification**: Outbound
- **Phase**: 4

**Integration Points**
- **API Endpoints**: **[NEEDS INPUT: Specific endpoint names/paths - REP-style GetPersonnelByDistrict pattern?]**
  - **[NexSys pulls staff roster and assignment details from MiEdWorkforce]**
  - **[Possible filtering by entity, date range, etc.]**
- **Trigger**: NexSys-initiated (pulls from MiEdWorkforce)
- **Data Exchanged**: Staff Unique ID, Entity ID, Assignment Details, Position, FTE, Salary (for comparability calculations), **[NEEDS INPUT: Other fields required for Title I?]**
- **Frequency**: At least monthly; as needed for Title I reporting

**Environment Strategy**
- **DEV**: Mocked - Log outbound data without actual transmission
- **QA**: **[NEEDS INPUT: NexSys Test/Staging environment?]**
- **PROD**: NexSys Production

**What We Need from NexSys**
- **Access & Credentials**: 
  - **[NEEDS INPUT: Authentication method for NexSys to access MiEdWorkforce API?]**
  - **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - Data requirements for Title I Comparability reporting
  - Required fields and formats
  - Query parameters needed (entity filters, date ranges)
  - Expected data volumes and response times
- **Test Data**: 
  - **[NEEDS INPUT: Does NexSys provide test queries/scenarios?]**

**Implementation Notes**
- **Dependencies**: Phase 3 (EEM, MiDH, CTEIS) - complete staffing data needed
- **Blocks**: Title I comparability reporting
- **Data Mapping**: **[NEEDS INPUT: CEDS-compliant or needs transformation?]**
- **Special Considerations**: Monthly reporting window provides buffer for issues. Approximately 20MB per month data volume. REP-style API exposure (GetPersonnelByDistrict pattern).

---

## 14. SendGrid

**System Identity**
- **Purpose**: Enterprise email solution for high-volume automated communications (submission confirmations, credential approvals, alerts)
- **Classification**: Outbound
- **Phase**: 4

**Integration Points**
- **API Endpoints**: SendGrid REST API
  - `POST /v3/mail/send` - Send single email
  - **[NEEDS INPUT: Using transactional templates? Dynamic template endpoint?]**
  - **[NEEDS INPUT: Batch sending endpoint if needed?]**
- **Trigger**: MiEdWorkforce-initiated (event-driven: submissions, approvals, alerts)
- **Data Exchanged**: Recipient email, Subject, Body/Template ID, Template variables, Attachments (if any)
- **Frequency**: Real-time (triggered by system events)

**Environment Strategy**
- **DEV**: **[NEEDS INPUT: Mock email service (log emails without sending)? Or SendGrid dev/sandbox account?]**
- **QA**: **[NEEDS INPUT: SendGrid test account or production with test templates/recipients?]**
- **PROD**: SendGrid Production

**What We Need from SendGrid**
- **Access & Credentials**: 
  - API Key for each environment
  - **[NEEDS INPUT: When needed? Lead time for account setup?]**
- **Documentation**: 
  - SendGrid API documentation (already public)
  - Template creation and management guidance
  - Webhook setup for delivery status tracking
  - Best practices for high-volume sending
  - Bounce/spam rate management
- **Test Data**: 
  - Test email addresses for various scenarios
  - Template testing procedures

**Implementation Notes**
- **Dependencies**: None (can operate without automated emails initially)
- **Blocks**: Automated user notifications
- **Data Mapping**: None (email content generation is internal to MiEdWorkforce)
- **Special Considerations**: Use Azure Service Bus to queue emails if SendGrid unavailable. Not blocking for transactions - graceful degradation.

---

## 15. PowerBI

**System Identity**
- **Purpose**: Primary reporting solution for data quality dashboards; helps administrators identify and resolve data anomalies
- **Classification**: Outbound
- **Phase**: 4

**Integration Points**
- **Data Connection Method**: **[NEEDS INPUT: Direct Query to Azure SQL/Cosmos? Import mode? Azure Synapse Analytics connection? Parquet files from data lake?]**
- **Data Sources Accessed**: **[NEEDS INPUT: Which specific databases/data stores? MiEdWorkforce transactional DB? CEPI DW? Both?]**
- **Trigger**: PowerBI-initiated (scheduled refresh or real-time Direct Query)
- **Data Exchanged**: All transactional and warehouse data for reporting/visualization
- **Frequency**: **[NEEDS INPUT: Real-time refresh? Hourly? Daily? Multiple schedules?]**

**Environment Strategy**
- **DEV**: **[NEEDS INPUT: PowerBI connected to DEV database? Or separate dev workspace?]**
- **QA**: **[NEEDS INPUT: QA workspace connected to QA data sources?]**
- **PROD**: PowerBI Production workspace

**What We Need from PowerBI Team/Microsoft**
- **Access & Credentials**: 
  - Azure AD service principal for data source access (per environment)
  - PowerBI workspace creation and permissions
  - **[NEEDS INPUT: When needed? Lead time?]**
- **Documentation**: 
  - Data source connection best practices
  - Direct Query vs. Import mode guidance for Azure architecture
  - Row-level security implementation (if needed for entity-based access)
  - Performance optimization for large datasets
- **Test Data**: 
  - **[NEEDS INPUT: Sample dashboard designs? Report requirements from stakeholders?]**

**Implementation Notes**
- **Dependencies**: Phase 3 (CEPI Data Warehouse) - historical data needed for comprehensive dashboards
- **Blocks**: Data quality dashboards, administrative reporting (not blocking for transactions)
- **Data Mapping**: May require data transformations in Power Query; no external system mapping
- **Special Considerations**: PowerBI outage doesn't impact transactional system. Focus on data quality dashboards to identify anomalies.

---

## Summary: Critical Information Still Needed

To complete this integration plan, the following information is most critical:

### **Immediate Priority (Needed for all/most integrations):**
1. **Specific API endpoint paths/names** for each integration
2. **Authentication methods** (Bearer token, API key, OAuth, certificate, Azure AD)
3. **Environment names** for external systems (Staging vs. QA vs. Production)
4. **Credential request timing** and lead times from each vendor
5. **Data volumes and frequencies** (API calls/day, file sizes, batch schedules)

### **High Priority (Needed for implementation planning):**
1. **Data format specifics** (JSON schema, CSV structure, Parquet layout)
2. **Test data availability** in external staging environments
3. **Rate limits or usage quotas** from external APIs
4. **Open decisions** (e.g., NASDTEC check timing, MiDH integration frequency)

### **Medium Priority (Needed for coordination):**
1. **Team member assignments** (credential owners for each integration)
2. **Target completion dates** for each phase
3. **Vendor contact information** for coordination
