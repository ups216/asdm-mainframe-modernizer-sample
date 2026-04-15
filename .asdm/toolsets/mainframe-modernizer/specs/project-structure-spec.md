# Project Structure Specification

## Purpose
This document defines the target directory structure and project organization for the modernized banking application in `modern-banking/`.

## Target Directory Structure

```
modern-banking/
├── README.md                              ## Project overview and getting started guide
├── docker-compose.yml                     ## Local development environment
├── pom.xml                                ## Maven parent POM (multi-module)
│
├── backend/                               ## Spring Boot backend services
│   ├── pom.xml                            ## Parent POM for services
│   ├── common/                            ## Shared library module
│   │   ├── pom.xml
│   │   └── src/main/java/com/banking/common/
│   │       ├── config/                    ## Shared configurations
│   │       ├── dto/                       ## Shared DTOs
│   │       ├── exception/                 ## Shared exception classes
│   │       └── util/                      ## Shared utilities
│   │
│   ├── account-service/                   ## Account management microservice
│   │   ├── pom.xml
│   │   ├── src/main/java/com/banking/account/
│   │   │   ├── AccountServiceApplication.java
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── model/
│   │   │   ├── dto/
│   │   │   ├── config/
│   │   │   └── exception/
│   │   ├── src/main/resources/
│   │   │   ├── application.yml
│   │   │   └── db/migration/             ## Flyway/Liquibase migrations
│   │   └── src/test/java/
│   │
│   ├── customer-service/                  ## Customer management microservice
│   │   └── (same structure as account-service)
│   │
│   ├── transaction-service/               ## Transaction processing microservice
│   │   └── (same structure as account-service)
│   │
│   ├── batch-service/                     ## Spring Batch processing service
│   │   └── src/main/java/com/banking/batch/
│   │       ├── BatchServiceApplication.java
│   │       ├── job/                       ## Job configurations
│   │       ├── step/                      ## Step definitions (reader/processor/writer)
│   │       └── config/
│   │
│   └── api-gateway/                       ## API Gateway service
│       └── src/main/java/com/banking/gateway/
│           ├── GatewayApplication.java
│           └── config/
│
├── frontend/                              ## React/Vue frontend application
│   ├── package.json
│   ├── src/
│   │   ├── App.tsx
│   │   ├── pages/                         ## Page components (from BMS maps)
│   │   ├── components/                    ## Shared UI components
│   │   ├── services/                      ## API service modules
│   │   ├── hooks/                         ## Custom React hooks
│   │   ├── types/                         ## TypeScript type definitions
│   │   └── utils/                         ## Utility functions
│   └── public/
│
├── infrastructure/                        ## Deployment and infrastructure
│   ├── docker/                            ## Dockerfiles for each service
│   │   ├── account-service.Dockerfile
│   │   ├── customer-service.Dockerfile
│   │   ├── transaction-service.Dockerfile
│   │   ├── batch-service.Dockerfile
│   │   └── api-gateway.Dockerfile
│   ├── k8s/                               ## Kubernetes manifests
│   │   ├── namespace.yaml
│   │   ├── deployments/
│   │   ├── services/
│   │   ├── configmaps/
│   │   └── ingress.yaml
│   └── ci-cd/                             ## CI/CD pipeline definitions
│       └── github-actions/
│           ├── build.yml
│           ├── test.yml
│           └── deploy.yml
│
└── docs/                                  ## Documentation
    ├── architecture/                       ## Architecture decision records
    ├── api/                               ## OpenAPI specifications
    ├── migration/                         ## Migration mapping documents
    └── runbooks/                          ## Operational runbooks
```

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Java package | `com.banking.{service}` | `com.banking.account` |
| Service module | `{domain}-service` | `account-service` |
| REST endpoint | `/api/v1/{resource}` | `/api/v1/accounts` |
| Database table | `snake_case` | `bank_account` |
| JPA Entity | `PascalCase` | `BankAccount` |
| DTO | `{Entity}Request/Response` | `AccountCreateRequest` |

## Constraints
- Each microservice must be independently buildable and deployable
- Shared code belongs in the `common` module, not duplicated across services
- All configuration must be externalized (application.yml, environment variables)
- Database schemas are owned by their respective services — no cross-service schema access
