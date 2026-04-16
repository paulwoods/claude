---
name: woods-spring-architecture
description: >
  Enforces standard backend architecture conventions including package
  layout, layer boundaries, and project-wide coding standards for
  Java/Spring backend services.
version: 1.0.0
---

# Backend Architecture

## Package Layout

Use this package structure for all backend code:

| Package       | Purpose                              |
|---------------|--------------------------------------|
| `entity`      | Database entities                    |
| `dto`         | Transport models                     |
| `mapper`      | Entity/DTO conversion                |
| `repository`  | Persistence interfaces               |
| `exception`   | Exceptions and controller advice     |
| `service`     | Business logic and orchestration     |
| `controller`  | REST controllers                     |
| `model`       | Non-entity domain models             |
| `util`        | Utility classes and enums            |

## Layer Boundaries

Each layer has a single, well-defined responsibility:

- **Entities** are for persistence only.
- **DTOs** are for transport in and out of controllers only.
- **Mappers** convert between entities and DTOs.
- **Repositories** handle persistence access only.
- **Services** orchestrate application behavior and transactions.
- **Controllers** handle HTTP concerns only.

### What this means in practice

- A controller must never return an entity directly — always map to a DTO first.
- A service must never accept or return raw HTTP types (e.g., `HttpServletRequest`, `ResponseEntity`).
- An entity must never implement serialization interfaces intended for transport (e.g., no `@JsonProperty` on entity fields for API shaping).
- A repository must contain no business logic — queries only.
- Mappers must contain no business logic — conversion only.

## Project-Wide Conventions

### Jakarta APIs
- Always use `jakarta.*` imports, never `javax.*`.
- This applies to validation, persistence, servlet, and all other Jakarta namespaces.

### DTO / Entity Separation
- Keep DTOs and entities strictly separate — no shared base classes, no dual-purpose objects.
- Annotate entities with JPA/persistence annotations only.
- Annotate DTOs with validation and serialization annotations only.

### API Design
- Prefer explicit, shallow API models over exposing entity graphs.
- Do not expose nested entity relationships in DTOs unless explicitly required.
- Design DTOs to represent what the API consumer needs, not what the database stores.

### Security
- Keep secrets and environment-specific credentials out of source code.
- Keep secrets and credentials out of documentation.
- Use environment variables or a secrets manager for all sensitive configuration.

## Checklist

When generating or reviewing backend code, verify:

- [ ] New classes are placed in the correct package.
- [ ] Controllers return DTOs, not entities.
- [ ] Services do not reference HTTP types.
- [ ] All imports use `jakarta.*` not `javax.*`.
- [ ] No credentials or secrets appear in code or comments.
- [ ] DTOs and entities are separate with no shared dual-purpose objects.
- [ ] Mapper classes handle all entity ↔ DTO conversion.
- [ ] Repositories contain queries only — no business logic.
