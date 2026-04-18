---
name: document
description: Generate or update documentation — JSDoc comments, README sections,
  inline comments, or a CHANGELOG entry. Use after adding a feature, writing a
  public API, or when documentation is stale or missing.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
  - Bash
---

# Document

Write or update documentation for JS/TS code. Targets: JSDoc, README, and
CHANGELOG.

## Step 1: Understand What Needs Documenting

Read the relevant source files. Focus on:
- Exported functions, classes, and types
- Non-obvious logic that lacks inline comments
- Public-facing APIs (what a caller needs to know)
- Recent changes that may be undocumented

```bash
git log --oneline -10   # what changed recently
```

## Step 2: JSDoc

Add JSDoc to every exported function, class, and type that lacks it.

**Template:**
```typescript
/**
 * Brief one-line description in imperative mood.
 *
 * More detail if behaviour is non-obvious. Explain edge cases,
 * side effects, or important constraints here.
 *
 * @param paramName - Description and valid values
 * @param options - Configuration object
 * @param options.timeout - Milliseconds before abort (default: 5000)
 * @returns Description of the return value
 * @throws {TypeError} When paramName is not a string
 * @example
 * ```typescript
 * const result = myFunction('hello', { timeout: 3000 })
 * console.log(result) // 'HELLO'
 * ```
 */
```

**Rules:**
- First line: single imperative sentence ("Parse the response", not "This parses")
- Document `@throws` for every error a caller might need to catch
- Include `@example` for any function whose usage is non-obvious
- Types visible in the signature don't need to be restated in prose

## Step 3: README

When asked to update or create a README:

```markdown
# Project Name

One-sentence description.

## Installation
\`\`\`bash
npm install
\`\`\`

## Usage
[Minimal working example]

## API Reference
[For libraries: key exported functions with signatures]

## Development
\`\`\`bash
npm run dev      # dev server
npm test         # run tests
npm run lint     # lint
npm run build    # production build
\`\`\`

## License
[License name]
```

## Step 4: CHANGELOG Entry

Use Keep a Changelog format (https://keepachangelog.com):

```markdown
## [Unreleased]

### Added
- New feature description

### Changed
- Modified behaviour description

### Fixed
- Bug fix description

### Removed
- Removed feature description
```

Populate from git log since the last tagged release:
```bash
git log <last-tag>..HEAD --oneline
# or if no tags yet:
git log --oneline
```

## Wrap Up

List every file modified or created with a one-line summary of what was
documented. Flag any exported symbols still lacking documentation so the
user can prioritise them.
