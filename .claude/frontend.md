# Frontend (React)

## React Version & Patterns

- Before implementing any React feature, consult **[react.dev](https://react.dev)** to verify the current recommended approach. Use `WebFetch` on react.dev when in doubt — never rely solely on training data for React APIs.
- Use **React Server Components (RSC)** for data fetching and server-side logic by default. Mark components `"use client"` only when interactivity or browser APIs require it.
- Use **hooks exclusively** — no class components, no legacy lifecycle methods.
- Prefer modern built-in hooks: `use()`, `useActionState`, `useOptimistic`, `useFormStatus`. Avoid legacy workarounds they replace.
- Never use patterns marked deprecated on react.dev. Migrate proactively on major releases.
- Avoid `useEffect` for data fetching; use RSC or Suspense-compatible data fetching instead.

## Component Design

- **Composition over configuration**: small, single-purpose components composed together.
- Keep components **pure**: no side effects during render. Side effects belong in event handlers or `useEffect` (sparingly).
- Colocate state as close as possible to where it is used. Lift only when siblings genuinely share it.
- The React Compiler handles most memoization — do not manually add `useMemo`/`useCallback`/`React.memo` unless profiling proves it necessary.

## State Management

- **Local state first** (`useState`, `useReducer`). Lift to shared context only when needed.
- Use `useContext` for low-frequency global state (theme, current user). For high-frequency updates, use Zustand or Jotai.
- **Server state belongs server-side**: RSC or TanStack Query — not `useState` + `useEffect`.
- Avoid Redux unless already established in the project.

## Accessibility

- Use **semantic HTML** elements. Never use a `<div>` or `<span>` where a semantic element exists.
- Every interactive element must be keyboard-navigable and have an accessible label.
- Use ARIA attributes only when semantic HTML is insufficient.
- Accessibility violations are bugs. Include axe-core checks in the E2E suite.

## TDD for Frontend

- Use **React Testing Library + Vitest** for component and hook tests.
- Test **behaviour, not implementation**: query by role, label, or text — never by CSS class or component internals.
- Never assert on internal state directly.
- Mock at the **network boundary using MSW** (Mock Service Worker), not at the module level.
- One failing test at a time — same hard rules as backend TDD apply.

## E2E Testing with Playwright

- E2E tests cover **complete user journeys**: happy path + critical error paths.
- Use **Page Object Model (POM)** to encapsulate selectors and interactions.
- Prefer semantic selectors: `getByRole` → `getByLabel` → `getByText` → `data-testid`.
- Run Playwright tests in CI against a real running application, not mocks.
- Include **axe-core accessibility checks** (`@axe-core/playwright`) on every critical page.
