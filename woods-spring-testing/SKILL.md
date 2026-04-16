---
name: woods-spring-testing
description: >
  Enforces standard conventions for backend testing including libraries,
  test types, naming conventions, and how to run tests.
version: 1.0.0
tags: backend, java, testing, junit, mockito
---

# Testing Guidelines

## Libraries

Always use the following testing libraries:

| Library  | Purpose                                  |
|----------|------------------------------------------|
| JUnit    | Test runner and assertions framework     |
| AssertJ  | Fluent assertion library                 |
| Mockito  | Mocking and stubbing                     |

Do not introduce additional testing libraries without an explicit decision.

## Test Types

### Unit Tests
- Use for focused behavior in services and mappers.
- Mock all external dependencies.
- Keep tests fast and isolated.
- Do not load Spring context in unit tests.

### Integration Tests
- Use when persistence, transactions, controller wiring, or Spring behavior must be verified.
- Load only the required Spring context slices where possible.
- Prefer real database interactions over mocking repositories in integration tests.

## Naming Conventions

| Test Type        | Naming Pattern       |
|------------------|----------------------|
| Unit test        | `*Test.java` or `*Tests.java` |
| Integration test | `*IT.java`           |

Maven Failsafe picks up integration tests automatically via the `*IT.java` naming convention.

## Running Tests

Run backend unit tests:

```bash
./mvnw clean test
```

Run backend integration tests:

```bash
./mvnw verify
```

Integration tests are separated through Maven Failsafe using the `*IT.java` naming convention.

## What to Test

### Services
- Unit test all standard methods.
- Verify correct delegation to repositories and mappers.
- Integration test persistence behavior and transaction boundaries.

### Mappers
- Unit test null handling for all mapper methods.
- Unit test field mapping correctness.
- Unit test PUT replace semantics on `updateEntity`.
- Unit test authorization and business rule enforcement.

### Controllers
- Integration test endpoint wiring, request validation, and HTTP response codes.
- Do not unit test controllers in isolation unless there is specific logic to verify.

### Entities
- Add tests only when the entity has meaningful behavior beyond boilerplate field access.

### Repositories
- Test custom queries through integration tests against a real or embedded database.

## Checklist

When generating or reviewing tests, verify:

- [ ] Unit tests use JUnit, AssertJ, and Mockito only.
- [ ] Unit tests are named `*Test.java` or `*Tests.java`.
- [ ] Integration tests are named `*IT.java`.
- [ ] Mapper tests cover all null-handling cases.
- [ ] Mapper tests cover PUT replace semantics.
- [ ] Service unit tests mock all dependencies.
- [ ] Integration tests verify real persistence and transaction behavior.
