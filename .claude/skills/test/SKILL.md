---
name: test
description: Run the test suite, interpret results, identify missing coverage,
  and suggest or write new tests for JS/TS code. Use when adding features,
  fixing bugs, or reviewing test coverage.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - Edit
---

# Test

Run tests, interpret failures, and improve test coverage for JS/TS projects.

## Phase 1: Run Existing Tests

Detect the test runner from `package.json` and run tests:

```bash
cat package.json   # check "scripts" and "devDependencies"
```

Run appropriately:
- **Vitest**: `npm run test -- --reporter=verbose`
- **Jest**: `npm test -- --verbose`
- **Node test runner**: `node --test`

For a single file: `npm test -- path/to/file.test.ts`

## Phase 2: Interpret Results

For each failure:
1. Read the failing test and the source file it covers
2. Determine: is the test wrong, or is the code wrong?
3. Explain the root cause clearly before suggesting a fix
4. Never update tests to make them pass if the code is the problem — fix the source

For a clean run: report the coverage summary if available, then move to Phase 3.

## Phase 3: Identify Coverage Gaps (when requested or coverage is low)

Find untested code paths:
- Functions or modules with no corresponding `*.test.ts` file
- Exported functions not imported in any test
- Error paths and edge cases missing from existing tests

Cross-reference exports with tests:

```bash
# Find all exports in src
grep -r "^export" src/ --include="*.ts" -l

# Find which modules are imported in tests
grep -r "from '\.\." tests/ --include="*.ts"
```

## Phase 4: Write or Suggest Tests

Follow these conventions:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
// or: import { describe, it, expect } from '@jest/globals'
import { functionUnderTest } from '../src/module'

describe('functionUnderTest', () => {
  it('returns expected value for valid input', () => {
    expect(functionUnderTest('input')).toBe('expected')
  })

  it('throws on invalid input', () => {
    expect(() => functionUnderTest(null)).toThrow(TypeError)
  })
})
```

**Principles:**
- Test behaviour, not implementation — tests should survive a refactor
- One logical assertion per test where feasible; group with `describe`
- Cover: happy path, edge cases (empty, null, 0, boundary values), error paths
- Mock at the boundary (network, filesystem, time), not inside business logic
- Use `beforeEach` to reset state; avoid shared mutable state between tests

## Wrap Up

Summarise:
- Tests run: X passed, Y failed
- For each failure: root cause and recommended fix
- Coverage gaps identified (if analysed)
- Tests written or suggested, with file paths and test names
