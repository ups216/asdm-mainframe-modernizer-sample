# Java General Purpose Specs

Comprehensive Java development guidelines based on Effective Java, covering best practices for objects, classes, generics, enums, lambdas, streams, methods, exceptions, concurrency, and serialization.

## Overview

This spec provides guidelines for writing high-quality, maintainable Java code following industry best practices and principles.

## AI Developer Profile

**Role**: Senior Java Developer

**Core Principles**:
- **SOLID**: Single responsibility, Open-closed, Liskov substitution, Interface segregation, Dependency inversion
- **DRY**: Don't Repeat Yourself
- **KISS**: Keep It Simple, Stupid
- **YAGNI**: You Aren't Gonna Need It
- **OWASP**: Security best practices
- **DOP**: Data-Oriented Programming
- **FP**: Functional Programming
- **DDD**: Domain-Driven Design

## Technical Stack

| Component | Specification |
|-----------|---------------|
| Build Tool | Maven |
| Java Version | 24 |
| Libraries | Eclipse Collections, Commons Lang3, Guava, VAVR |
| Testing | JUnit 5, JQwik |
| Benchmarking | JMH |
| Language | English (code & comments) |

## Spec File Index

| File | Description |
|------|-------------|
| [creating-objects.md](./creating-objects.md) | Object creation, singletons, builders, and object lifecycle |
| [classes-interfaces.md](./classes-interfaces.md) | Class design, interfaces, composition, and inheritance |
| [generics-enums.md](./generics-enums.md) | Generics best practices and enum/annotation usage |
| [lambdas-streams.md](./lambdas-streams.md) | Lambda expressions and stream operations |
| [methods-api-design.md](./methods-api-design.md) | Method design, parameters, return types, and general programming |
| [exceptions-concurrency.md](./exceptions-concurrency.md) | Exception handling and concurrent programming |
| [serialization.md](./serialization.md) | Serialization alternatives and best practices |
| [programming-paradigms.md](./programming-paradigms.md) | Functional programming and data-oriented programming guidelines |

## Usage

Reference this spec in AI assistant context:

```
Use java-general spec to write Java code
```

## Related Specs

- [java-springboot-jpa](../java-springboot-jpa/) - Spring Boot specific guidelines
