# Design Target Architecture

## Description
Design the target cloud-native microservices architecture based on the legacy architecture analysis. This action defines service boundaries, API contracts, data models, and the overall system topology for the modernized banking application.

## Usage
```
/asdm-design-target-architecture
```

## Process
1. Review the legacy architecture analysis results
2. Identify bounded contexts and service boundaries based on business capabilities
3. Define microservice responsibilities and ownership
4. Design API contracts (REST endpoints, request/response schemas)
5. Define data models and database schemas
6. Plan inter-service communication patterns (synchronous REST, async messaging)
7. Design the overall deployment topology

## Output
- Service catalog with responsibilities and boundaries
- API contract definitions (OpenAPI 3.0 specs)
- Data model diagrams and schemas
- Service communication flow diagrams
- Deployment topology diagram

## Related Specifications
See [target-architecture-spec.md](../specs/target-architecture-spec.md) for detailed specifications.
