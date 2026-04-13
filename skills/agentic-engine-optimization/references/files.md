# AEO file guide

## llms.txt

A structured index at the project root that tells agents what documentation exists and where to find it. Think of it as a sitemap for agents.

Format: flat Markdown. Keep the index itself under 5,000 tokens.

```markdown
# Project Name

> One-line description of what this project does.

## Docs

- [Quick Start](/docs/quickstart.md): Set up and run your first example (~2K tokens)
- [API Reference](/docs/api.md): All endpoints with request/response examples (~8K tokens)
- [Configuration](/docs/config.md): Environment variables and options (~3K tokens)
```

Guidelines:
- Organize by task, not by product hierarchy.
- Describe what agents will *find* at each link, not just the page title.
- Include approximate token counts so agents can budget context. Estimate roughly 1 token per 4 characters of English prose. For code-heavy files, use `wc -c <file>` divided by 3 as a conservative estimate (code and symbols tokenize more densely than prose).
- Link to raw Markdown where possible (agents parse it more cheaply than HTML).

**`llms-full.txt`:** A companion format that concatenates all documentation into a single file. Useful for agents that prefer one fetch over following links. Place it at the project root alongside `llms.txt`. The `llms.txt` spec originated from llmstxt.org and defines both formats.

## AGENTS.md

Repository-level guidance loaded by AI coding agents on every request. Keep it under ~200 lines and as minimal as possible, for two reasons:

1. **Context budget.** Every token in AGENTS.md loads on every request, displacing tokens available for the actual task. Compliance degrades as instruction count grows.
2. **Over-specification.** Dense rules cause agents to satisfy the rules rather than solve the problem. They treat the stated list as exhaustive, spend reasoning on compliance instead of correctness, and follow instructions mechanically rather than finding the right solution. Less guidance often produces better outcomes.

**Include only what agents cannot discover themselves.** Current frontier models can run `ls`, `grep`, read file trees, and infer conventions from existing code. Do not document file structure, obvious patterns, or anything a capable developer would figure out in a few commands.

**What belongs here (and only here at the root level):**
- One-sentence project description
- Non-standard tooling or commands (build, test, lint) that deviate from conventions the agent would reasonably assume
- Hard constraints not visible in the code (e.g. required credentials, environment assumptions)

**Progressive disclosure:** Move domain-specific rules to their own files and link to them:

```markdown
This is a Go HTTP API for managing user accounts and billing.

Run tests: make test
Run linter: golangci-lint run

For API conventions, see docs/API.md
For error handling patterns, see docs/ERRORS.md
```

Agents navigate to those files only when the task requires it, keeping irrelevant content out of context.

**What not to include:**
- Language-specific style rules (extract to a separate file)
- Testing patterns (extract to a separate file)
- File structure descriptions (the agent can explore)
- Reusable workflows or procedures (these belong in agent skills, not AGENTS.md)
- Vague aspirational directives ("write clean code", "follow best practices") that add no actionable constraint

**Size:** Keep each file under ~200 lines. Anthropic's official Claude Code documentation states adherence degrades beyond this threshold. The same principle applies to other tools.

**Tool-specific files:** Different coding agents read different files. `CLAUDE.md` for Claude Code, `GEMINI.md` for Gemini CLI, `AGENTS.md` for Cursor and GitHub Copilot. Maintain one source of truth in `AGENTS.md` and have the others import it:

- Claude Code supports `@path/to/file` import syntax. Create a `CLAUDE.md` containing `@AGENTS.md`, or symlink: `ln -s AGENTS.md CLAUDE.md`
- Gemini CLI reads `GEMINI.md`; create it with the same import or symlink approach

When managing multiple tool-specific files, prefer symlinks: they work transparently for tools that cannot resolve `@` imports. Use `@AGENTS.md` only when `CLAUDE.md` needs to extend AGENTS.md with Claude-specific additions.

**Monorepos:** Place a root AGENTS.md scoped to the whole repo and a per-package AGENTS.md scoped to that package. Each stays focused on its own level.

## README improvements

Many projects do not need new files; they need a README that works better for agents.

- Lead with a clear, one-sentence description of what the project does and who it is for.
- Put installation and quickstart instructions near the top.
- Use consistent heading hierarchy (H1, H2, H3; no skipping).
- Place code examples immediately after the prose they illustrate.
- Use tables for parameter/option references instead of prose lists.
- Keep the README under ~15K tokens. Link to detailed docs rather than embedding everything.

## robots.txt

If the project has a website or hosted docs, check that robots.txt does not block AI agent user-agents. A misconfigured robots.txt silently prevents agents from accessing docs with no analytics signal.

There are two distinct categories of AI user-agents with different implications:

**Training crawlers** (index content for model training): `GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `Bytespider`. Blocking these is a legitimate choice if the project does not want its content used for training.

**Inference-time fetchers** (retrieve docs during an active agent session): `ChatGPT-User`, `Claude-User`, `Claude-Code`, `Gemini-Deep-Research`, `Perplexity-User`. Blocking these prevents agents from reading docs *while helping a user*, which is almost never the intent.

A common mistake is using a broad `Disallow: /` that catches both categories:

```
# Blocks training AND live agent doc fetches:
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /
```

If the project wants to block training crawlers but allow inference-time access, be specific:

```
# Block training crawlers only:
User-agent: GPTBot
Disallow: /

# Allow inference-time access (omit these entirely, or explicitly allow):
User-agent: Claude-User
Allow: /
```

## MCP server

If the project exposes an MCP (Model Context Protocol) server, add a `.mcp.json` at the repo root so agents can connect without manual configuration. The file maps server names to their startup commands:

```json
{
  "mcpServers": {
    "my-project": {
      "command": "npx",
      "args": ["-y", "my-project-mcp"]
    }
  }
}
```

MCP is defined at `modelcontextprotocol.io`. The protocol uses JSON-RPC 2.0 over stdio (for local servers) or Streamable HTTP (for remote). Cursor, VS Code with GitHub Copilot, Claude Code, and Gemini CLI all support MCP clients. There is no automatic server discovery in the MCP spec; clients must be given the server address or command, which is what `.mcp.json` provides.

## Structured docs

For projects with extensive documentation:
- Expose raw Markdown via URL convention (e.g., append `.md` or a query parameter).
- Lead each page with an outcome statement in the first ~200 words: what will the reader learn or be able to do.
- Keep individual pages under ~25K tokens. Chunk large references by resource or topic.
- Use tables for structured data (parameters, options, error codes).
