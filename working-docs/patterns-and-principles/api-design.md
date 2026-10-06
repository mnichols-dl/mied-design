# API Design Standards

**Version:** 1.0  
**Last Updated:** February 2025

---

## Purpose

This document defines REST API design standards for MiEdWorkforce. Following these conventions ensures:
- Consistent developer experience across all domains
- Self-documenting API surfaces
- Easy integration for frontend and external consumers
- Contract-first development with OpenAPI validation

**Scope:** All HTTP APIs in MiEdWorkforce (internal and external). Does NOT apply to:
- Azure Service Bus message formats (see Event Schema Standards)
- Internal gRPC services (if introduced)
- GraphQL endpoints (if introduced)

---

## Core Principles

### 1. Resource-Oriented Design
APIs model domain resources, not RPC-style actions.

**Good:**
```
GET    /applications/{id}          # Get application
POST   /applications                # Create application
DELETE /applications/{id}          # Delete application
POST   /applications/{id}/approve   # Approve application (state transition)
```

**Bad:**
```
POST   /getApplication              # RPC-style, use GET
POST   /createApplication           # RPC-style, use POST /applications
POST   /deleteApplication/{id}      # RPC-style, use DELETE
GET    /approveApplication/{id}     # State change should be POST
```

### 2. Consistent Naming
- **URLs:** lowercase-kebab-case (`/credential-definitions`, not `/CredentialDefinitions` or `/credential_definitions`)
- **JSON properties:** camelCase (`applicantName`, not `ApplicantName` or `applicant_name`)
- **Query parameters:** camelCase (`?pageSize=20`, not `?page_size=20`)

### 3. Contract-First Development
- OpenAPI spec written **before** implementation
- Spec is the **source of truth** for API contract
- Runtime-generated spec validated against design spec in CI/CD
- Breaking changes caught before merge

---

## URL Structure

### Base Pattern
```
https://{host}/{domain}/api/{version}/{resource}
```

**Examples:**
```
Internal:  http://credentialing-api.credentialing.svc.cluster.local/api/v1/applications
External:  https://api.miedworkforce.mi.gov/credentialing/api/v1/applications
```

### Resource Naming

**Use plural nouns for collections:**
```
GET  /applications           # List applications
POST /applications           # Create application
GET  /applications/{id}      # Get specific application
```

**Use singular for singleton resources:**
```
GET  /my/profile             # Current user's profile (always singular)
GET  /system/health          # System health (always singular)
```

**Use sub-resources for relationships:**
```
GET  /applications/{id}/endorsements           # Endorsements for this application
POST /applications/{id}/processing-notes       # Add note to application
GET  /credentials/{id}/endorsements/{eid}      # Specific endorsement on credential
```

**When to flatten vs nest:**
- **Nest** when the child resource cannot exist without the parent
  - `/applications/{id}/processing-notes` (note belongs to application)
- **Flatten** when the child has independent existence
  - `/endorsement-definitions` (not `/credentials/{id}/endorsement-definitions`)

### Action Resources (State Transitions)

For operations that don't fit REST verbs, use action sub-resources:

```
POST /applications/{id}/approve      # Approve application
POST /applications/{id}/deny         # Deny application
POST /credentials/{id}/suspend       # Suspend credential
POST /credentials/{id}/reinstate     # Reinstate credential
POST /applications/{id}/on-hold      # Place on hold
```

**When to use action resources:**
- State transitions that aren't simple CRUD (`approve`, `deny`, `suspend`)
- Operations requiring additional parameters beyond the resource itself
- Operations that have side effects beyond updating the resource

**Naming convention:**
- Verb in imperative form (`approve`, not `approval` or `approving`)
- Use hyphens for multi-word actions (`on-hold`, `re-review`)

---

## HTTP Methods

### Standard CRUD Operations

