# Project Instructions

This file gives Claude context about this project. Replace bracketed sections
when copying this template to a real project.

## Project Overview

**Type**: [Web app / CLI script / library / API service]
**Stack**: [e.g. React + Vite + TypeScript / Node.js + Express / plain JS scripts]
**Purpose**: [One sentence describing what this project does]

## Tech Stack

- **Runtime**: Node.js [version] / Browser
- **Language**: TypeScript [version] (strict mode)
- **Package manager**: [npm / pnpm / yarn] — use one consistently
- **Bundler**: [Vite / esbuild / tsc / none]
- **Test runner**: [Vitest / Jest / Node test runner]
- **Linter/formatter**: ESLint + Prettier

## Project Structure

```
src/
  components/   # UI components (if applicable)
  lib/          # Shared utilities and helpers
  types/        # TypeScript type definitions
  scripts/      # Standalone scripts
tests/          # Test files mirror src/ structure
dist/           # Build output (gitignored)
```

## Development Commands

```bash
npm install       # Install dependencies
npm run dev       # Start dev server
npm run build     # Production build
npm test          # Run test suite
npm run lint      # Lint with ESLint
npm run format    # Format with Prettier
npm run typecheck # tsc --noEmit
```

## Code Conventions

- TypeScript strict mode is enabled — no implicit `any`
- Prefer `const` over `let`; avoid `var`
- Use named exports; avoid default exports except for React components
- Async functions use `async/await`; avoid raw Promise chains
- Error handling: always handle rejected promises, never swallow errors silently
- File naming: `kebab-case.ts` for modules, `PascalCase.tsx` for components
- See `.claude/rules/code-style.md` for the full style guide

## Testing Approach

- Unit tests live in `tests/`, mirroring the `src/` structure
- Test file naming: `*.test.ts`
- Write behaviour-focused tests, not implementation-detail tests
- Run a single test: `npm test -- --testNamePattern "pattern"`
- Mocking: use Vitest's `vi.mock()` / Jest's `jest.mock()`

## Git Conventions

- Commit style: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`)
- Branch naming: `feat/description`, `fix/description`, `chore/description`
- Never commit: `node_modules/`, `.env`, `dist/`, build artifacts
- Use `/commit` skill to generate commit messages

## Security Notes

- Never commit API keys, tokens, or secrets — use `.env` files
- `.env` is gitignored; `.env.example` is committed with placeholder values
- Secrets loaded via `process.env` only, never hardcoded

## Working with Claude

- Use `/review` before opening a PR or merging a branch
- Use `/commit` to generate a conventional commit message and commit
- Use `/test` when adding features — it will suggest missing test cases
- Use `/document` when a function or module needs JSDoc or README updates
- Ask Claude to explain before editing unfamiliar code
- Prefer small, focused requests over large "do everything" prompts
