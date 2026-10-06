# Authentication & Authorization Patterns

## Overview
This document defines the technical implementation patterns for authentication and authorization across MiEdWorkforce. All services must follow these patterns to ensure consistent security, user experience, and integration with MiLogin OIDC.

**Related Documents:**
- [Technology Standards](../technology-standards.md) - Platform decisions and rationale
- [IAM Domain Documentation](../../solution-areas/iam/iam-domain.md) - Authorization business logic
- [Security Standards](./security-standards.md) - Encryption, secrets management, network security

---

## MiLogin OIDC Integration

### Overview
Brief description of MiLogin as the state's SSO provider, supported user types (citizen, business, worker), and authentication flow (authorization code flow with PKCE).

---

## .NET API Authentication

### Program.cs Configuration
Complete setup for ASP.NET Core authentication middleware, including:
- JWT Bearer authentication configuration
- MiLogin authority and audience settings
- Token validation parameters
- Clock skew handling
- Development vs production configuration differences

### JWT Token Validation
Implementation details for:
- Signature verification against MiLogin public keys
- Issuer validation
- Audience validation
- Expiration and not-before claims validation
- Custom validation logic (if any)

### Claims Extraction and Mapping
How to extract claims from validated JWT and map to internal user identity:
- `sub` claim -> internal user lookup/creation
- `email` -> primary identifier
- `uniqueSecurityName` -> secondary identifier  
- `userType` -> role/permission implications
- Custom claims handling

### Controller Authorization
Patterns for protecting API endpoints:
- `[Authorize]` attribute usage
- Role-based authorization
- Policy-based authorization
- Custom authorization requirements
- Accessing current user information in controllers

### Authorization Check Integration (IAM)
How APIs call IAM service to verify permissions:
- When to check permissions (before business logic)
- HTTP client configuration for IAM API
- Request/response format
- Caching authorization results
- Handling IAM service unavailability (circuit breaker, fallback)
- Error handling and user-friendly messages

---

## React SPA Authentication

### MSAL.js Configuration
Setup for Microsoft Authentication Library for JavaScript:
- MSAL configuration object structure
- MiLogin authority endpoints (dev, qa, staging, prod)
- Client ID configuration per environment
- Redirect URIs configuration
- Scopes requested
- Cache configuration (sessionStorage vs localStorage)

### Login Flow
Implementation of user login:
- Redirect to MiLogin authorization endpoint
- Handling authorization code callback
- Token acquisition and storage
- Silent token refresh
- Handling login errors and user cancellation

### Token Management
Patterns for managing access tokens:
- Where to store tokens (security considerations)
- Automatic token refresh (before expiration)
- Token expiration handling
- Silent authentication vs interactive re-authentication
- Logout and token cleanup

### Protected Routes
How to protect React routes requiring authentication:
- Higher-order component (HOC) pattern
- React Router integration
- Redirecting unauthenticated users to login
- Preserving intended destination after login

### API Request Authentication
Adding authentication headers to API calls:
- Acquiring current access token
- Adding `Authorization: Bearer {token}` header
- Handling 401 Unauthorized responses
- Token refresh on 401
- Request retry after token refresh

### User Context Provider
React context pattern for sharing user identity across components:
- User context structure (user ID, email, display name, roles)
- Context provider setup
- Consuming user context in components
- Loading states during authentication

---

## Authorization Enforcement

### Permission Check Pattern (Frontend)
When and how to check permissions in React:
- Hiding UI elements user cannot access
- Disabling actions user cannot perform
- Querying IAM API for permissions (vs relying on token claims)
- Optimistic UI vs strict enforcement
- Caching permission check results

### Permission Check Pattern (Backend)
Enforcing permissions in .NET APIs:
- Synchronous permission check before command execution
- Calling IAM permission check endpoint
- Request format (userId, permission, organizationCode)
- Handling permission denied (403 Forbidden)
- Logging authorization failures for security audit

### Scope-Based Authorization
How organizational scope affects authorization:
- Entity hierarchy traversal (ISD -> District -> Building)
- Transitive permission resolution (permission at ISD grants access to all child entities)
- Scope validation patterns
- Multi-scope permissions (user authorized for multiple organizations)

