# Sequence Diagram Standardization Review

Scope: the 11 area sequence documents (credentialing, communications, documents, epp, iam, organizations, payments, proflearning, profpractice, reporting, staffing). 144 sections, 142 Mermaid diagrams in 139 sections, 5 sections with no diagram. Per-diagram metrics are in `sequence-metrics.csv` (same folder). No existing file was edited.

## 1. Summary of findings

1. **Size is mostly fine, but a tail is too big and mixes workflows.** Median diagram: 5 participants, 18 arrows, nesting depth 1. Against the proposed limits (7 participants, 25 arrows, depth 2, 6 notes, 6 self arrows), 51 of 142 diagrams exceed at least one and 29 exceed two or more. The worst offenders bundle several workflows (request, approve and execute; add, edit and delete; job plus per-record handling). Section 3 ranks them and proposes split points.
2. **Authorization checks are drawn in 53 of 142 diagrams (37%), in five different styles.** 92 diagrams (65%) draw none (20 of those are IAM, where authorization is the subject). Of the 47 arrows that call IAM, 42 start at the UI. That conflicts with the design: the IAM permission check endpoint is marked `x-access: internal-service` in iam-api.yml and the API standards say the backend enforces permissions. 35 of the 47 answers are a bare "Permission confirmed". **Recommendation: do not draw permission checks.** State the permission key in the "Who" line instead. Draw an IAM call only when its result changes the flow (section 2.3).
3. **Participant naming is inconsistent.** The same service appears under many labels (IAM: 10 labels, Communications and Notification: 9, Credentialing: 7). The event bus has 4 names (EventBus 47, Event Bus 39, ServiceBus 6, EventGrid 4). The UI has 22 different labels. 12 diagrams use 18 participants that are never declared.
4. **The three API kinds are never labeled on arrows.** Only 352 of 1,413 cross-participant request arrows (25%) carry a verb and path, and almost none mention the auth mechanism. Wiring is inconsistent: UI calls other domains' APIs directly in some areas and goes through the owning service in others, and the Documents area contradicts its own "domain API calls Documents" pattern.
5. **External systems are drawn inconsistently.** 36 external-system participants are declared `participant` and 1 is declared `actor`, against the template rule (actor = external system). Mi-Key appears as "Mi-Key API", "Mi-Key (external)" and "Identity Resolution". 8 of the 15 catalog systems appear as a participant; MiDH, CTEIS, NexSys, MSDS TSDL, STARR and Pearson EdReports (MTTC) have no sequence.
6. **Events and notifications are drawn five ways.** 183 arrows publish to a bus participant, but Staffing publishes 15 "events" straight to CommAPI, IAM and Documents call Notification directly (33 arrows), and EPP, Payments and Proflearning draw the bus calling Communications.
7. **Disagreements with the API Architecture** (section 3.4): UI calling the IAM service API, four inbound external calls (webhooks, callbacks, public form) with four different treatments, external HR/SIS callers of Staffing never drawn, and "Managed Identity" used where the architecture says mTLS.

## 2. Style guide

### 2.1 Subject and size

- One diagram, one subject: one trigger, one outcome, one owning area. The owning service is the subject; other areas appear as a single participant at the point of call, and their internals are referenced ("see Evaluate PPR Clearance"), not redrawn.
- Split when a diagram has more than one actor stage (request, approve, execute), more than one CRUD verb, a batch job plus its per-record handling, or a publisher plus the consumer's reaction. The template already says to split flows with more than about 4 branches.
- Limits per diagram:

| Measure | Target | Hard limit |
|---|---|---|
| Participants (including actor and UI) | 4 to 6 | 7 |
| Arrows (requests plus responses) | 12 to 18 | 25 |
| alt/opt/loop/par nesting | 1 | 2 |
| Notes | 0 to 3 | 5 |
| Self arrows | 0 to 2 | 4 |

- Replace nested decision chains (status transitions, rule priority lists, command codes) with a decision table in the text blocks, and draw one representative path plus one rejection path.
- Omit plain return arrows ("OK", "Confirmed"). Keep a response only when it carries data used later or a status code on an error path. Responses are 856 of 2,769 arrows (31%).
- Do not draw database, repository or cache participants unless storage is the subject (Blob upload by SAS, the Organizations cache diagrams). Use a self arrow or a Note ("persist X"). Documents, Communications and Organizations draw 126 database or cache arrows.
- Keep open questions and editorial remarks out of diagrams (for example the "not yet resolved which endpoint" note in Application Auto-Approval); route them to the tracking files.

