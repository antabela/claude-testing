---
description: JavaScript and TypeScript code style rules for this project.
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.mjs"
---

# Code Style: JavaScript and TypeScript

## TypeScript

- Always use `strict: true` in tsconfig — no exceptions
- Prefer `type` aliases for unions/intersections; use `interface` for object shapes that may be extended
- No `any` — use `unknown` when the type is genuinely unknown, then narrow it
- No non-null assertions (`!`) — handle `null`/`undefined` explicitly
- Explicit return types on exported functions; inferred types are fine for private/internal functions
- Use `satisfies` operator when you want type checking without widening

## Naming

- Variables and functions: `camelCase`
- Types, interfaces, classes, enums: `PascalCase`
- Constants known at compile time: `SCREAMING_SNAKE_CASE`
- Files: `kebab-case.ts` for utilities/modules, `PascalCase.tsx` for components
- Boolean variables: prefix with `is`, `has`, `can`, or `should`

## Functions

- Prefer function declarations for top-level named functions
- Arrow functions for callbacks, event handlers, and inline expressions
- Keep functions under 40 lines; extract helpers rather than nesting logic
- Pure functions are preferred — no hidden side effects
- Single responsibility: one function does one thing

## Modules

- Named exports only, except for React components (default export acceptable)
- Import order: built-ins → external packages → internal modules → types
- No circular imports — restructure into a shared module if needed
- Barrel files (`index.ts`) are fine for public APIs; avoid deep nesting of them

## Async / Error Handling

- Always `await` promises or explicitly handle them; never fire-and-forget without a comment
- `try/catch` should catch specific error types where possible
- Never swallow errors silently — at minimum log in development
- Prefer returning a typed result object (`{ data, error }`) over throwing in business logic; let infrastructure throw

## React (when applicable)

- Functional components with hooks only — no class components
- Props interface defined above the component
- Custom hooks start with `use`
- Avoid `useEffect` for derived state; use `useMemo` or computed values instead
- Event handler functions named `handle<Event>` (e.g., `handleClick`, `handleSubmit`)

## Formatting (enforced by Prettier)

- 2-space indentation
- Single quotes for strings
- Trailing commas in multi-line structures
- Max line length: 100 characters

## Comments

- JSDoc on all exported functions, types, and classes
- Inline comments explain *why*, not *what* — good names explain what
- TODO comments include a date: `// TODO(2026-04-18): description`
- Delete commented-out code; git has history
