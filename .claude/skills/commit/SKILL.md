---
name: commit
description: Generate a Conventional Commits message for current staged or unstaged
  changes, then commit. Use after finishing a logical unit of work.
allowed-tools:
  - Bash
  - Read
---

# Commit

Generate a well-formed Conventional Commits message and commit the changes.

## Workflow

### 1. Understand the Changes

```bash
git status
git diff --cached    # staged changes
git diff             # unstaged changes (for context)
```

If nothing is staged, ask the user which files to stage before proceeding.

### 2. Read Context (if needed)

For non-obvious changes, read the relevant source files to understand intent.
Never invent a commit message without understanding what the code does.

### 3. Draft the Commit Message

Follow the Conventional Commits spec:

```
<type>(<optional scope>): <short imperative description>

<optional body — explain WHY, not WHAT. Wrap at 72 chars.>

<optional footer: BREAKING CHANGE: ..., Closes #123, etc.>
```

**Types:**
- `feat` — new feature or user-visible capability
- `fix` — bug fix
- `refactor` — restructuring without behaviour change
- `test` — adding or fixing tests
- `docs` — documentation only
- `chore` — tooling, deps, config, build
- `perf` — performance improvement
- `style` — formatting, whitespace (no logic change)
- `ci` — CI/CD pipeline changes

**Rules:**
- Subject line: imperative mood ("add feature", not "added feature")
- Subject line: 50 characters or fewer
- Subject line: no trailing period
- Body: explain motivation and contrast with previous behaviour
- Breaking changes: add `BREAKING CHANGE:` footer

### 4. Stage and Commit

Stage the appropriate files (ask the user if scope is ambiguous), then commit:

```bash
git add <files>
git commit -m "$(cat <<'EOF'
<message here>
EOF
)"
```

### 5. Confirm

Show `git log --oneline -3` so the user can see the commit landed.

## Examples

```
feat(auth): add JWT refresh token rotation

Tokens were previously never invalidated. This adds sliding-window
rotation so a compromised refresh token expires within one hour of
non-use.

Closes #42
```

```
fix(api): handle empty response body in fetchUser

fetchUser threw an uncaught TypeError when the API returned 204.
Added an early return for empty bodies.
```

```
chore: upgrade Vite to 6.1.0
```