### 2.2 Participants

- Order left to right: human actor, UI, owning API, other areas' APIs, Event Bus, external systems.
- Actor is for humans only. Every non-human, including external systems, is a `participant`. Update the Conventions text in the template accordingly.
- Group with Mermaid `box` so the API kind is visible without reading labels: `box Browser` (UI), `box MiEdWorkforce (AKS)` (services, Event Bus), `box External` (external systems). `box` needs Mermaid 10 or later; confirm the docs renderer supports it before adopting.
- Always declare every participant with `participant Alias as Label` (no implicit participants). One UI participant per diagram: alias `UI`, label "UI" ("Admin UI" only if a second frontend is really involved). Do not use roles as participants (AuditorRole, ISDAuditor as participant).
- Canonical service labels: `<Name> API` using the architecture names: Credentialing API, EPP API, IAM API, Professional Learning API, PPR API, Staffing API, Communications API, Documents API, Organizations API, Payments API, Reporting API. Alias is `<Short>Api` (CredApi, EppApi, IamApi). Never use "Application Service", "Domain Service", "Notification", "Identity", "Credentials", "CommAPI" or "IAM_API" for these.
- Event Bus: alias `EventBus`, label "Event Bus". Do not draw Event Grid or Service Bus as separate participants (Event Grid is "optional / TBD" in the architecture) unless the Azure callback is the subject.
- Clarify before use: "Business Rule Engine" (10 diagrams) is not defined in solution-architecture.md. If it is a library inside the owning service it should not be a participant.

### 2.3 Authorization checks

**Recommendation: do not represent authorization checks as arrows.**

Reasons:
1. Consistency: 65% of diagrams already omit them, and where drawn they use five styles (section 4.3), so they are not a reliable guide today.
2. Correctness: 42 of 47 drawn checks go from the UI to IAM. The check is a Service API called by the owning service, so the drawn flow misstates the design.
3. Low information: 35 of 47 answers are a bare "Permission confirmed". The permission key is already recorded in the permissions catalog, the API catalog ("Permissions Needed" column) and the OpenAPI `x-permissions-required`.
4. Cost: 109 arrows (4% of all arrows) and one participant in 43 diagrams. The saving is modest; the gain is mainly consistency and correctness.

What to do instead:
- Add to the "Who" line: `Permission: epp.enrollment.verify (scoped to the caller's EPP)`. This keeps the template rule "every permission is traceable to at least one sequence" and can be checked by script.
- Add one standing sentence to the Conventions list: "Every application API call is authorized by the owning service through the cached IAM permission check (Service API). It is not drawn unless noted."
- Draw an IAM call only when (a) the result drives later steps (Reporting: org scope feeds parameter resolution), (b) the denial path is the point of the diagram, or (c) IAM is the subject. Then draw one arrow from the owning API, never from the UI, tagged as a service call, and keep the response only because it carries data.
- UI-side show or hide checks are a frontend concern and are not drawn.

Alternative if the owner wants checks visible: one standard form only. The owning API calls IAM once, right after receiving the request, response drawn only on a deny branch:

```mermaid
EppApi->>IamApi: SVC POST /permissions/check (epp.enrollment.verify)
Note over IamApi: Cached 5 min
alt Denied
    EppApi-->>UI: 403 Forbidden
end
```

Retire the UI-to-IAM arrows, the "Validate permissions" self arrows, and the "caller already authorized" notes.

### 2.4 Labeling the three API kinds (plus outbound)

The architecture defines Application, External and Service APIs. All three describe calls into MiEdWorkforce. Calls out to an external system (SendGrid, CEPAS, Power BI, NASDTEC, CHRISS) are none of the three, so add a fourth tag.

| Tag | Meaning | Typical participants |
|---|---|---|
| `APP` | Application API, user delegated token, UI hostname under /api | UI to owning API |
| `SVC` | Service API, in-cluster mTLS, never through APIM | API to API |
| `EXT` | External API, inbound OAuth client credentials on the API hostname | external system or provider to a distinct external-facing API participant |
| `OUT` | Outbound call to an external system (protocol shown) | API to external system |

