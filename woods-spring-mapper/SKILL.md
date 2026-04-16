---
name: woods-spring-mapper
description: >
  Enforces standard conventions for mapper classes including design
  constraints, null handling, PUT replace semantics, standard methods,
  and testing requirements.
version: 1.0.0
tags: backend, java, spring, mapper, conversion
---

# Mapper Guidelines

## Purpose
Mappers convert between DTOs and entities.

Mappers are also the correct place for:
- Authorization checks
- Business rule enforcement
- Complex domain invariant validation

## Design Constraints

Mappers must always be:
- Stateless
- Deterministic
- Free of mutable internal state
- Free of time-based or random behavior

Mappers must never:
- Call repositories
- Call entity managers
- Load relations from the database

## Naming
Name the class using the entity name followed by `Mapper`.

Example: `TopicMapper`, `UserMapper`

## Annotation
Annotate every mapper with `@Component`.

## Standard Methods

### `createEntity(dto)`
Creates a new entity from a DTO.

Rules:
- Return `null` if `dto` is `null`.
- Instantiate a new entity.
- Map only fields that are allowed to be set on creation.
- Apply required authorization, business rules, and invariant checks.
- Do not attach or load relations from the database.

### `createDto(entity)`
Creates a new DTO from an entity.

Rules:
- Return `null` if `entity` is `null`.
- Instantiate a new DTO.
- Avoid accidental deep graph traversal.
- Keep formatting and presentation concerns out of the mapper.

### `updateEntity(entity, dto)`
Applies DTO values to an existing entity using **PUT replace semantics**.

Rules:
- Return `null` if `entity` is `null`.
- Return `entity` unchanged if `dto` is `null`.
- Mutate and return the same entity instance — do not create a new one.
- Overwrite every updatable field with the DTO value, even if the DTO value is `null`.
- Apply required authorization, business rules, and invariant checks.
- Do not map `id` or audit fields from DTO into the entity.

## Null Handling

All three methods must be null-tolerant:

| Call                          | Expected result          |
|-------------------------------|--------------------------|
| `createEntity(null)`          | Returns `null`           |
| `createDto(null)`             | Returns `null`           |
| `updateEntity(null, dto)`     | Returns `null`           |
| `updateEntity(entity, null)`  | Returns `entity` unchanged |

## Class-Level Documentation

Add a Javadoc comment at the class level stating:
- The mapper uses lenient null handling.
- `updateEntity` uses PUT replace semantics.
- `id` and audit fields are never mapped from DTO.

## Testing

Add unit tests covering:
- Null handling for all three methods.
- Field mapping behavior for `createEntity` and `createDto`.
- PUT replace behavior for `updateEntity`, including `null` DTO field values overwriting entity values.
- Any authorization, business rule, or invariant enforcement logic.

## Checklist

When generating or reviewing a mapper, verify:

- [ ] Named `<Entity>Mapper`.
- [ ] Annotated with `@Component`.
- [ ] All three standard methods are present.
- [ ] All null-handling cases are correctly implemented.
- [ ] `updateEntity` uses PUT replace semantics.
- [ ] `id` and audit fields are not mapped from DTO.
- [ ] No repository or entity manager calls are present.
- [ ] Class-level Javadoc documents null handling and PUT replace semantics.
- [ ] Unit tests cover null handling, field mapping, and business rules.
