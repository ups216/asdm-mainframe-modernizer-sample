# Target Architecture Specification

## Purpose
This document defines the rules and constraints for designing the target cloud-native microservices architecture for the modernized banking application.

## Architecture Principles

1. **Service Autonomy** — Each microservice owns its data and logic
2. **API-First** — All service interactions through well-defined REST APIs
3. **Stateless Services** — Services should be stateless; state belongs in databases or caches
4. **Domain-Driven Design** — Service boundaries aligned with business domains
5. **Cloud-Native** — Designed for containerized deployment and horizontal scaling

## Service Design Rules

### Service Boundaries
- Each legacy CICS transaction family maps to one or more API endpoints within a microservice
- Programs that share data structures and form a cohesive business capability should be in the same service
- Cross-service data access must go through APIs, not shared databases

### API Contract Rules
- All APIs must be documented with OpenAPI 3.0 specifications
- Request/response schemas must align with legacy data structures
- HTTP methods follow REST conventions (GET for reads, POST for creates, PUT for updates, DELETE for deletes)
- Error responses use standard HTTP status codes with structured error bodies

### Data Model Rules
- Each service owns its database schema (database-per-service pattern)
- Legacy VSAM files map to relational tables via JPA entities
- Cross-service data consistency uses Saga patterns or eventual consistency

## Target Stack
| Layer | Technology |
|-------|-----------|
| **Runtime** | Java 17+ / Spring Boot 3.x |
| **UI** | React or Vue.js |
| **API** | REST / OpenAPI 3.0 |
| **Data** | Spring Data JPA / PostgreSQL |
| **Batch** | Spring Batch |
| **Messaging** | Spring Kafka / RabbitMQ |
| **Security** | Spring Security / OAuth2 |
| **Build** | Maven |
| **CI/CD** | GitHub Actions |
| **Container** | Docker / Kubernetes |

## Output Format
- Service catalog with domain, responsibilities, and API endpoints
- OpenAPI 3.0 specification files per service
- Database schema definitions (DDL or JPA entity diagrams)
- Deployment topology diagram
- Service communication matrix