Rules:
- Every request arrow between participants starts with its tag, then verb and path copied from the API catalog (no body fields), for example `APP POST /candidate-enrollments/{id}/accept`. Responses carry no tag.
- Drop the `/api` prefix and `/create`-style verbs (spot checks: Staffing draws `POST /positions/create` and `PATCH /positions/{id}/status` where the catalog has `POST /positions` and `POST /positions/{positionId}/status`; Communications draws `/api/communications/...` paths that are not in the catalog).
- Draw external-facing endpoints as their own participant ("Webhook Endpoint", "External API"), not as the main service, because External APIs are handled by different services. Communications and Profpractice already do this; IAM and Documents do not.
- Show browser redirect flows (MiLogin, CEPAS) with the user's browser as the participant that makes the call, as Payments does.
- Unauthenticated public endpoints (removal request form, public catalog search) are listed as `external-public` in the API catalog, a fifth case the three-kind model does not name. Tag them `EXT` with "public" in the label until the architecture says otherwise.

### 2.5 Events

- Publish with `Api--)EventBus: EventName` (async arrow), no wording like "Publish X event". Every event listed under Events Published is drawn once.
- Never publish directly to another domain (Staffing to CommAPI, 15 arrows).
- Do not draw email sends in other areas' diagrams. A notification is an event on the bus with a Note: "Consumed by Communications (sends email to the candidate)". Draw consumers only in a diagram whose subject is the consumer's reaction (for example Auto-Approval starting from `EventBus->>CredApi`).
- External systems never publish to our Event Bus (the IAM "Identity Resolution" participant does). IAM publishes after the callback.

### 2.6 External systems

Use the canonical names from solution-integrations.md as the label and a short alias, inside `box External`:

| Label | Alias | Integration pattern to show (from the catalog) |
|---|---|---|
| MiLogin | MiLogin | OIDC redirect (browser as caller) |
| Mi-Key | MiKey | `OUT` REST match/resolve; callbacks as `EXT` |
| Educational Entity Master (EEM) | EEM | Inbound org data; reached through Organizations API, not directly |
| MSP CHRISS Rap Back | MspRapBack | `EXT` webhook in; `OUT` SOAP GET_RAPBACK |
| Pearson EdReports (MTTC) | EdReports | Inbound results |
| NASDTEC Clearinghouse | NASDTEC | `OUT` REST GET |
| STARR, MSDS TSDL, CEPI, Michigan DataHub (MiDH), CTEIS, NexSys | as named | MiDH and CTEIS call us (`EXT`) |
| CEPAS | CEPAS | Browser redirect, `OUT` REST refund, SFTP posting file |
| SendGrid | SendGrid | `OUT` send; `EXT` webhook |
| Power BI | PowerBi | `OUT` REST GenerateToken; embed in browser |

Do not use suffixes such as "API", "SOAP API" or "(external)" in the label; put the protocol in the arrow.

### 2.7 Error paths

- One `alt` for the most important business failure per diagram; show the status code on the response arrow (`403`, `409`, `422`).
- All other failures go in the Error Scenarios text block. Do not nest error branches inside happy-path branches.
- Variants get their own diagram titled `<Area> - <Flow> - <Variant>` and start at the point of divergence (Reporting "Generate Embed Token" with Happy Path plus variants is the good model; Payments variants repeat the whole redirect preamble).

### 2.8 Content that is not a sequence

Standards, decision matrices, API specs and notes do not belong under a sequence heading. See section 3.3.

## 3. Diagrams to revisit

Metrics: P participants, A arrows, D nesting depth, N notes, S self arrows. Split points marked "(inferred)" come from the diagram's alt/loop structure, not from a full reading.

### 3.1 Ranked list (biggest offenders first)

