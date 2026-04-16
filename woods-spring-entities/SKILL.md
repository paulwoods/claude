---
name: woods-spring-entities
description: >
  Enforces standard conventions for defining JPA entities including
  annotations, ID generation, equals/hashCode, toString, and documentation.
version: 1.0.0
tags: backend, java, spring, jpa, entity
---

# Entity Guidelines

## Purpose
Entities represent persisted database state only.
They must not be used as transport models or returned from controllers.

## Required Annotations

Apply these annotations to every entity class:

| Annotation             | Purpose                            |
|------------------------|------------------------------------|
| `@Entity`              | Marks class as a JPA entity        |
| `@Table`               | Declares the mapped table          |
| `@Getter`              | Lombok-generated getters           |
| `@Setter`              | Lombok-generated setters           |
| `@NoArgsConstructor`   | Required by JPA                    |
| `@AllArgsConstructor`  | Convenience constructor            |

## ID Field Strategy

Always use sequence-based ID generation.

Required annotations on the ID field:

- `@Id`
- `@SequenceGenerator(...)`
- `@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "...")`

Name the generator after the entity to avoid conflicts across entities.

## Methods

Every entity must implement:

### `equals`
- Use a Hibernate-safe pattern.
- Compare by ID only.
- Handle the case where the ID is `null` (transient entity).
- Use `instanceof` with a proxy-safe check.

### `hashCode`
- Use a Hibernate-safe pattern.
- Return a constant or ID-based hash that is stable before and after persistence.

### `toString`
- Include non-sensitive, non-lazy fields only.
- Use a key-value, comma-separated format.
- Do not include passwords, tokens, secrets, or personal data.
- Do not include fields that trigger lazy loading.

## Documentation
- Add Javadocs at the class level describing the entity's purpose.
- Add Javadocs for any non-trivial methods.

## Testing
- Do not add tests for boilerplate-only entities.
- Add tests only when the entity has meaningful behavior beyond field access.

## Checklist

When generating or reviewing an entity, verify:

- [ ] All required annotations are present.
- [ ] ID field uses sequence-based generation.
- [ ] `equals` is Hibernate-safe.
- [ ] `hashCode` is Hibernate-safe.
- [ ] `toString` excludes sensitive and lazy fields.
- [ ] Class-level Javadoc is present.
- [ ] No transport annotations (e.g., `@JsonProperty`) used for API shaping.
