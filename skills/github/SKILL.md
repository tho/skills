---
name: github
description: "This skill should be used when the user wants to work with GitHub: opening or reviewing a pull request, checking CI or workflow run status, managing issues, or running gh CLI commands. Typical triggers include 'create a PR', 'check CI on this branch', 'list open issues', 'merge this PR', or 'review PR #42'."
---

# GitHub

## Shared guidance

- When running `gh` outside the target repository, always pass `--repo owner/repo`.
- Prefer URLs as inputs when the user already gave one (PR, issue, run, repository).
- Keep GitHub ceremony here and delegate deep code-analysis work to more focused skills when available.

## Routing

Load the relevant reference file only when the task requires it, not proactively.

- **Committing code** → use `/skill:git-commit`
- **Reviewing a pull request** → load `references/pr-review.md`; PR review also delegates code analysis to `/skill:review` (load it alongside this skill for full analysis)
- **Using the `gh` CLI for issues, PRs, CI, or API calls** → load `references/gh-cli.md`

## Notes

- If the task mixes categories, load every relevant reference file instead of proceeding without reference documentation.