| Method | Purpose | Idempotent? | Safe? | Request Body | Success Code |
|--------|---------|-------------|-------|--------------|--------------|
| **GET** | Retrieve resource(s) | Yes | Yes | No | 200 |
| **POST** | Create resource | No | No | Yes | 201 |
| **PUT** | Replace entire resource | Yes | No | Yes | 200 |
| **PATCH** | Partial update | No | No | Yes | 200 |
| **DELETE** | Remove resource | Yes | No | No | 204 |

### Usage Guidelines

#### **GET - Retrieve Resources**

**List collection:**
```http
GET /applications?status=Submitted&pageSize=20
```

**Get single resource:**
```http
GET /applications/3fa85f64-5717-4562-b3fc-2c963f66afa6
```

**Response:** 200 OK with resource(s) in body, or 404 if not found

**Never use GET for:**
- State changes (use POST for actions)
- Operations with side effects

#### **POST - Create or Trigger Actions**

**Create resource:**
```http
POST /applications
Content-Type: application/json

{
  "applicationType": "New",
  "credentialDefinitionId": "...",
  "applicantUniqueId": "MI123456789"
}
```

**Response:** 201 Created with `Location` header and resource in body
```http
HTTP/1.1 201 Created
Location: /api/v1/applications/3fa85f64-5717-4562-b3fc-2c963f66afa6
Content-Type: application/json

{
  "applicationId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "Draft",
  ...
}
```

**Trigger action:**
```http
POST /applications/3fa85f64-5717-4562-b3fc-2c963f66afa6/approve
Content-Type: application/json

{
  "processorNotes": "All requirements met"
}
```

**Response:** 200 OK with updated resource state

#### **PUT - Replace Entire Resource**

**Use sparingly.** Prefer PATCH for partial updates.

```http
PUT /credential-definitions/3fa85f64-5717-4562-b3fc-2c963f66afa6
Content-Type: application/json

{
  "printName": "Professional Teaching Certificate",
  "category": "Teacher",
  "isPermanent": false,
  "canApply": true,
  "canRenew": true,
  "validityPeriodYears": 5,
  "feeAmount": 100.00
}
```

**Response:** 200 OK with updated resource

**Important:** Client must send **all** fields. Missing fields are treated as null/default.

#### **PATCH - Partial Update**

**Preferred for updates.** Only send changed fields.

```http
PATCH /applications/3fa85f64-5717-4562-b3fc-2c963f66afa6
Content-Type: application/json

{
  "status": "PendingDocuments"
}
```

**Response:** 200 OK with updated resource

**Format:** JSON Merge Patch (RFC 7396) - simple and sufficient for our needs

#### **DELETE - Remove Resource**

```http
DELETE /applications/3fa85f64-5717-4562-b3fc-2c963f66afa6
```

**Response:** 204 No Content (empty body)

**Important:** Most deletes are **soft deletes** (set `is_deleted = true`). Hard deletes are rare.

---

## HTTP Status Codes

### Success Codes (2xx)

| Code | Meaning | When to Use |
|------|---------|-------------|
| **200 OK** | Request succeeded | GET, PUT, PATCH, POST (actions) |
| **201 Created** | Resource created | POST (create) |
| **204 No Content** | Success, no body | DELETE |

### Client Error Codes (4xx)

| Code | Meaning | When to Use | Example |
|------|---------|-------------|---------|
| **400 Bad Request** | Malformed request | Invalid JSON, missing required fields, type mismatch | `{"error": "Invalid JSON in request body"}` |
| **401 Unauthorized** | Authentication required | Missing or invalid JWT | `{"error": "Authentication required"}` |
| **403 Forbidden** | Permission denied | Valid auth, insufficient permissions | `{"error": "Insufficient permissions to approve applications"}` |
| **404 Not Found** | Resource doesn't exist | Requested ID not in database | `{"error": "Application not found"}` |
| **409 Conflict** | Resource state conflict | Delete submitted application, duplicate resource | `{"error": "Cannot delete - application already submitted"}` |
| **422 Unprocessable Entity** | Business rule violation | Valid format, but business logic rejects | `{"error": "PPR clearance required before approval"}` |
| **429 Too Many Requests** | Rate limit exceeded | Istio or APIM rate limiting | `{"error": "Rate limit exceeded", "retryAfter": 60}` |