| # | Area | Section | Metrics | Problem | Recommended change |
|---|---|---|---|---|---|
| 1 | documents | Staged File Upload for Synapse Processing | P10 A45 N15 | Too long; four workflows (upload, scan, Synapse processing, 7-day cleanup); both EventGrid and Defender drawn; 14 database arrows; UI calls Documents API directly | Split into "Documents - Stage and Scan Import File", "Documents - Process Staged File in Synapse", "Documents - Staging File Cleanup Job". Drop MetadataDB, merge Defender and EventGrid into one scan callback |
| 2 | credentialing | Application Auto-Approval | P6 A21 D4 S9 N8 | Depth 4 decision chain; 9 self arrows; contains open-question notes | Decision table in text; diagrams "Auto-Approve Application" (happy path) and "Auto-Approval Held for Manual Review" (PPR hold, conditional, rule failure). Move open questions to tracking |
| 3 | documents | Administrator Requests Retention Override | P7 A40 N13 | Three workflows separated by Note banners; auth by self arrows; "Notifications" naming | Split into "Request Retention Override", "Approve Retention Override" (dual approval alt), "Execute Approved Override" |
| 4 | epp | Recommend Candidate for Credential | P8 A40 D3 | UI fans out to Credentialing API (x2), EPP API and IAM; nested validation chain | Split into "Load Recommendation Context" and "Recommend Candidate for Credential"; EPP API calls Credentialing API (`SVC`); rules to a table |
| 5 | documents | Document Upload with Malware Scanning | P8 A34 N10 | Upload request and scan result are separate flows; 10 database arrows; UI calls Documents API, contradicting the pattern in the same file | Split into "Request Upload and Upload File" and "Process Malware Scan Result" (reused by 6 and 22) |
| 6 | documents | Document Replacement and Versioning | P9 A32 N9 | Same upload plus scan plus archive | Start from the replace request; reuse "Process Malware Scan Result"; show archive as one step |
| 7 | epp | Manage Candidate Programs | P4 A38 D3 S9 | Add, edit and delete in one alt | "Add Candidate Program" (duplicate check), "Remove Candidate Program" (status rules); edit as text |
| 8 | staffing | Update Position Status | P4 A27 D3 S10 L58 | Five target statuses in nested alts | Transition table plus one diagram with a single rejection path |
| 9 | staffing | Assign Employee to Position | P6 A39 | Validation, create, and justification variants; undeclared participant; roles as participants | "Validate Placement", "Create Assignment", "Create Assignment with Credential Error Justification" |
| 10 | staffing | Update Employee Demographics | P5 A35 D3 | Update, Mi-Key near-match resolution and cross-notification in one | "Update Employee Demographics" and "Resolve Near Match on Demographic Update" (reference the IAM near-match diagram) |
| 11 | iam | Identity Administrator - Resolve Identity Request | P6 A34 D3 S9 | Two request families and origin-based follow-ups | "Resolve Identity Request (Match, Create New, Deny)" and "Resolve Link ID Request"; origin follow-ups as a table |
| 12 | iam | Citizen User - Initial Sign-In & Identity Resolution | P6 A26 D3 S10 | Sign-in, account creation and Mi-Key outcomes in one; Mi-Key named "Identity Resolution", publishes events and replies to the citizen; undeclared Notification | "Citizen Sign-In and Authorization Check" and "Citizen Account Creation (Mi-Key Match)"; IAM publishes events |
| 13 | payments | Daily Posting File Reconciliation | P6 A26 D3 S9 | Batch plus per-record handling; Monitoring receives emails from the bus | "Reconcile Posting File" and "Reconcile One Posting Record"; command codes to a table |
| 14 | payments | Refund Request Processing (Single) and (Dual) | P8 A26 / A33 | Same flow twice; undeclared participant | One diagram "Refund Request and Approval" with an amount-threshold alt |
| 15 | credentialing | Submit Permit Application, Submit New Certificate Application, Renew Credential, Issue Temporary Permit | P7-8 A29-34 D2-3 | Same eligibility check plus submit pattern; PPR branches redrawn | Split each into "Check Eligibility" and "Submit Application"; reference one PPR clearance diagram |
| 16 | communications | Event-Triggered Email Send, Email Resend, Manual Email Send | P7-9 A23-30 | Repository participants; "Identity" naming; SendGrid declared participant | Event-triggered: "Render Email from Event" and "Send and Record Email". Resend: "Load Resend Context" and "Resend Email". Drop repositories |
| 17 | documents | Bulk Document Upload | P9 A30 | The doc says bulk is a UX distinction only | Merge into "Upload Document" with a loop note |
| 18 | documents | Soft Delete and Hard Delete Lifecycle | P7 A15 N7 | Two lifecycles | "Soft Delete Document" and "Hard Delete Expired Documents (Nightly Job)" |
| 19 | staffing | Certify Collection, ISD Auditor Review | A31, A33 | Multi-phase; roles as participants; events to CommAPI | Certify: "Run Quality Review" and "Certify Collection". Audit: "Request Documentation and Record Findings" and "Finalize ISD Audit" (inferred) |
| 20 | epp | Update Candidate Enrollment Status | P6 A33 D3 | Bulk status path with nested rules | Rules to a table; one diagram (inferred) |
| 21 | epp | Manage EPP Approved Endorsements, Manage EPP Certificate Category Approvals | A38, A34 | Add, edit and delete bundled | Same treatment as 7 |
| 22 | profpractice | Manage Account Markers | P5 A35 | Four marker actions, 8 event arrows | One generic diagram; marker table; permission keys to the Who line |
| 23 | epp | Bulk Upload Candidate Tracking Data | P7 A28 | UI calls Document Service directly; Synapse inside | "Upload Tracking File" and "Process Tracking File" |
| 24 | payments | Bulk Payment Processing and the four Individual Payment variants | A32, A17-21 | Redirect preamble repeated; bulk has 9 self arrows | Base diagram plus variants starting at the divergence |
| 25 | communications | Mass Email with Consolidation | P8 A20 N9 | Dense notes; SendGrid participant | Move consolidation rules to text (inferred) |

