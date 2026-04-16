---
name: woods-spring-controller
description: >
  Enforces standard conventions for REST controllers including annotations,
  dependencies, endpoint structure, logging, and delegation rules.
version: 1.0.0
tags: backend, java, spring, rest, controller
---

# Controller Guidelines

## Purpose
Controllers handle HTTP concerns only.
All application behavior must be delegated to services.

## Naming
Name the class using the entity name followed by `Controller`.

Example: `TopicController`, `UserController`

## Class Annotations

Apply all of the following:

| Annotation                        | Purpose                              |
|-----------------------------------|--------------------------------------|
| `@RestController`                 | Marks class as a REST controller     |
| `@RequestMapping("/resource")`    | Base path — use singular noun        |
| `@Slf4j`                          | Lombok logger                        |
| `@RequiredArgsConstructor`        | Lombok constructor injection         |

Use a **singular** path value in `@RequestMapping` (e.g., `/topic` not `/topics`).

## Dependencies

Inject the following by default:

- The entity service
- The entity mapper
- `LocationService` when building resource locations

Do **not** inject repositories directly into controllers.

## Standard Endpoints

### `search(pageable, filters)` — GET
- Annotation: `@GetMapping`
- Parameters:
  - `Pageable pageable`
  - `@RequestParam(name = "filter", required = false) List<String> filters`
- Returns: `PagedModel<Dto>`
- Rules:
  - Log: `search: pageable={}, filters={}`
  - Call `service.search(pageable, filters)`
  - Extract page from `SmartResult`
  - Map entities to DTOs
  - Return `PagedModel` of DTOs

### `create(dto)` — POST
- Annotation: `@PostMapping`
- Authorization: `@PreAuthorize("hasRole('ROLE_ADMIN')")`
- Parameters: validated request DTO
- Returns: `ResponseEntity<Dto>`
- Rules:
  - Log: `create: dto={}`
  - Call service create method
  - Build location using `LocationService`
  - Return created DTO with location header

### `read(id)` — GET
- Annotation: `@GetMapping("/{id}")`
- Parameters: `@PathVariable Long id`
- Returns: `Dto`
- Rules:
  - Log: `read: id={}`
  - Call service read method
  - Map entity to DTO
  - Let not-found exception propagate

### `update(id, dto)` — PUT
- Annotation: `@PutMapping("/{id}")`
- Authorization: `@PreAuthorize("hasRole('ROLE_ADMIN')")`
- Parameters: path id + validated request DTO
- Returns: `Dto`
- Rules:
  - Log: `update: id={}, dto={}`
  - Call service update method
  - Map updated entity to DTO

### `delete(id)` — DELETE
- Annotation: `@DeleteMapping("/{id}")`
- Status: `@ResponseStatus(HttpStatus.NO_CONTENT)`
- Authorization: `@PreAuthorize("hasRole('ROLE_ADMIN')")`
- Rules:
  - Log: `delete: id={}`
  - Delegate to service
  - Let not-found exception propagate

## Do Not
- Put business rules in controllers.
- Access repositories directly from controllers.
- Perform heavy mapping beyond request/response conversion.
- Return entities directly — always map to DTOs first.

## Checklist

When generating or reviewing a controller, verify:

- [ ] Class is named `<Entity>Controller`.
- [ ] All four class-level annotations are present.
- [ ] Base path uses a singular noun.
- [ ] No repository is injected.
- [ ] Each endpoint logs at the correct level with the correct message pattern.
- [ ] Admin-only endpoints have `@PreAuthorize("hasRole('ROLE_ADMIN')")`.
- [ ] No business logic is present in the controller.
- [ ] All endpoints return DTOs, not entities.
