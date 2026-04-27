---
name: jira-story
description: >
  Captures rough messages, Slack snippets, or short descriptions into Jira
  stories, tasks, or bugs. Triggers on "create a Jira story/ticket/task",
  "add to the backlog", "log this in Jira", "make a ticket for this",
  "file a bug", "write up a story for", or a pasted message in a
  Jira-adjacent context. Handles routing, right-sized enrichment,
  clarification, and creation end to end.
---

# Jira Story Creation

Capture work into Jira with the right amount of detail. A story is a
**pointer to a future conversation**, not a spec. The goal is clear and
useful, not exhaustive.

Requires the Atlassian MCP server (`mcp-atlassian` or the Atlassian Remote
MCP). If its tools are not available, stop and tell the user to connect it.

This skill covers **creating** new Jira issues. If the user asks to update,
edit, or add detail to an existing issue, use the MCP's update tools
directly; this workflow does not apply.

---

## Workflow

**1. Route to a project.** In priority order:

- Parent ticket key in the message → fetch it, inherit its project.
- Project key in the message → use it.
- Keyword match against the registry below → use matched project.
- Otherwise → list available projects via the MCP and ask the user to pick.

**2. Decide the depth.** See the depth guide below. This is the most
important judgment call: do not default to the full template.

**3. Gather context silently.** Once routed, fetch the project's components
via the MCP so you only suggest names that exist. For labels, infer from
context (the user's message, related issues if needed); never invent.

**4. Clarify only what's genuinely missing.** Ask at most two questions in
one turn. Worth asking:

- Project (if step 1 came up empty)
- Story vs Task vs Bug (only if truly ambiguous)
- Core intent (if the message is too vague to draft anything)

Never ask about labels, components, priority, or story points. Infer and
let the user correct in review. Never demand a "so that" or AC if they
don't add information.

**5. Show the draft and confirm.** Display the full draft (summary,
type, description, AC if any), then end with a single confirmation line:

> "Create as a **Story** in **PLAT**, under **PLAT-12**? (yes / edits)"

**6. Create via the MCP**, then reply with the new key and a link of the
form `<JIRA_BASE_URL>/browse/<KEY>`.

---

## Depth guide

Pick one. The default is **Light**.

### Light: for chores, tweaks, small bugs, spikes, investigations

```
Summary:    <specific imperative; what + where>
Type:       Task | Bug
Description: 1-3 sentences of context. Why it matters or what triggered it.
```

No user story format. No acceptance criteria. No padding. Examples of work
that belongs here: typo fixes, dependency bumps, "look into why X is slow",
"add a log line", small refactors with obvious scope.

### Standard: for normal feature work and defined tasks

```
Summary:    <specific; what + where>
Type:       Story | Task | Bug
Description:
  Story → As a <persona>, I want <capability>[, so that <benefit>].
          Skip "so that" when it adds nothing (e.g. "so I can log in").
  Task  → 2-4 sentences: context, what's in scope, what's out.
  Bug   → Steps to reproduce. Expected vs actual. Environment if relevant.
Acceptance Criteria: include only if the story isn't self-evident from
                    the description. 2-5 bullets, plain language, testable.
```

### Heavy: for stories with real ambiguity, multiple scenarios, or cross-team handoff

Use Standard plus:

- **Gherkin AC** (Given/When/Then) when behaviour has clear preconditions
  and multiple scenarios worth pinning down.
- An explicit **Out of scope** line if scope creep is plausible.

Use this sparingly. If you find yourself reaching for it on most stories,
the work probably needs to be split.

---

## Calibration heuristics

Signals that **lighter is right**:

- Input is one or two sentences and the work is obvious.
- Operational, internal, or single-step work.
- The author is also likely the implementer.
- Phrases like "quick", "small", "just", "while we're at it".

Signals that **more structure is warranted**:

- Multiple stakeholders or teams will touch it.
- The behaviour has edge cases or non-obvious failure modes.
- The user's message itself contains conditions ("if X then Y", "but not when…").
- It's user-facing and will be tested by someone other than the author.

When unsure, prefer Light. It's easier to add detail later than to remove
ceremony from a backlog.

---

## What to avoid

- Writing acceptance criteria that just restate the summary.
- Forcing "As a / I want / so that" onto operational work where no real
  persona benefits ("As a developer, I want CI to be faster" is noise).
- Inventing requirements the user didn't state. If the input is vague, the
  ticket can be vague: that's what refinement is for.
- Inflating bullet counts. Three sharp criteria beat seven fuzzy ones.
- Inventing label or component names. Use only ones that exist in the
  project (components are returned by `jira_get_project_components`; for
  labels, infer from the user's words or from related issues).

---

## MCP quirks worth knowing

The Atlassian MCP exposes its tool schemas directly; trust those for names
and types. A few things the schema does not make obvious:

- **`additional_fields` is a JSON-encoded string**, not a dict. Labels,
  priority, and parent all live inside it:
  `'{"labels": ["..."], "priority": {"name": "High"}, "parent": "PROJ-123"}'`
- **`components` is a comma-separated string**, e.g. `"auth,email"`, not
  a list.
- **Acceptance criteria belong inside the description** as a Markdown
  subsection. There is no dedicated AC field.
- **Component names must already exist in the project.** Fetch them with
  `jira_get_project_components` before referencing; do not invent.

---

## Routing registry

Optional. One row per Jira project the user works with regularly. An
empty registry is fine; the workflow already handles it in step 1 by
listing projects via the MCP and asking the user to pick. The registry
just speeds up routing for repeat projects.

| Key | Name | Keywords | Default type | Notes |
|-----|------|----------|--------------|-------|
| ??? | ???  | ???      | Story / Task | ???   |

- **Key**: Jira project key (Project Settings > Details).
- **Keywords**: comma-separated words you naturally use to refer to this
  project. Be liberal. Matched as whole words, case-insensitive.
- **Default type**: `Story` for product-facing projects, `Task` for
  infra/ops. Used when intent is ambiguous.
- **Notes**: routing edge cases, e.g. "hotfixes go in OPS not here".

If multiple projects match, pick the one with most keyword hits; tie → ask.
An explicit project key always wins over keywords.

---

## Examples

### Light

**Input:** "The /healthz endpoint returns 200 even when the DB is down. PLAT."

**Draft:**
```
Summary:    /healthz returns 200 when DB is unreachable
Type:       Bug
Project:    PLAT

Description:
  The liveness probe at /healthz currently returns 200 regardless of DB
  connectivity, so a dead DB doesn't trigger restarts or alerts. Should
  fail when the DB check fails.
```

### Standard

**Input:** "Users want to reset their password by email. AUTH project."

**Draft:**
```
Summary:    Self-serve password reset by email
Type:       Story
Project:    AUTH

Description:
  As a user, I want to reset my password by email so that I'm not blocked
  when I forget it.

Acceptance Criteria:
  - Reset link is sent to the verified email on file
  - Link expires after 1 hour and can only be used once
  - Resetting invalidates any active sessions
```

### Heavy (used because the input has real conditional behaviour)

**Input:** "We need rate limiting on the public API. Different tiers should
get different limits, and authenticated requests should bypass the
anonymous limit. INFRA."

**Draft:**
```
Summary:    Add tiered rate limiting to the public API
Type:       Story
Project:    INFRA

Description:
  As an API consumer, I want rate limits that match my plan tier so that
  authenticated integrations are not throttled at the anonymous baseline.

  Out of scope: dashboard UI for viewing rate-limit usage; per-endpoint
  overrides (follow-up story).

Acceptance Criteria:
  Given an unauthenticated request
  When the anonymous rate limit is exceeded
  Then the API returns 429 with a Retry-After header

  Given an authenticated request on the free tier
  When the free-tier limit is exceeded
  Then the API returns 429 with a Retry-After header

  Given an authenticated request on the pro tier
  When the pro-tier limit is not exceeded
  Then the request is processed normally, even if it would exceed the
  free-tier limit
```

Gherkin AC is warranted here because there are multiple scenarios with
distinct preconditions worth pinning down before implementation.