### 3.2 Other diagrams over a limit

| Area | Section | Over |
|---|---|---|
| credentialing | Add Endorsement to Certificate; Configure Credential Definition; Configure Endorsement Definition | arrows |
| credentialing | Validate Assessment Results | depth, notes |
| communications | Template Version Update and Activation | arrows |
| communications | SendGrid Webhook - Delivery Status Update | depth |
| communications | Template Variable Resolution (Internal) | notes (internal component, not a workflow) |
| documents | Legal Hold Application and Release; Bulk Document Soft Delete with Validation | notes |
| iam | Lead Administrator Bootstrap; Citizen User - Update Account Demographics | depth |
| iam | Business User - Request New ID (Near Match Escalation) | arrows |
| profpractice | Submit Professional Practice Review Response | notes |
| profpractice | Review Disclosure and Update Status | arrows |
| profpractice | Process NASDTEC Nightly Batch | depth |
| profpractice | Evaluate PPR Clearance for Credential Application | notes, self arrows |
| staffing | Add New Employee | arrows, depth |

### 3.3 Sections that are not sequences, or are better elsewhere

| Area | Section | Why | Suggested home |
|---|---|---|---|
| communications | Integration Flows (line 890) | API spec and SendGrid REST examples, no diagram | communications-api.yml and the SendGrid section of solution-integrations.md |
| communications | Template Variable Formatting Standards | Standard | Communications capability doc |
| communications | Event Payload Design Standards | Standard | Event standards in patterns-and-principles, or the capability doc |
| communications | Resend vs. New Send Decision Matrix | Matrix | Key Decisions of Email Resend, or the capability doc |
| iam | Notes (UI Context Management) | Standard about X-Organization-Context | iam-domain.md or patterns-and-principles/authentication.md |
| iam | Integration Flows | Tables of Organizations calls and caching, plus a MiLogin sign-in diagram that overlaps the sign-in diagrams | solution-integrations.md (MiLogin, Mi-Key) and the Organizations capability |
| documents | Integration Pattern: Domain API Calling Documents | A pattern; real sequence but contradicted by 9 sibling diagrams | patterns-and-principles, linked from Documents |
| profpractice | Evaluate PPR Clearance; Evaluate Roster Eligibility | Priority rules, not interaction (8 and 6 self arrows) | Decision tables in the domain doc; keep a two-arrow sequence |
| credentialing | Validate Assessment Results | State machine logic | Decision table |
| payments | Reconciliation command code handling | Matrix | Table in the capability doc |
| staffing | Update Position Status | Transition matrix | Table in the domain doc |
| read-only flows | 24 diagrams with no state change and no events (nine in EPP: search and view) | Little logic beyond cross-domain reads | Keep only those that call other areas (view detail); fold the rest into one "search and view" diagram per queue, or the API catalog |

### 3.4 Disagreements with the API Architecture

