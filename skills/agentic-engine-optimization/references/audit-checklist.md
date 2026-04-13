# AEO audit checklist

Use this to assess a project's current agent-readiness and identify improvements.

## Discovery

- [ ] README leads with a clear, one-sentence project description
- [ ] README includes install/quickstart instructions near the top
- [ ] `llms.txt` exists with a structured index of docs (if the project has multiple doc pages)
- [ ] `AGENTS.md` exists if agents will work in the codebase (covers project description, package manager, non-standard commands)
- [ ] `AGENTS.md` is minimal: no embedded style guides, testing rules, or file structure docs (move those to separate files and link from AGENTS.md)
- [ ] Each instruction file is under ~200 lines (beyond this, compliance measurably degrades)
- [ ] Tool-specific instruction files exist for each agent tool in use: `AGENTS.md` is the shared source of truth (read by Cursor and GitHub Copilot); `CLAUDE.md` for Claude Code and `GEMINI.md` for Gemini CLI each import or symlink it
- [ ] `robots.txt` does not inadvertently block inference-time agent user-agents (if hosted docs exist)
- [ ] If the project exposes an MCP server, a `.mcp.json` configuration file exists so agents can connect without manual setup

## Content structure

- [ ] Documentation available as plain Markdown (not only rendered HTML)
- [ ] Consistent heading hierarchy (no skipped levels)
- [ ] Each doc page leads with what the reader will learn or accomplish
- [ ] Code examples appear immediately after the prose they illustrate
- [ ] Parameters, options, and flags use tables, not prose paragraphs
- [ ] No single doc page exceeds ~25K tokens

## Capability signaling

- [ ] Project purpose and capabilities are stated, not just described by implementation
- [ ] API endpoints, CLI commands, or library functions have usage examples
- [ ] Constraints (rate limits, auth requirements, platform limits) are documented near the relevant capability
- [ ] Error codes or failure modes are documented
