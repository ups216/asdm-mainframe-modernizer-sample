# Integration Specification

## Purpose
This document defines the rules and constraints for integrating microservices in the modernized banking application, including API gateway routing, service discovery, and inter-service communication.

## Integration Patterns

### Synchronous Communication
- Service-to-service REST calls via `OpenFeign` or `RestTemplate`
- All inter-service calls routed through the API Gateway in production
- Circuit breaker pattern (Resilience4j) for fault tolerance
- Request timeout: 30s for standard calls, 60s for batch operations

### Asynchronous Communication
- Event-driven patterns for decoupled operations (e.g., transaction notifications)
- Spring Kafka for reliable message delivery
- Event schema follows CloudEvents specification

### Data Consistency
- Database-per-service pattern — no shared databases
- Saga pattern for cross-service transactions
- Eventual consistency for non-critical data propagation
- Outbox pattern for reliable event publishing

## API Gateway Configuration

### Routing Rules
- All external requests enter through the API Gateway
- Path-based routing: `/api/v1/{service}/**` → `{service}-service`
- Rate limiting per service and per client
- JWT token validation at the gateway level

### Security
- OAuth2 / OpenID Connect for authentication
- Role-based access control (RBAC) for authorization
- API keys for service-to-service internal calls
- TLS for all communications

## Service Discovery
- Kubernetes-native service discovery (DNS-based)
- Spring Cloud Kubernetes for Spring Boot integration
- Health check endpoints (`/actuator/health`) for readiness/liveness probes

## Resilience Patterns
| Pattern | Implementation | Purpose |
|---------|---------------|---------|
| Circuit Breaker | Resilience4j | Prevent cascading failures |
| Retry | Spring Retry | Handle transient failures |
| Fallback | Resilience4j | Provide degraded responses |
| Rate Limiting | Gateway filter | Protect services from overload |
| Timeout | HTTP client config | Prevent indefinite waits |

## Constraints
- No synchronous call chains longer than 3 services deep
- All inter-service calls must have timeout configuration
- Idempotent endpoints for retry safety
- Circuit breaker thresholds: 50% failure rate in 10s window
