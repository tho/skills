# PR Review Workflow

Review a GitHub pull request given its URL. Use `gh` to fetch PR metadata and diff, then analyse using the lenses, severity calibration, and noise discipline from `/skill:review`.

## Steps

### 1. Parse the PR URL

Extract `owner`, `repo`, and `pr_number` from the link (for example `https://github.com/owner/repo/pull/123`).

### 2. Fetch PR metadata

```bash
gh pr view <pr_number> --repo owner/repo --json title,body,author,baseRefName,headRefName,labels,reviewDecision,state,additions,deletions,changedFiles,reviews
```

Read and understand the PR title and description. These are the author's claims about what the PR does. Note any existing reviews so findings are not redundant.

### 3. Check CI status

```bash
gh pr checks <pr_number> --repo owner/repo
```

Note any failing checks — these are blocking signals relevant to the verdict.

### 4. Fetch the diff

```bash
gh pr diff <pr_number> --repo owner/repo
```

If the diff is very large (>200 KB), fetch the file list first and review in batches:

```bash
gh pr diff <pr_number> --repo owner/repo --name-only
```

Then read individual files or hunks as needed.

### 5. Analyse and report

Using the lenses, severity calibration, and noise discipline from `/skill:review`, analyse the diff and produce the following report. Gather any extra repository context needed (surrounding files, tests, docs) before forming judgments.

Produce one unified report with these sections:

#### Summary

One short paragraph: what the PR actually does based on the diff, plus an overall one-line assessment and severity counts (`blocking: N, warning: N, note: N`). If blocking findings exist, name them here.

#### Claims vs Reality

Compare the PR description against the actual changes.

- **[confirmed]**: the description says it and the diff supports it.
- **[unsubstantiated]**: the description says it but the diff does not show it.
- **[undisclosed]**: the diff does something the description does not mention.

Skip this section if the description is empty or trivially matches.

#### Change Analysis

For each changed file or logical file group, briefly note what changed, why it likely changed, and anything unusual or risky. If the PR touches more than ~15 files, summarise by logical grouping rather than file-by-file.

#### Findings

Findings from the review, grouped by lens (omit lenses with no findings). Within each lens, order by severity: `blocking` first, then `warning`, then `note`.

#### Verdict

Choose one and tie it directly to the relevant findings by title:

- **Approve** — no blocking findings
- **Request Changes** — at least one blocking finding; name which ones
- **Comment** — observations only, no ask

## Submitting the review

After presenting the review, offer to submit it to GitHub via `gh pr review`.
Only do so when the user confirms.
Create as a **draft** unless the user requests otherwise or project convention differs.

## Notes

- Keep PR-specific framing here; keep reusable review methodology in `/skill:review`.
