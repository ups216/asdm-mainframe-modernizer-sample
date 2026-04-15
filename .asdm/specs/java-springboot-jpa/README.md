# Java Spring Boot JPA Specs

Coding standards and best practices for Java Spring Boot 3 with JPA, following SOLID, DRY, KISS, and YAGNI principles.

## Overview

This spec defines standards for building robust Java Spring Boot applications with JPA. Covers architecture design, entity management, repository patterns, service layer, DTOs, and REST controllers.

## Spec File Index

| File | Description |
|------|-------------|
| [architecture-design.md](./architecture-design.md) | Application logic design and layer responsibilities |
| [entities.md](./entities.md) | Entity class guidelines and annotations |
| [repositories.md](./repositories.md) | Repository/DAO patterns and JPQL best practices |
| [services.md](./services.md) | Service layer design and transaction management |
| [dtos.md](./dtos.md) | Data Transfer Object patterns and validation |
| [rest-controllers.md](./rest-controllers.md) | REST controller design and API routing |
| [response-handling.md](./response-handling.md) | API response format and exception handling |

## Technology Stack

- **Framework**: Java Spring Boot 3 with Maven
- **Java Version**: Java 17
- **Dependencies**: Spring Web, Spring Data JPA, Thymeleaf, Lombok, PostgreSQL driver

## Core Principles

- **SOLID**: Single responsibility, Open-closed, Liskov substitution, Interface segregation, Dependency inversion
- **DRY**: Don't Repeat Yourself
- **KISS**: Keep It Simple, Stupid
- **YAGNI**: You Aren't Gonna Need It
- **OWASP**: Follow security best practices

## Layer Architecture

```
RestController → Service Interface → ServiceImpl → Repository → Entity
       ↓              ↓
    DTOs ←───────────┘
```

## Usage

Reference this spec in AI assistant context:

```
Use java-springboot-jpa spec to create Spring Boot application code
```

## Related Specs

- [reactjs](../reactjs/) - Frontend React.js development
- [nextjs-react-tailwind](../nextjs-react-tailwind/) - Full-stack Next.js development
