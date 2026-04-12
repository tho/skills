# GitHub CLI Workflows

Use the `gh` CLI to interact with GitHub.

## Pull Requests

List open PRs:
```bash
gh pr list --repo owner/repo
```

Check CI status on a PR:
```bash
gh pr checks <pr_number> --repo owner/repo
```

View failed workflow logs:
```bash
gh run view <run-id> --repo owner/repo --log-failed
```

Re-run only the failed jobs:
```bash
gh run rerun <run-id> --failed --repo owner/repo
```

List recent workflow runs (useful to find a run-id):
```bash
gh run list --repo owner/repo --limit 10
```

## Creating Pull Requests

Before creating, check for a PR template at the conventional paths and use it to structure the description:

- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/pull_request_template.md`
- `docs/pull_request_template.md`
- `PULL_REQUEST_TEMPLATE.md`

If no template is found, write a description covering motivation, changes, and testing.

Create as a **draft** unless the user requests otherwise or project convention differs:
```bash
gh pr create --repo owner/repo --draft --title "..." --body "..."
```

Once ready for review, promote the draft:
```bash
gh pr ready <pr_number> --repo owner/repo
```

Update title, body, labels, or assignees after creation:
```bash
gh pr edit <pr_number> --repo owner/repo --title "..." --body "..."
```

## Merging Pull Requests

Check the project's merge conventions before merging (repo settings, contributing guide, or ask the user). If no convention is established, squash works well for short-lived feature branches; merge commits preserve granular history on long-lived branches.

```bash
# Merge with squash
gh pr merge <pr_number> --repo owner/repo --squash --delete-branch

# Merge with a merge commit
gh pr merge <pr_number> --repo owner/repo --merge --delete-branch

# Merge automatically once all checks pass
gh pr merge <pr_number> --repo owner/repo --squash --auto
```

> **Note:** Only pass `--delete-branch` when the branch is a short-lived feature branch and the user has not indicated it should be kept.

## Reviewing Pull Requests

Only approve or request changes when explicitly asked by the user — never proactively.

```bash
# Approve
gh pr review <pr_number> --repo owner/repo --approve

# Request changes
gh pr review <pr_number> --repo owner/repo --request-changes --body "..."

# Comment without approving or blocking
gh pr review <pr_number> --repo owner/repo --comment --body "..."
```

Add a standalone comment to a PR:
```bash
gh pr comment <pr_number> --repo owner/repo --body "..."
```

## Issues

View an issue:
```bash
gh issue view <issue_number> --repo owner/repo
```

Create an issue:
```bash
gh issue create --repo owner/repo --title "..." --body "..."
```

Close an issue with a comment:
```bash
gh issue close <issue_number> --repo owner/repo --comment "Fixed in #<pr_number>"
```

> **Note:** This comment is informational only. To auto-close an issue when a PR merges, use a closing keyword in the **PR body** instead: `Closes #<issue_number>`, `Fixes #<issue_number>`, etc.

## API for Advanced Queries

Use `gh api` for fields or endpoints not exposed by named subcommands.

Get PR with specific fields:
```bash
gh api repos/owner/repo/pulls/<pr_number> --jq '.title, .state, .user.login'
```

## JSON Output

Most list commands (`gh pr list`, `gh issue list`, `gh run list`, etc.) support `--json` for structured output. Use `--jq` to filter:

```bash
gh issue list --repo owner/repo --json number,title --jq '.[] | "\(.number): \(.title)"'
gh pr list --repo owner/repo --json number,title,state --jq '.[] | "\(.number): \(.title) [\(.state)]"'
```
