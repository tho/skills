---
name: git-commit
description: >
  This skill should be used when the user wants to commit changes, stage files and commit,
  create a git commit message, or asks what conventional commit type to use. Also applies
  when the user says "save my changes", "write a commit message", "prepare a commit",
  "stage and commit", or invokes "/commit".
---

# Git Commit with Conventional Commits

## Conventional Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit Types

| Type       | When to Use                             | Example                                  |
| ---------- | --------------------------------------- | ---------------------------------------- |
| `feat`     | New files/functions/features added      | `feat(api): add user profile endpoint`   |
| `fix`      | Fixes bug/error/incorrect behavior      | `fix(parser): handle null values`        |
| `docs`     | Only docs/README/comments modified      | `docs: update installation guide`        |
| `style`    | Formatting/linting changes only         | `style: format code with prettier`       |
| `refactor` | Code restructuring (no logic change)    | `refactor(core): simplify error handler` |
| `perf`     | Performance optimization                | `perf: optimize database queries`        |
| `test`     | Only test files modified                | `test: add integration tests for API`    |
| `build`    | Dependencies/build config               | `build: upgrade webpack to v5`           |
| `ci`       | CI/CD configuration                     | `ci: add automated deployment`           |
| `chore`    | Other maintenance                       | `chore: update gitignore`                |
| `revert`   | Reverting previous commit               | `revert: undo breaking API change`       |

**Breaking Changes:** Add an exclamation mark after type/scope (e.g., `feat!:`) or use a `BREAKING CHANGE:` footer. See [references/SPECIFICATION.md](references/SPECIFICATION.md) for full details.

## Agent Workflow

Follow these steps to create a commit:

### Step 1: Assess Current State

Run `git status --short` and `git diff --staged` in parallel.

- If staged changes exist → proceed to analyze them
- If nothing staged → check `git diff` for unstaged changes

### Step 2: Determine Commit Type and Scope

Analyze the diff in a single pass to determine both the commit type and scope simultaneously:

- Map changes to the appropriate type from the [Commit Types](#commit-types) table above
- Extract scope from file paths or affected modules:
  - File in `src/auth/` → scope: `auth`
  - Changes to `api/users.ts` → scope: `api` or `users`
  - Multiple unrelated changes → omit scope
- If the diff spans multiple unrelated concerns, stop and split: stage and commit each concern separately before proceeding

### Step 3: Generate Description

Create a concise description (imperative mood, present tense, <72 chars):
- ✓ "add login endpoint"
- ✓ "fix null pointer exception"
- ✗ "added login endpoint"
- ✗ "fixes bug"

### Step 4: Stage Files (if needed)

If Step 1 found nothing staged, stage the relevant files now (before committing). Stage specific files by path or glob pattern.

- **NEVER** use `git add -A` or `git add .`
- **NEVER** commit secrets (.env, credentials.json, API keys, private keys)

### Step 5: Execute Commit

```bash
# Single line commit
git commit -m "<type>(<scope>): <description>"

# Multi-line with body/footers (use HEREDOC for proper formatting)
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

Additional context about the changes.
Why this change was needed.

Closes #123
EOF
)"
```

## Message Writing Guidelines

- **One logical change per commit** - Group related changes together
- **Present tense, imperative mood** - "add feature" not "added feature" or "adds feature"
- **Lowercase description** - Don't capitalize first word unless it's a proper noun
- **No period at end** - Description should not end with punctuation
- **Keep description under 72 characters** - Be concise
- **No dashes for punctuation** - Never use dashes for punctuation (project convention, not part of the Conventional Commits spec)
- **No unicode symbols or emojis** - Commit messages are plain text only
- **Reference issues in footer** - Use `Closes #123` or `Refs #456`

## Git Safety Protocol

- **NEVER** update git config
- **NEVER** run destructive commands (`--force`, `hard reset`) without explicit user request
- **NEVER** skip hooks (`--no-verify`, `--no-gpg-sign`) unless user explicitly asks
- **NEVER** force push to `main` or `master` branches
- **NEVER** commit sensitive files (.env, credentials.json, private keys, API keys)
- **If commit fails due to hooks** - Fix the issue and create a NEW commit (don't use `--amend` unless explicitly requested)

## References

- [Full Conventional Commits Specification](references/SPECIFICATION.md) - Complete RFC-style specification with breaking change examples
