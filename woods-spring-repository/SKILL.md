---
name: woods-spring-repository
description: >
  Enforces standard conventions for repository interfaces including
  inheritance, naming, annotations, and rules around what repositories
  must not contain.
version: 1.0.0
tags: backend, java, spring, repository, persistence
---

# Repository Guidelines

## Purpose
Repositories provide persistence access for a single entity type.
They must contain queries only — no business logic, no orchestration.

## Required Inheritance

Every repository must extend both:

```java
JpaRepository<Entity, IdType>
JpaSpecificationExecutor<Entity>
```

`JpaSpecificationExecutor` enables dynamic, type-safe querying via Specifications.

## Naming
Name the interface using the entity name followed by `Repository`.

Example: `TopicRepository`, `UserRepository`

## Annotation
Apply `@Repository` where required by project conventions.

## Rules

Repositories should:
- Expose persistence operations only.
- Prefer type-safe APIs over raw queries where possible.
- Use Specifications for dynamic querying.

## Do Not
- Put business logic in repositories.
- Validate business rules in repositories.
- Calculate derived values in repositories.
- Orchestrate multi-step application workflows in repositories.
- Add `@Transactional` to the repository interface without a specific documented reason.
- Add fields to repository interfaces.

## Checklist

When generating or reviewing a repository, verify:

- [ ] Named `<Entity>Repository`.
- [ ] Extends `JpaRepository<Entity, IdType>`.
- [ ] Extends `JpaSpecificationExecutor<Entity>`.
- [ ] `@Repository` annotation is present where required.
- [ ] No business logic is present.
- [ ] No `@Transactional` added without a documented reason.
- [ ] No fields declared on the interface.
