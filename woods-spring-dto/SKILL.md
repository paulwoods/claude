---
name: woods-spring-dto
description: >
  Enforces standard conventions for Data Transfer Objects including
  structure, naming, field design, validation, and rules around
  what DTOs must not do.
version: 1.0.0
tags: backend, java, spring, dto, transport
---

# DTO Guidelines

## Purpose
DTOs are transport models for requests and responses only.
They must not contain business logic or persistence concerns.

## Structure
Use a Java `record` for all DTOs.

```java
public record TopicDto(Long id, String name) {}
```

## Naming
Name the record using the entity name followed by `Dto`.

Examples: `TopicDto`, `UserDto`, `AboutDto`

## Field Design

Prefer flat scalar fields:

| Type         | Use for                        |
|--------------|--------------------------------|
| `Long`       | IDs and counts                 |
| `String`     | Text values                    |
| `Integer`    | Numeric values                 |
| `BigDecimal` | Monetary or precise decimals   |
| `Boolean`    | Flags                          |
| `UUID`       | Unique identifiers             |
| `Instant`    | Timestamps                     |

## Relationship Design
- Do not embed full entity graphs in DTOs.
- Prefer `relatedId` fields (e.g., `ownerId`, `categoryId`).
- Use shallow nested DTOs only when the API contract explicitly requires it.

## Collections
- Prefer `List<T>` over arrays.

## Validation
Use Jakarta Bean Validation annotations to enforce request shape:

| Annotation    | Use for                         |
|---------------|---------------------------------|
| `@NotBlank`   | Required string fields          |
| `@NotNull`    | Required non-string fields      |
| `@Size`       | Length and collection size      |

Keep validation focused on:
- Required fields
- Length constraints
- Format constraints
- Basic shape constraints

Do **not** place domain or business rules in DTOs.

## General Rules
- Include `id` in response DTOs when needed by the consumer.
- DTO fields may be nullable unless the API contract requires otherwise.
- Do not hide missing values with smart defaults.
- Do not add methods to DTOs.

## Do Not
- Add JPA annotations to DTOs.
- Put business logic in DTOs.
- Inject or call repositories or services from DTOs.
- Use entities as request or response models.
- Mirror full entity graphs unless the API contract explicitly requires it.

## Checklist

When generating or reviewing a DTO, verify:

- [ ] Implemented as a Java `record`.
- [ ] Named `<Entity>Dto`.
- [ ] Fields are flat scalar types where possible.
- [ ] No full entity graphs embedded.
- [ ] Validation annotations are present on required request fields.
- [ ] No business logic present.
- [ ] No JPA annotations present.
- [ ] No methods added to the record.

