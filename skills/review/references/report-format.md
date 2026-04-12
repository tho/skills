# Review Report

Use this as the core output shape for review findings.

## Summary

A short paragraph covering:
- what was reviewed and how the scope was determined
- the overall assessment in one line (e.g., "two blocking issues in auth; otherwise solid")
- counts by severity, formatted as `blocking: N, warning: N, note: N`

If any blocking findings exist, list their titles in the summary so a reader can see what must be fixed without scrolling.

## Findings

Group findings by lens. Omit lenses with no findings. Within each lens, order findings by severity: `blocking` first, then `warning`, then `note`.

### [Lens Name]

#### [blocking|warning|note] Finding title
- **Location**: `path/to/file.ext:line` or a clear region/function name
- **What**: what the issue is, concretely
- **Impact**: the failure mode, maintenance risk, or exploit path
- **Fix**: what to change, if a concrete fix is appropriate

**Severity guide**
- `blocking` — do not merge: correctness bug, security hole, data-loss risk, or broken contract
- `warning` — real concern; address before or shortly after merging
- `note` — low-urgency observation; use sparingly so the signal-to-noise ratio stays high

## Verdict (optional)

Include this only when the calling workflow asks for a decision, such as a PR review.

Use one of:
- **Approve** — no blocking findings
- **Request Changes** — at least one blocking finding; name which ones
- **Comment** — observations only, no ask

Tie the verdict directly to the blocking findings by title so the author knows exactly what to address.

## Workflow-specific wrappers

Other workflows may add extra sections around this core report.
Examples include PR-specific sections such as:
- Claims vs Reality
- Change Analysis
- Merge recommendation rationale
