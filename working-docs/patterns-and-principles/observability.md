## Logging Standards

### Structured Logging (Serilog)
[Code example]

### Log Levels
- Trace: Method entry/exit (disabled in prod)
- Debug: Diagnostics (disabled in prod)
- Information: Business events (state transitions)
- Warning: Recoverable errors (retries, fallbacks)
- Error: Exceptions, failures
- Critical: System-wide failures

### Correlation IDs
[Code example showing correlation ID propagation]

---

## Observability

### Application Insights Configuration
[Code example - Program.cs setup]

### OpenTelemetry Configuration
[Code example - Activity/tracing setup]

### Standard Metrics (All Domains Track)
- Request rate (req/sec by endpoint)
- Request duration (p50, p95, p99)
- Error rate (% 5xx responses)
- Dependency duration (SQL, Service Bus, HTTP)

### Custom Metrics (Domain-Specific)
Documented in each domain's docs. Examples:
- Credentialing: Auto-approval rate, manual review backlog
- Payments: Reconciliation match rate, refund approval time