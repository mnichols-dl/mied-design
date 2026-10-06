# Testing Standards

## Unit Testing

### What to Test
- Aggregate business logic
- Domain validation rules
- Command/query handlers
- Value object behavior

### What NOT to Test
- Database queries (use integration tests)
- External API calls (use integration tests)
- Framework code (already tested)

### Conventions
- Test class name: `{ClassUnderTest}Tests`
- Test method name: `{MethodUnderTest}_{Scenario}_{ExpectedOutcome}`
- Example: `SubmitApplication_WhenPprBlocked_ThrowsInvalidOperationException`

### Mocking Strategy
- Use Moq for interface dependencies
- Avoid mocking concrete classes
- Mock external service clients (IAM API, PPR API)
- Don't mock value objects or domain entities

### Example
[Code example showing aggregate unit test]

---

## Integration Testing

### What to Test
- API endpoint contracts
- Database queries and updates
- Event publishing/consuming
- External service integration (with mocks)

### Testcontainers Strategy
- SQL Server container for database tests
- Redis container for cache tests
- Azure Service Bus emulator for messaging tests

### Example
[Code example showing API integration test]

---

## Contract Testing (API Spec Validation)

### Runtime Spec Generation
APIs generate OpenAPI spec at runtime from controllers/DTOs.

### Design-Time Spec
Handwritten OpenAPI YAML in source control.

### Validation
CI pipeline compares runtime spec vs design spec:
- Breaking changes fail build
- Non-breaking changes warn

### Example
[Code example showing spec comparison test]

---

## Test Data Management

### Use Builders
Create test data via Builder pattern for readability.

### Example
[Code example showing test data builder]

---

## Performance Testing

### Load Testing Targets
- 5,000 concurrent users (system-wide)
- 100 req/sec per API (sustained)
- Sub-10ms for IAM permission checks

### Tools
- k6 for load testing
- Application Insights for monitoring