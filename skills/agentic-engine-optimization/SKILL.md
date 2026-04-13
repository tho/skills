---
name: agentic-engine-optimization
description: "This skill should be used when the user wants to make a project easier for AI agents to discover, understand, and use. Triggers include 'make this repo agent-friendly', 'add llms.txt', 'optimize for AI agents', 'add AGENTS.md', 'improve AGENTS.md', 'audit my repo for AI readiness', 'check my project for agent readiness', 'AEO audit', or any request about improving agent discoverability for repos, APIs, libraries, or CLI tools."
---

# Agentic Engine Optimization (AEO)

Structure project documentation so AI coding agents can discover, evaluate, and use the project with minimal friction. AEO applies to any kind of project: library, API, CLI tool, framework, or service.

## Workflow

1. **Understand the project.** Read the README, existing docs, and project structure. Identify what the project is, who it is for, and what problems it solves. If the request is open-ended (e.g. "audit my repo"), briefly confirm the desired scope with the user before proceeding.
2. **Audit current state.** Read `references/audit-checklist.md`, then apply each item to the project to identify gaps.
3. **Plan changes.** Propose a focused set of improvements ranked by impact. Prefer small, high-leverage additions over sweeping rewrites. Confirm the proposed changes with the user before implementing.
4. **Implement.** Read `references/files.md`. Apply changes file by file, following the guidance for each file type.

## Principles

- **Agents compress navigation into one or two HTTP requests.** They do not click around, render JS, or read visual layout. Every important fact must be reachable from a plain-text entry point.
- **Context is finite.** Large documents risk blowing an agent's context window, triggering truncation or hallucination. Keep pages concise and surface token costs where possible.
- **Signal capability, not just content.** Agents need to know what a project *can do* before deciding whether to read further. Lead with outcomes, not implementation details.
- **Serve both audiences.** Good AEO does not hurt human readers. Markdown with clear headings, tables for parameters, and code examples right after prose works for everyone.

## Notes

- Scale additions to the project's complexity. A small CLI tool may only need a better README; a large API platform may benefit from OpenAPI Specification, llms.txt, AGENTS.md, and structured capability docs.
- Do not rewrite existing documentation wholesale. Improve structure and add agent entry points alongside what already exists.
- When the project already has strong docs, the work is often just adding an index (llms.txt) and a capability summary, not rewriting content.
