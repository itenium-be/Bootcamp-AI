# Architecture

The system is a **modular monolith**: a single deployable unit divided into cohesive modules with explicit boundaries. Modules communicate through well-defined interfaces (ports); never by directly referencing each other's internals. A module maps to a bounded context.

## Default: Clean (Onion) Architecture

Apply when domain logic is non-trivial:

- **Domain layer** (center): entities, value objects, aggregates, domain events, domain services. Zero dependencies on anything outside this layer.
- **Application layer**: use cases / command & query handlers (CQRS). Orchestrates domain objects. No infrastructure dependencies — depends on domain interfaces only.
- **Infrastructure layer**: persistence, messaging, external APIs. Implements interfaces defined inward. Never referenced by domain or application.
- **Presentation layer**: API controllers, CLI. Thin — validates input, delegates to application layer, maps response.
- Dependencies point **inward only**. Enforce with architecture tests (e.g., ArchUnit, NetArchTest).

## Alternative: Vertical Slice Architecture

Prefer vertical slices when a feature is CRUD-heavy with little shared domain logic and minimal interaction with other domain concepts. Each slice owns its request, handler, validation, and response — fully self-contained. When logic is shared across slices, extract it to the domain layer.

## Domain-Driven Design

- Use **ubiquitous language** everywhere: class names, methods, variables, and tests must use domain terminology, never technical jargon.
- **Bounded contexts** align with module boundaries. Each module owns its domain model. Shared concepts are translated at the boundary via an anti-corruption layer — never leaked directly.
- **Aggregates** protect invariants. Only the aggregate root is reachable from outside; enforce consistency within the aggregate boundary.
- **Value objects** for concepts defined by value, not identity (e.g., `Money`, `Email`, `DateRange`). Make them immutable.
- **Domain events** for cross-context communication. Never call another bounded context's internals directly.