| # | Where | What the diagram shows | Architecture or catalog says |
|---|---|---|---|
| 1 | credentialing (9 diagrams), epp (24), proflearning (2), profpractice (3) | UI calls IAM to verify permission | Permission check is `internal-service` (Service API) called by the owning service |
| 2 | communications SendGrid Webhook | `POST /api/webhooks/sendgrid`, HMAC | Catalog: internal-service, not through APIM. Architecture: webhooks are External APIs with client credentials |
| 3 | profpractice Receive Rap Back Notification | `POST /webhooks/rapback` | Catalog: external-client through APIM. Different from item 2 |
| 4 | documents Upload diagrams | Event Grid calls `/api/webhooks/defender-scan-result` on the Documents API | Catalog: internal-service; third treatment |
| 5 | iam Split/Retire ID | Mi-Key callback drawn into the same IAM participant | External callbacks should reach a separate external-facing service |
| 6 | iam External Party removal request | Public Website calls IAM, unauthenticated | Catalog: external-public through APIM; not one of the three kinds, drawn as the main IAM service |
| 7 | staffing (all) | Only DistrictUser through StaffingUI | Catalog: many Staffing endpoints are also callable by external HR/SIS systems (external-client); never drawn |
| 8 | documents Integration Pattern | "Managed Identity" for service-to-service | Architecture: Istio STRICT mTLS and service accounts; Managed Identity is for Azure services |
| 9 | documents (9 diagrams), plus 10 UI arrows in credentialing, epp, profpractice, proflearning | UI calls Documents API directly | Pattern says domain API authorizes then calls Documents; Documents does not accept direct browser calls |
| 10 | staffing (15 arrows), iam and documents (33 arrows) | Events to CommAPI, direct Notification calls | Domains publish to their own Service Bus topic |
| 11 | iam Citizen Sign-In and others | Mi-Key ("Identity Resolution") publishes `UniqueIdAssigned` and replies to the citizen | An external system does not publish to our bus |
| 12 | organizations CEPI Nightly Sync vs epp and staffing | Org data source is "CEPI CEDS JSON-LD API"; epp and staffing call "Organization Reference Data API" and "EEMAPI" | Catalog: EEM is the organization source; callers should use Organizations API. Confirm intended source |
| 13 | credentialing, epp | "Reference Data API" and "Credentialing API" for `/endorsement-definitions` | No Reference Data API in the catalog; those paths are listed under Credentialing |
| 14 | iam (19 of 20 diagrams) | Human actor calls IAM directly, no UI | Application APIs are called by the frontend |

Not drawn at all (coverage, not disagreement): no sequence for MiDH or CTEIS (both External API integrations), NexSys, MSDS TSDL, STARR, EdReports. Pearson EdReports is named in 3 lines of credentialing text only.

## 4. Evidence

### 4.1 Method and heuristics

