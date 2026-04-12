---
name: review
description: This skill should be used when the user asks to review code, check a diff, look over staged or unstaged changes, assess whether changes are ready to merge, or asks questions like "does this look right", "any concerns with this PR", "is this safe to ship", or "review this function".
---

# Review

## Workflow

1. **Determine the review scope.** Use explicit user input first, then any diff handed in by another workflow, then staged changes, then unstaged changes, then a branch diff. If both staged and unstaged changes exist and the request is ambiguous, ask which scope to review.
2. **Gather context before judging.** First understand what the change claims to do — read the commit message, PR description, or any linked issue. Then read the changed files, nearby code, tests, and any repository docs or conventions needed to understand how this area works. A clear picture of the stated intent is required to judge whether the implementation actually achieves it.
3. **Load the relevant lenses.** Load all lenses by default; skip any with no plausible relevance to the specific changes (e.g., a pure dependency version bump has low architecture relevance; a new external call has high security and operability relevance). Read the reference file for each loaded lens before forming judgments. Narrow or broaden scope if the user asks.
4. **Focus on judgment, not mechanics.** Do not spend the review on formatting, lint, or other purely automatable issues unless they point to a deeper correctness, design, or safety problem. Skip findings that reflect personal preference with no concrete harm, or where multiple valid implementations exist.
5. **Produce the report.** Read `references/report-format.md` and produce the report following that structure. Group findings by lens and omit empty sections.
6. **Suggest alternative approaches only when needed.** Keep this off by default. Offer alternatives only if the user asks or the review reveals a concrete structural problem.

## Available lenses

| Lens | Use when reviewing for | Reference |
|---|---|---|
| Architecture | boundaries, coupling, abstraction fit, pattern drift | `references/lenses/architecture.md` |
| Correctness | logic errors, edge cases, state handling, failure modes | `references/lenses/correctness.md` |
| Security | trust boundaries, auth, secrets, unsafe input handling | `references/lenses/security.md` |
| Quality | clarity, maintainability, docs, performance, accessibility | `references/lenses/quality.md` |
| Testing | missing coverage, weak assertions, regression gaps | `references/lenses/testing.md` |
| Dependencies | new packages, version bumps, supply-chain and maintenance risk | `references/lenses/dependencies.md` |
| Operability | observability, rollout safety, failure visibility, production debuggability | `references/lenses/operability.md` |

## Review notes

- State the scope you reviewed before presenting findings.
- Prefer concrete findings over generic advice.
- When a concern depends on missing context, say what would need to be checked.
- If no issues are found in a loaded lens, omit that lens from the final report.

### Severity calibration

- **Blocking** — the change should not merge as-is: a correctness bug, security hole, data-loss risk, or breaking contract. Use it precisely and rarely.
- **Warning** — a real concern that should be addressed, but not necessarily before merging if a clear follow-up path exists.
- **Note** — a helpful observation with no urgency. Use sparingly; a review dominated by notes trains authors to ignore them.

### Noise discipline

Do not raise findings for:
- Formatting, whitespace, or style issues that a linter or formatter should enforce
- Subjective preferences where multiple valid approaches exist and the choice carries no concrete risk
- Theoretical concerns without evidence of a realistic trigger or failure path
- Architecture opinions on small, isolated changes where no boundary is actually crossed

When the same low-severity pattern repeats, flag it once with the scope noted rather than listing every instance.
