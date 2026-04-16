---
name: woods-spring-service
description: >
  Enforces standard conventions for service classes including annotations,
  dependencies, SmartTable integration, standard method signatures,
  logging patterns, and testing requirements.
version: 1.0.0
tags: backend, java, spring, service, business-logic
---

# Service Guidelines

## Purpose
Services orchestrate persistence, transactions, and application behavior.
Services are the single home for business logic.

## Naming
Name the class using the entity name followed by `Service`.

Example: `TopicService`, `UserService`

## Class Annotations

Apply all of the following:

| Annotation              | Purpose                              |
|-------------------------|--------------------------------------|
| `@Service`              | Marks class as a Spring service      |
| `@RequiredArgsConstructor` | Lombok constructor injection      |
| `@Transactional`        | Default transaction boundary         |
| `@Slf4j`                | Lombok logger                        |

## Typical Dependencies

Inject the following as needed:

- The entity repository
- The entity mapper
- `SecurityService`
- `SmartTableService` when search behavior is required

## SmartTable Columns Field

When the service participates in SmartTable search, declare a `columns` field:

- Use `CopyOnWriteArrayList` as the implementation.
- Initialize with one `SmartColumn` per searchable/sortable entity field.
- Use the field name as the column id.
- Use title case for the label.
- Mark columns as sortable.

## Standard Methods

### `search(Pageable pageable, List<String> filters)`
- Returns: `SmartResult<Entity>`
- Annotation: `@Transactional(readOnly = true)`
- Log (debug): `search: pageable={}, filters={}`
- Delegate to: `smartTableService.search(repository, columns, pageable, filters)`

### `create(Dto dto)`
- Log (debug): `create: dto={}`
- Map DTO to entity using mapper.
- Delegate to `create(Entity entity)`.

### `create(Entity entity)`
- Log (debug): `create: entity={}`
- Call `entity.updateAuditData(securityService.userOrSystem())`.
- Save via repository.
- Return the same entity instance.

### `read(Long id)`
- Returns: `Entity`
- Annotation: `@Transactional(readOnly = true)`
- Log (debug): `read: id={}`
- Load using `repository.findById(id).orElseThrow(...)`.
- Throw `<Entity>NotFoundException` if not found.

### `update(Long id, Dto dto)`
- Returns: `Entity`
- Log (debug): `update: id={}, dto={}`
- Load entity via `read(id)`.
- Apply changes via mapper.
- Save and return updated entity.

### `delete(Long id)`
- Log (debug): `delete: id={}`
- Load entity via `read(id)`.
- Delete via repository.
- Throw `<Entity>NotFoundException` if not found.

## Documentation
- Add class-level Javadoc describing the service's purpose.
- Add Javadoc for any non-obvious methods.

## Testing
- Add unit tests for service behavior.
- Add integration tests when persistence, transaction behavior, or framework wiring must be verified.

## Checklist

When generating or reviewing a service, verify:

- [ ] Named `<Entity>Service`.
- [ ] All four class-level annotations are present.
- [ ] All standard methods are present with correct signatures.
- [ ] Each method logs at debug level with the correct message pattern.
- [ ] `read` throws `<Entity>NotFoundException` when not found.
- [ ] `create(Entity)` calls `updateAuditData` before saving.
- [ ] `search` is annotated `@Transactional(readOnly = true)`.
- [ ] `read` is annotated `@Transactional(readOnly = true)`.
- [ ] SmartTable `columns` field is present when search is required.
- [ ] Class-level Javadoc is present.
- [ ] Unit and integration tests are present.