---

## Impersonation Support

### Backend Implementation
How admins impersonate users in .NET APIs:
- Impersonation session tracking
- Overriding current user context during impersonation
- Audit logging (who impersonated whom, when, what actions)
- Impersonation token validation
- Ending impersonation session

### Frontend Implementation  
UI patterns for impersonation in React:
- Admin-only impersonation controls
- User context override during active impersonation
- Visual indicators of impersonation mode (banner, user switcher)
- Ending impersonation and returning to admin context
- Restrictions during impersonation (e.g., cannot impersonate another user)

---

## Error Handling

### Authentication Errors
Handling authentication failures:
- Invalid token (401 Unauthorized)
- Expired token (401 Unauthorized + token refresh)
- MiLogin service unavailable (retry with backoff)
- User-friendly error messages
- Logging authentication failures

### Authorization Errors
Handling permission denied scenarios:
- 403 Forbidden responses
- User-friendly error messages ("You do not have permission to...")
- Redirecting to appropriate page (dashboard vs error page)
- Logging authorization failures for security monitoring

---

## Security Considerations

### Token Storage Security
Best practices for token storage:
- Never store tokens in localStorage (XSS vulnerability)
- Use httpOnly cookies when possible
- Session storage for SPAs (cleared on tab close)
- Token encryption at rest (if stored)

### CSRF Protection
Patterns for preventing Cross-Site Request Forgery:
- Anti-forgery tokens for state-changing operations
- SameSite cookie configuration
- Custom request header validation

### CORS Configuration
Cross-Origin Resource Sharing setup:
- Allowed origins per environment
- Allowed methods and headers
- Credentials handling
- Preflight request caching

---

## Testing Patterns

### Unit Testing with Authentication
Mocking authentication in unit tests:
- Mocking `IHttpContextAccessor` for current user
- Mocking IAM permission check client
- Testing authorization logic without external dependencies

### Integration Testing with Authentication
Testing authenticated endpoints:
- Generating test JWT tokens
- Configuring test authentication handler
- Testing different user roles and permissions
- Testing authorization failures

### E2E Testing with Authentication
End-to-end authentication testing:
- Automated login via MiLogin test accounts
- Handling MFA in test environments
- Testing impersonation workflows
- Testing session expiration and refresh

---

## Environment-Specific Configuration

### Development Environment
Special considerations for local development:
- Using MiLogin QA for local/dev
- Test user accounts and credentials
- Bypassing authentication for development (not recommended)
- Mock authentication for isolated component development

### QA/Staging Environments
Configuration for test environments:
- MiLogin QA endpoints
- Test user provisioning
- Separate client IDs per environment
- Testing MFA scenarios

### Production Environment
Production-specific settings:
- MiLogin Production endpoints
- Production client IDs and secrets
- Secret rotation procedures
- Monitoring authentication failures and suspicious activity

---

## Troubleshooting

### Common Authentication Issues
Debugging guide for frequent problems:
- "Invalid token" errors
- Token refresh loops
- Redirect URI mismatches
- CORS errors during authentication
- Claims missing from token

### Logging and Diagnostics
What to log for debugging authentication issues:
- Authentication attempts (success/failure)
- Token validation failures
- Permission check results
- Impersonation session events
- Correlation IDs for distributed tracing

---

## Code Examples

### .NET API: Complete Authentication Setup
[Placeholder for full Program.cs example]

### .NET API: Permission Check in Controller
[Placeholder for controller action with IAM check]

### React: MSAL Configuration
[Placeholder for MSAL setup in App.tsx]

### React: Protected Route Component
[Placeholder for ProtectedRoute HOC]

### React: Authenticated API Call
[Placeholder for API client with auth headers]

---

## Migration Notes

### Transitioning from Legacy Authentication
If migrating from existing authentication system:
- Phased rollout strategy
- Dual authentication support (legacy + MiLogin)
- User migration procedures
- Session transfer/invalidation
- Rollback procedures
