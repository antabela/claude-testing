---
name: code-review
description: Review staged or specified code for quality, security, correctness,
  and JS/TS best practices. Use when preparing to open a PR, merge a branch, or
  audit a file or function.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Code Review

Perform a thorough code review focused on JS/TS quality, security, and
correctness. Produce actionable, prioritised findings.

## Scope

If the user specifies files or a diff, review those. Otherwise:
1. Run `git diff main...HEAD` to get the full changeset vs main
2. If on main, run `git diff --cached` for staged changes
3. Read any files not fully represented in the diff

## Review Checklist

Work through each category. Skip categories with no issues rather than writing
"No issues found."

### Correctness
- Logic errors or off-by-one conditions
- Unhandled edge cases (empty arrays, null/undefined, 0, empty string)
- Race conditions in async code
- Incorrect comparisons (`==` vs `===`, wrong type checks)

### TypeScript Quality
- Unsafe `any` usage
- Non-null assertions (`!`) without justification
- Missing or inaccurate types on exported APIs
- Overly broad types that defeat type safety

### Security
- Secrets or credentials hardcoded in source
- User-controlled input used in SQL, shell commands, or file paths (injection)
- Missing input validation on external data
- Sensitive data logged to console

### Performance
- N+1 query patterns or unnecessary re-fetching
- Missing `useMemo`/`useCallback` on expensive computations (React)
- Synchronous operations blocking the event loop
- Unnecessarily large bundle imports (e.g. `import _ from 'lodash'`)

### Maintainability
- Functions over 40 lines — suggest splitting
- Deep nesting (> 3 levels) — suggest early returns
- Magic numbers or strings without named constants
- Duplicated logic that should be extracted to a shared helper
- Misleading variable or function names

### Tests
- New code paths without test coverage
- Tests asserting implementation details instead of behaviour
- Missing error-path tests

## Output Format

Structure findings as:

**[CRITICAL]** — Must fix before merge. Security issue or likely bug.
**[MAJOR]** — Should fix. Correctness or significant quality problem.
**[MINOR]** — Worth addressing. Maintainability or style issue.
**[SUGGESTION]** — Optional improvement. Performance or elegance.

For each finding include:
- File name and line number (if known)
- Concise description of the problem
- A concrete fix or corrected code example

End with a one-paragraph verdict: ready to merge, needs minor fixes, or needs
significant rework.