Script (Node, kept in scratch space) parsed each `## ` section, extracted every ```mermaid block and counted:
- Participants: declared `participant` or `actor` lines plus any name used in an arrow but never declared. The CSV shows declared and implicit separately.
- Arrows: lines with `->>`, `-->>`, `--)`, `-)`, `->`, `-->`. Requests are non-dashed arrows (including `--)`); responses are `-->>`.
- Nesting: depth of `alt`, `opt`, `loop`, `par`, `critical`, `break`, `rect`, closed by `end`.
- Notes: lines starting with `Note`. Lines: non-blank lines after `sequenceDiagram`.
- auth_iam: request arrow from a non-actor to IAM, IAM API, IAM_API or Identity & Access API whose text mentions permission, authoriz or allowed, or any `/permissions/check`. Not counted in the IAM area.
- auth_self: self arrow whose text starts with Validate, Check or Verify and mentions permission, or starts with "Validate: user has".
- events: request arrow whose target label matches Event Bus, ServiceBus or EventGrid. events_direct_non_bus: `--)` arrows with "event" in the text and a non-bus target.
- external: request arrow with a named external system at either end (SendGrid, CEPAS, MiLogin, Mi-Key including "Identity Resolution", CEPI, NASDTEC, MSP CHRISS Rap Back, Power BI, EEM). Azure platform services (Synapse, Defender, Blob) are not counted.
- cross_domain: request arrow to a service of another area (label regex per area), excluding UI, storage, bus, actors and IAM permission checks.
- db_storage: target label contains DB, database, repository, storage, container, cosmos, redis, cache or data warehouse.
- http_labeled: text starts with a verb and a path or URL. kind_marked: text mentions mTLS, OAuth, bearer and similar.
Limits: label regexes are approximate; counts are of arrows, not diagrams; the CSV is the authoritative detail.

### 4.2 Metrics by area

| Area | Diagrams | Median participants | Median arrows | Max arrows | Over a limit | With drawn auth check |
|---|---|---|---|---|---|---|
| credentialing | 18 | 6 | 22 | 34 | 9 | 9 |
| communications | 11 | 6 | 20 | 30 | 7 | 5 |
| documents | 10 | 7 | 30 | 45 | 8 | 4 |
| epp | 26 | 6 | 17 | 40 | 6 | 24 |
| iam | 20 | 5 | 17 | 34 | 5 | 0 (subject) |
| organizations | 6 | 5 | 9 | 16 | 0 | 0 |
| payments | 11 | 7 | 22 | 33 | 5 | 1 |
| proflearning | 10 | 6 | 19 | 23 | 0 | 2 |
| profpractice | 15 | 5 | 15 | 35 | 5 | 3 |
| reporting | 6 | 4 | 8 | 14 | 0 | 4 |
| staffing | 9 | 5 | 27 | 39 | 6 | 1 |
| All | 142 | 5 | 18 | 45 | 51 | 53 |

| Area | Arrows | Requests | Self | To bus | Direct non-bus events | External | Cross-domain | DB or cache | Verb and path |
|---|---|---|---|---|---|---|---|---|---|
| credentialing | 390 | 251 | 49 | 33 | 0 | 0 | 19 | 0 | 67 |
| communications | 227 | 148 | 40 | 16 | 0 | 5 | 7 | 29 | 27 |
| documents | 258 | 187 | 30 | 24 | 0 | 0 | 3 | 75 | 15 |
| epp | 531 | 360 | 88 | 20 | 0 | 0 | 35 | 0 | 75 |
| iam | 330 | 251 | 86 | 36 | 0 | 17 | 43 | 0 | 12 |
| organizations | 59 | 40 | 5 | 3 | 0 | 2 | 0 | 22 | 6 |
| payments | 257 | 195 | 63 | 18 | 14 | 16 | 14 | 0 | 23 |
| proflearning | 176 | 128 | 30 | 9 | 4 | 0 | 7 | 0 | 35 |
| profpractice | 246 | 177 | 49 | 24 | 0 | 7 | 5 | 0 | 42 |
| reporting | 53 | 32 | 9 | 0 | 0 | 2 | 1 | 0 | 14 |
| staffing | 242 | 144 | 51 | 0 | 15 | 1 | 20 | 0 | 36 |
| All | 2,769 | 1,913 | 500 | 183 | 33 | 50 | 154 | 126 | 352 |

Requests include 500 self arrows (26%); cross-participant requests are 1,413.

### 4.3 How authorization is drawn

| Style | Diagrams | Where |
|---|---|---|
| UI to IAM "Verify permission (key)", then "Permission confirmed" (sometimes "with EPP scope"), at the start of the flow | 36 | credentialing 9, epp 24, profpractice 3 |
| UI to IAM "Check permission (key)", answer "Authorized" | 2 | proflearning |
| Owning API to IAM `POST /permissions/check`, answer carries `Authorized` and org scope or `allowed` | 5 | reporting 4, staffing 1 |
| Self arrow "Validate permissions" or "Validate: user has key" | 10 | communications 5, documents 4, payments 1 |
| Note saying the caller already authorized | 4 | documents |
| Scope filtering by self arrow or note ("Filter by EPP scope from user context") | 5 | epp 4, organizations 1 (transitive match) |
| Nothing drawn | 92 including IAM's 20 | all other diagrams |

No diagram shows the 5-minute cache of the IAM check. Of 47 answers, 35 are the bare text "Permission confirmed".

### 4.4 Participant naming

| Target | Labels used (count) |
|---|---|
| IAM | IAM 20, Identity & Access API 38, IAM API 6, IAM_API 4, Identity 4, Identity API 1 |
| Credentialing | Application Service 18, Credentialing API 10, Credentials 5, CredentialingAPI 3, Credentials API 1, Credentialing Service 1 |
| PPR | PPR Service 15, Professional Practices API 7, ProfPracticeAPI 1 |
| Communications | Notification 15, Communications Service 10, CommAPI 11, Communications 7, Communications API 7, CommunicationsAPI 4, CommunicationsWorker 4, Notifications 1 |
| Documents | Documents API 10, Document Service 9, DocumentsAPI 2 |
| Staffing | StaffingAPI 12, Staffing API 1, Staffing Service 1, Staffing 2 |
| Organizations | Organizations API 12, Organization Reference Data API 3, Reference Data API 3, EEMAPI 1 |
| EPP | EPP API 26, Educator Prep API 1 |
| Event bus | EventBus 47, Event Bus 39, ServiceBus 6, EventGrid 4 |
| UI | UI 15, EPP UI 21, MiEdWorkforce UI 10, Credentialing Admin UI 9, StaffingUI 8, MiEdWorkforce Portal 6, Credentialing UI 6, ProfLearningUI 5, EPP Admin UI 5, PPR UI 4, Worklist UI 4, and 11 others |

Own-service naming schemes: Application Service (credentialing), `<X> API` with space (documents, epp, organizations, reporting), CamelCase `<X>API` (staffing, proflearning), `<X> Service` (communications, profpractice), bare name (payments "Payments", iam "IAM"). Diagram title format "[Area] - [Flow]" is followed in all 142 diagrams. The "Note over X: State: A > B" convention is used in only 29 diagrams (credentialing 14, epp 9, profpractice 6).

Undeclared participants (12 diagrams, 18 names): iam Citizen Sign-In; payments both Refund diagrams; proflearning (4 diagrams); profpractice Log Non-System Action and Configure PPR Worklist; staffing Assign Employee to Position, Certify Collection, ISD Auditor Review.

### 4.5 External systems versus solution-integrations.md

| Catalog system | In diagrams | Notes |
|---|---|---|
| MiLogin | "MiLogin", participant, 4 iam diagrams | Matches |
| Mi-Key | "Identity Resolution" (5), "Mi-Key API" (2, profpractice), "Mi-Key (external)" (1) | Three names; Staffing never draws it |
| EEM | "EEMAPI" (staffing), "Organization Reference Data API" (epp) | Organizations diagrams call the source "CEPI CEDS JSON-LD API" |
| MSP CHRISS Rap Back | "Rap Back System (MSP)" as actor, "PPR Webhook Endpoint", "CHRISS SOAP API" | Only external declared as actor |
| NASDTEC | "NASDTEC API" | Catalog name is NASDTEC Clearinghouse |
| CEPAS | "CEPAS" (11) plus "File Transfer Service" (2) | Matches; SFTP shown |
| CEPI | "CEPI CEDS JSON-LD API" (1) | Catalog splits CEPI into Transactional Layer, Data Warehouse, MSLDS |
| SendGrid | "SendGrid" (5) | Matches |
| Power BI | "Power BI Service API" (2), "Reporting Engine (Power BI)" (1) | Two names |
| Pearson EdReports, STARR, MSDS TSDL, MiDH, CTEIS, NexSys | not drawn | Pearson and STARR named in text only |

### 4.6 Events and notifications

| Pattern | Count |
|---|---|
| Publish arrow to a bus participant (`--)`) | 183 (event arrows total) |
| Bus drawn calling a consumer | 47 arrows (15 `->>`, 32 `--)`) |
| Staffing "Publish X event" to CommAPI | 15 |
| IAM calls Notification directly | 29, plus 1 to CommAPI |
| Documents calls Notifications directly | 3 |
| Bus to Communications: epp 7, payments 8, proflearning 4 (via CommunicationsWorker), communications 3 | 22 |
| Diagrams where listed events are not drawn | 0 |
| Diagrams drawing an event not listed in text | 2 (iam Cancel Pending Update Request, payments Payment Retry After Failure) |

### 4.7 API call labeling and wiring

- Labeled with verb and path: 352 of 1,413 cross-participant requests (25%). Path style: no `/api` prefix in 8 areas; communications and documents use `/api/...`; payments mixes `/api/v1` (3).
- Arrows or notes naming an auth mechanism: 1 arrow (CEPI OAuth2 client credentials), plus notes for Managed Identity (2), HMAC (1). No arrow carries a kind tag.
- UI fan-out (arrows from a UI to a non-owning service): EPP UI to Credentialing API 6 and EPP Admin UI to Credentialing API 1, to Professional Practices API 2, to Organization Reference Data API 3, to Document Service 2; credentialing UIs to Document Service 3 and Professional Learning API 1; profpractice UIs to Document Service 4; proflearning to Documents API 1 and QuestionSetAPI 1; documents UI to Documents API 14. Other areas (staffing, payments, communications) route through the owning service.
- Webhook and callback treatments: SendGrid (HMAC, separate Webhook Endpoint participant), Rap Back (separate endpoint participant, no auth shown), Defender via Event Grid (main Documents API), Mi-Key callback (main IAM participant).
- Reporting "Generate Embed Token and View Report" is split into three diagrams (12, 7 and 8 arrows) and is the reference for variants.