**400 vs 422 distinction:**
- **400:** Request doesn't match API contract (schema validation failed)
- **422:** Request matches contract but violates business rules

**Examples:**
```json
// 400 - Invalid request format
{
  "applicationType": "InvalidType"  // Not in enum [New, Renewal, Endorsement, Permit]
}

// 422 - Valid format, business rule violation
{
  "applicationType": "Renewal",
  "credentialId": "..." // Valid GUID, but credential not eligible for renewal
}
```

### Server Error Codes (5xx)

| Code | Meaning | When to Use |
|------|---------|-------------|
| **500 Internal Server Error** | Unhandled exception | Database failure, unhandled exception |
| **503 Service Unavailable** | Dependency unavailable | Circuit breaker open, database connection pool exhausted |

**Important:** 5xx errors trigger alerts. Minimize by handling errors gracefully.

---

## Error Response Format (RFC 7807)

All error responses use [RFC 7807 Problem Details](https://www.rfc-editor.org/rfc/rfc7807) format.

### Standard Structure

```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more fields failed validation",
  "instance": "/api/v1/applications",
  "traceId": "00-4bf92f3577b34da6a3ce929d0e0e4736-00",
  "errors": [
    {
      "field": "applicationType",
      "code": "CRED_VAL_INVALID_ENUM",
      "message": "Value must be one of: New, Renewal, EndorsementAddition, Permit"
    },
    {
      "field": "credentialDefinitionId",
      "code": "CRED_VAL_REQUIRED_FIELD",
      "message": "This field is required"
    }
  ]
}
```

### Field Definitions

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | URI reference identifying the error type (for documentation) |
| `title` | Yes | Human-readable summary (same for all instances of this error type) |
| `status` | Yes | HTTP status code (for convenience) |
| `detail` | No | Specific explanation for this instance |
| `instance` | Yes | URI reference to the request that caused the error |
| `traceId` | Yes | W3C Trace Context trace ID for distributed tracing |
| `errors` | No | Array of validation errors (for 400/422 responses) |

### Error Codes

Format: `{DOMAIN}_{CATEGORY}_{SPECIFIC}`

**Categories:**
- `VAL` - Validation errors (malformed input)
- `BIZ` - Business rule violations
- `AUTH` - Authorization/permission errors
- `SYS` - System/infrastructure errors

**Examples:**
```
CRED_VAL_INVALID_GRADE_RANGE       # Validation: low grade > high grade
CRED_VAL_REQUIRED_FIELD            # Validation: missing required field
CRED_BIZ_PPR_CLEARANCE_REQUIRED    # Business: PPR status not cleared
CRED_BIZ_DUPLICATE_APPLICATION     # Business: active application exists
CRED_AUTH_INSUFFICIENT_PERMISSION  # Authorization: user lacks permission
IAM_AUTH_SCOPE_MISMATCH            # Authorization: wrong organization scope
```

**Domain-Specific Error Codes:** Documented in each domain's documentation (e.g., `credentialing-domain.md#error-codes`)

### Example Responses

**400 - Validation Failure:**
```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Request contains invalid or missing fields",
  "instance": "/api/v1/applications",
  "traceId": "00-abc123-00",
  "errors": [
    {
      "field": "gradeBand.lowGrade",
      "code": "CRED_VAL_INVALID_GRADE_RANGE",
      "message": "Low grade must be less than or equal to high grade"
    }
  ]
}
```

**403 - Permission Denied:**
```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/forbidden",
  "title": "Forbidden",
  "status": 403,
  "detail": "User lacks permission to approve applications",
  "instance": "/api/v1/applications/abc-123/approve",
  "traceId": "00-def456-00"
}
```

**422 - Business Rule Violation:**
```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/business-rule-violation",
  "title": "Business Rule Violation",
  "status": 422,
  "detail": "Application cannot be approved due to business rule violations",
  "instance": "/api/v1/applications/abc-123/approve",
  "traceId": "00-ghi789-00",
  "errors": [
    {
      "field": null,
      "code": "CRED_BIZ_PPR_CLEARANCE_REQUIRED",
      "message": "Professional Practice Review clearance required before approval"
    },
    {
      "field": "assessmentResults",
      "code": "CRED_BIZ_ASSESSMENT_SCORES_INSUFFICIENT",
      "message": "Assessment score of 215 does not meet minimum passing score of 220"
    }
  ]
}
```

**409 - Conflict:**
```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/conflict",
  "title": "Conflict",
  "status": 409,
  "detail": "Cannot delete application that has already been submitted",
  "instance": "/api/v1/applications/abc-123",
  "traceId": "00-jkl012-00"
}
```

**503 - Service Unavailable:**
```json
{
  "type": "https://docs.miedworkforce.mi.gov/errors/service-unavailable",
  "title": "Service Unavailable",
  "status": 503,
  "detail": "Professional Practice Review service is currently unavailable",
  "instance": "/api/v1/applications",
  "traceId": "00-mno345-00"
}
```

---

## Pagination

### Query Parameters

All list endpoints support pagination with these query parameters:

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| `page` | integer | 1 | - | Page number (1-indexed) |
| `pageSize` | integer | 20 | 100 | Items per page |

**Example:**
```http
GET /applications?page=2&pageSize=50
```

### Response Format

Paginated responses include metadata:

```json
{
  "items": [
    { "applicationId": "...", "status": "Submitted", ... },
    { "applicationId": "...", "status": "Approved", ... }
  ],
  "pagination": {
    "page": 2,
    "pageSize": 50,
    "totalCount": 237,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPreviousPage": true
  }
}
```

### Alternative: Cursor-Based Pagination (Future)

For real-time data streams (e.g., event logs), consider cursor-based pagination:

```http
GET /events?cursor=abc123&pageSize=50

Response:
{
  "items": [...],
  "nextCursor": "def456",
  "hasMore": true
}
```

**When to use:**
- Data changes frequently (new items added while paginating)
- Need stable pagination (page numbers shift as data changes)

**For now:** Stick with offset-based (page number) pagination. Introduce cursor-based if needed.

---

## Filtering, Sorting, Searching

### Filtering

Use query parameters matching resource field names:

```http
GET /applications?status=Submitted&applicationType=New
GET /credentials?expiringBefore=2025-12-31
GET /applications?submittedAfter=2025-01-01&submittedBefore=2025-01-31
```

**Conventions:**
- Equality: `?status=Submitted`
- Comparison: `?expiringBefore=2025-12-31`, `?submittedAfter=2025-01-01`
- Multiple values: `?status=Submitted,Approved` (comma-separated OR)
- Prefix: `?name=John*` (if supported, document clearly)

### Sorting

Use `sortBy` and `sortOrder` query parameters:

```http
GET /applications?sortBy=submittedAt&sortOrder=desc
```

**Conventions:**
- `sortBy`: Field name in camelCase
- `sortOrder`: `asc` or `desc` (default: `asc`)
- Multiple sort fields: `sortBy=status,submittedAt` (comma-separated, priority order)

### Searching

For full-text search, use `search` or `q` parameter:

```http
GET /applications?search=John+Doe
GET /credential-definitions?q=mathematics
```

**Search behavior:**
- Searches across multiple text fields (name, description, etc.)
- Case-insensitive
- Partial match (contains)
- Document which fields are searched in API docs

---

## Versioning

### URL-Based Versioning

Version embedded in URL path:

```
https://api.miedworkforce.mi.gov/credentialing/api/v1/applications
                                                      ^^
```

**Version format:** `v{major}` (no minor version in URL)

**When to increment:**
- **v1 -> v2:** Breaking changes (removed fields, changed behavior, incompatible contracts)
- **v1 stays v1:** Additive changes (new optional fields, new endpoints)

**Lifecycle:**
- Old versions supported for **1 year** after new version released
- Deprecation warnings in response headers 6 months before sunset:
  ```http
  Deprecation: Sun, 01 Jun 2025 00:00:00 GMT
  Link: </api/v2/applications>; rel="successor-version"
  ```

### What's a Breaking Change?

**Breaking (requires new version):**
- Remove endpoint or field
- Rename field
- Change field type (`string` -> `int`)
- Make optional field required
- Change error codes or status codes
- Change authentication/authorization requirements

**Non-Breaking (stays in same version):**
- Add new endpoint
- Add new optional field
- Add new enum value (if clients handle unknown values gracefully)
- Deprecate field (marked in docs, still returned)
- Improve error messages

---

## Content Negotiation

### Request Content Type

**For requests with body:** Include `Content-Type` header

```http
POST /applications
Content-Type: application/json

{ "applicationType": "New", ... }
```

**Supported:** `application/json` only

**Unsupported:** `application/xml`, `application/x-www-form-urlencoded` (except for OAuth flows)

### Response Content Type

**Default:** `application/json`

**Client can request different format via `Accept` header:**
```http
GET /applications
Accept: application/json
```

**If unsupported format requested:** Return `406 Not Acceptable`

**File downloads:** Use appropriate MIME type
```http
GET /credentials/abc-123/certificate

Response:
Content-Type: application/pdf
Content-Disposition: attachment; filename="certificate-abc-123.pdf"
```

---

## Request/Response Headers

### Standard Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes* | Bearer token from MiLogin (`Bearer {jwt}`) |
| `Content-Type` | Yes** | `application/json` for requests with body |
| `X-Correlation-Id` | No | Client-provided correlation ID (for tracing) |

\* Except for public endpoints (e.g., `/health`)  
** Only for POST, PUT, PATCH requests

### Standard Response Headers

| Header | Always Included | Description |
|--------|-----------------|-------------|
| `Content-Type` | Yes | `application/json` (or appropriate for content) |
| `X-Correlation-Id` | Yes | Echo client's correlation ID, or generate if not provided |
| `X-Trace-Id` | Yes | W3C Trace Context trace ID (for distributed tracing) |
| `Location` | 201 only | URI of created resource |

### Security Headers

All responses include:

```http
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
```

**Set by:** Istio Envoy sidecar (no application code needed)

---

## OpenAPI Contract-First Development

### Workflow

1. **Design API:** Write OpenAPI spec (`credentialing-api.yaml`) **before** coding
2. **Review spec:** Team reviews API design (PR review)
3. **Generate stubs:** (Optional) Generate server stubs and client SDKs from spec
4. **Implement:** Backend implements endpoints, frontend consumes
5. **Validate:** CI/CD compares runtime spec vs design spec

### CI/CD Validation

**Build pipeline:**
```yaml
- name: Generate runtime OpenAPI spec
  run: dotnet test --filter "Category=ApiSpec" --logger "trx;LogFileName=api-spec.trx"
  
- name: Compare runtime spec vs design spec
  run: |
    dotnet tool install -g Swashbuckle.AspNetCore.Cli
    swagger tofile --output runtime-spec.json bin/Release/net8.0/CredentialingAPI.dll v1
    npx openapi-diff design/credentialing-api.yaml runtime-spec.json --fail-on breaking
```

**Breaking changes:** Fail build  
**Non-breaking changes:** Warning (reviewer decides)

### Design Spec Location

```
src/
├── Credentialing.API/
│   ├── Controllers/
│   ├── Program.cs
│   └── openapi/
│       └── credentialing-api.yaml    # Design-time spec (source of truth)
└── Credentialing.Tests/
    └── ApiSpecTests.cs                 # Runtime vs design validation
```

### OpenAPI Spec Template

**Minimal example:**
```yaml
openapi: 3.0.3
info:
  title: MiEdWorkforce Credentialing API
  version: 1.0.0
  description: Manages educator credential lifecycle

servers:
  - url: http://credentialing-api.credentialing.svc.cluster.local/api/v1
    description: Internal AKS
  - url: https://api.miedworkforce.mi.gov/credentialing/api/v1
    description: External via APIM

paths:
  /applications:
    post:
      summary: Submit credential application
      operationId: submitApplication
      tags: [Applications]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ApplicationCreate'
            examples:
              newTeachingCertificate:
                summary: New teaching certificate
                value:
                  applicationType: "New"
                  credentialDefinitionId: "3fa85f64-5717-4562-b3fc-2c963f66afa6"
                  applicantUniqueId: "MI123456789"
      responses:
        '201':
          description: Application created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Application'
        '400':
          $ref: '#/components/responses/BadRequest'

components:
  schemas:
    ApplicationCreate:
      type: object
      required: [applicationType, credentialDefinitionId, applicantUniqueId]
      properties:
        applicationType:
          type: string
          enum: [New, Renewal, EndorsementAddition, Permit]
        credentialDefinitionId:
          type: string
          format: uuid
        applicantUniqueId:
          type: string
          
  responses:
    BadRequest:
      description: Validation failed
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
            
  securitySchemes:
    MiLogin:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://milogin.michigan.gov/oauth/authorize
          tokenUrl: https://milogin.michigan.gov/oauth/token
          scopes:
            credentialing:read: Read credential data
            credentialing:write: Submit applications

security:
  - MiLogin: [credentialing:read]
```

**Always include:**
- **Examples:** Request/response examples for every endpoint (happy path + errors)
- **Descriptions:** Human-readable descriptions for complex fields
- **Validation:** Required fields, formats, enums, min/max

---

## Additional Guidelines

### Date/Time Formats

**ISO 8601 format:**
- **Date only:** `2025-02-13`
- **DateTime (UTC):** `2025-02-13T14:30:00Z`
- **DateTime (with timezone):** `2025-02-13T14:30:00-05:00`

**In JSON:**
```json
{
  "submittedAt": "2025-02-13T14:30:00Z",
  "expiresAt": "2030-06-30"  // Date only (no time component)
}
```

**Never use:** Unix timestamps, non-ISO formats

### GUIDs/UUIDs

**Format:** Standard UUID (lowercase, with hyphens)
```json
{
  "applicationId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Never use:** Uppercase, no hyphens, or custom formats

### Enums

**String enums preferred:**
```json
{
  "status": "Submitted"  // Not 1, "SUBMITTED", or "submitted"
}
```

**Case:** PascalCase for enum values (`Submitted`, not `submitted` or `SUBMITTED`)

**Handling unknown values:**
- Clients should gracefully handle unknown enum values (forward compatibility)
- Document all valid values in OpenAPI spec

### Null vs Omitted Fields

**In responses:**
- **Omit** fields with no value (don't include `"field": null`)
- **Exception:** Nullable fields that are explicitly set to null

**In requests:**
- **Omit** optional fields (don't send `"field": null`)
- **Send `null`** to explicitly clear a value (PATCH only)

**Example:**
```json
// Good response
{
  "applicationId": "...",
  "status": "Submitted",
  "approvedAt": "2025-02-13T14:30:00Z"
  // submittedAt omitted (not yet submitted)
}

// Bad response
{
  "applicationId": "...",
  "status": "Submitted",
  "approvedAt": "2025-02-13T14:30:00Z",
  "submittedAt": null  // Don't include null fields
}
```

### Boolean Fields

**Use `true`/`false` (not `1`/`0`, `"true"`/`"false"`)**
```json
{
  "isPermanent": true,
  "canRenew": false
}
```

---

## References

**Related Standards:**
- [Error Handling Standards](./cross-cutting-concerns.md#error-handling)
- [Authentication Standards](./cross-cutting-concerns.md#authentication)
- [Logging Standards](./cross-cutting-concerns.md#logging)

**External Standards:**
- [RFC 7807 - Problem Details](https://www.rfc-editor.org/rfc/rfc7807)
- [RFC 7396 - JSON Merge Patch](https://www.rfc-editor.org/rfc/rfc7396)
- [OpenAPI Specification 3.0](https://spec.openapis.org/oas/v3.0.3)
- [REST API Design Best Practices (Microsoft)](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
