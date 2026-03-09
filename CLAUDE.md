# SkillForge

## Before running backend
docker compose up -d

## Package manager
Use bun, not npm/yarn

## Before committing
bun run lint && bun run typecheck && bun run test
dotnet format && dotnet test

## Architecture

The system is a **modular monolith**: a single deployable unit divided into cohesive modules with explicit boundaries. Modules communicate through well-defined interfaces (ports); never by directly referencing each other's internals. A module maps to a bounded context.

### Default: Clean (Onion) Architecture

Apply when domain logic is non-trivial:

- **Domain layer** (center): entities, value objects, aggregates, domain events, domain services. Zero dependencies on anything outside this layer.
- **Application layer**: use cases / command & query handlers (CQRS). Orchestrates domain objects. No infrastructure dependencies — depends on domain interfaces only.
- **Infrastructure layer**: persistence, messaging, external APIs. Implements interfaces defined inward. Never referenced by domain or application.
- **Presentation layer**: API controllers, CLI. Thin — validates input, delegates to application layer, maps response.
- Dependencies point **inward only**. Enforce with architecture tests (e.g., ArchUnit, NetArchTest).

### Alternative: Vertical Slice Architecture

Prefer vertical slices when a feature is CRUD-heavy with little shared domain logic and minimal interaction with other domain concepts. Each slice owns its request, handler, validation, and response — fully self-contained. When logic is shared across slices, extract it to the domain layer.

### Domain-Driven Design

- Use **ubiquitous language** everywhere: class names, methods, variables, and tests must use domain terminology, never technical jargon.
- **Bounded contexts** align with module boundaries. Each module owns its domain model. Shared concepts are translated at the boundary via an anti-corruption layer — never leaked directly.
- **Aggregates** protect invariants. Only the aggregate root is reachable from outside; enforce consistency within the aggregate boundary.
- **Value objects** for concepts defined by value, not identity (e.g., `Money`, `Email`, `DateRange`). Make them immutable.
- **Domain events** for cross-context communication. Never call another bounded context's internals directly.

## Code Quality

- Write **human-readable code**: intention-revealing names, small focused functions, no clever tricks. Code is read far more than it is written.
- **DRY and SOLID apply strictly to production code.** Tests may duplicate setup, data, and assertions to remain readable and self-contained — never sacrifice test clarity for abstraction.
- Prefer **explicit over implicit**: no magic, no hidden conventions, no surprising side effects.
- Keep the **domain model pure**: no framework annotations, no ORM attributes, no HTTP concerns inside domain or application layers.

## Test-Driven Development

Work **one test at a time** through the red-green-refactor cycle:

1. **Red** — write one failing test for a single acceptance criterion. Run tests; confirm it fails for the right reason (not a compile error or wrong assertion).
2. **Green** — write the minimum code to make it pass. Fake it (hardcode a return value) if that suffices; only generalize when a new test forces it. Run tests; confirm green.
3. **Refactor** — improve structure and clarity without changing behaviour. Run tests; confirm still green.
4. Repeat for the next requirement.

Hard rules:
- Never write production code without a currently failing test that requires it.
- Never have more than one failing test at a time.
- Never refactor while any test is red.
- Each test validates exactly one acceptance criterion — name it accordingly.

For new features or non-trivial changes, always invoke the `tdd-guide` skill to drive the cycle. Skip only for trivial changes (typos, config, renaming).

## Frontend (React)

### React Version & Patterns

- Before implementing any React feature, consult **[react.dev](https://react.dev)** to verify the current recommended approach. Use `WebFetch` on react.dev when in doubt — never rely solely on training data for React APIs.
- Use **React Server Components (RSC)** for data fetching and server-side logic by default. Mark components `"use client"` only when interactivity or browser APIs require it.
- Use **hooks exclusively** — no class components, no legacy lifecycle methods (`componentDidMount`, `componentDidUpdate`, etc.).
- Prefer modern built-in hooks: `use()`, `useActionState`, `useOptimistic`, `useFormStatus`. Avoid legacy workarounds they replace.
- Never use patterns marked deprecated on react.dev. When a new major React version is released, migrate deprecated patterns proactively — do not leave legacy code in place.
- Avoid `useEffect` for data fetching; use RSC or Suspense-compatible data fetching instead.

### Component Design

- **Composition over configuration**: small, single-purpose components composed together. Avoid large all-in-one components.
- Keep components **pure**: no side effects during render. Side effects belong in event handlers or `useEffect` (used sparingly and only when no RSC alternative exists).
- Colocate state as close as possible to where it is used. Lift only when siblings genuinely share it.
- The React Compiler handles most memoization — do not manually add `useMemo`/`useCallback`/`React.memo` unless profiling proves it necessary.

### State Management

- **Local state first** (`useState`, `useReducer`). Lift to shared context only when multiple components genuinely need it.
- Use `useContext` for low-frequency global state (theme, current user). For high-frequency updates, use a lightweight store (Zustand or Jotai).
- **Server state belongs server-side**: RSC or TanStack Query for data from APIs — not in `useState` + `useEffect`.
- Avoid Redux unless it is already established in the project and the complexity justifies it.

### Accessibility

- Use **semantic HTML** elements (`<button>`, `<nav>`, `<main>`, `<article>`, `<section>`). Never use a `<div>` or `<span>` where a semantic element exists.
- Every interactive element must be keyboard-navigable and have an accessible label (`aria-label` or visible text).
- Use ARIA attributes only when semantic HTML is insufficient — ARIA does not fix non-semantic markup.
- Accessibility violations are bugs. Include axe-core checks in the E2E suite.

### TDD for Frontend

Apply the same red-green-refactor discipline as backend:

- Use **React Testing Library + Vitest** for component and hook tests.
- Test **behaviour, not implementation**: query by role, label, or text (`getByRole`, `getByLabelText`, `getByText`) — never by CSS class or component internals.
- Never assert on internal state directly. If you cannot test it through the UI, the design needs reconsideration.
- Mock at the **network boundary using MSW** (Mock Service Worker), not at the module level, to keep tests realistic.
- One failing test at a time — same hard rules as backend TDD apply.

### E2E Testing with Playwright

- E2E tests cover **complete user journeys**: happy path + critical error paths. Individual component behaviour belongs in unit tests, not E2E.
- Use **Page Object Model (POM)** to encapsulate selectors and interactions. Tests read as user stories, not DOM queries.
- Prefer semantic selectors in order: `getByRole` → `getByLabel` → `getByText` → `data-testid`. Use `data-testid` only as a last resort.
- Run Playwright tests in CI against a real running application, not mocks.
- Include **axe-core accessibility checks** (`@axe-core/playwright`) as part of the E2E suite — run on every critical page.
