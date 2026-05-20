# Enhanced OpenCode AGENTS.md

A production-grade `AGENTS.md` for [OpenCode](https://opencode.ai) that ports the best behavioral guardrails and workflow patterns from Claude Code's system prompt and MiniMax Token Plan best practices into OpenCode's agent instruction format.

## What This Is

OpenCode reads `AGENTS.md` to govern how its AI agent behaves — parallel tool use, confirmation gates, communication style, code quality rules, and more. This file distills the Claude Code defaults into explicit, portable instructions your agent will actually follow.

**What's included:**

- Reversibility checks and blast-radius awareness before destructive actions
- Parallel tool call discipline (independent calls always in the same message)
- Terse, signal-dense communication style with no emoji
- Task tracking rules (when to use todos vs. when not to)
- Blocker handling — surface, don't silently workaround
- External knowledge tool hierarchy (Context7 → web search → web fetch)
- Subagent prompting principles
- Code quality rules: no defensive over-engineering, no backward-compat shims, no what-comments
- Self-review checklist after every implementation
- Verification protocol (runtime observation, not just tests)
- Git conventions (Conventional Commits, stage by name, pre-commit hook footgun warning)

## Requirements

Two MCP plugins must be installed and configured for the full `AGENTS.md` to work — specifically the `ctx_fetch_and_index` / `ctx_search` and `context7` tool references.

### 1. context-mode

Indexes fetched documentation into a local searchable knowledge base so the agent can query it on-demand without re-fetching pages mid-task.

**Repo:** https://github.com/mksglu/context-mode

Follow the install instructions in that repo to register it as an MCP server in your OpenCode config.

### 2. Context7

Provides up-to-date library and SDK documentation (method signatures, parameters, usage patterns) fetched directly from source rather than relying on training data.

**Repo:** https://github.com/upstash/context7

Follow the install instructions to register `context7` as an MCP server.

## Installation

1. Copy `src/AGENTS.md` from this repo into `~/.config/opencode/AGENTS.md` (global) or into the root of any project (project-level).
2. Install context-mode and Context7 per their respective READMEs.
3. Open a new OpenCode session — the agent will pick up the instructions automatically.

Project-level `AGENTS.md` files take precedence over the global one, so you can override specific sections per-project without touching the global defaults.

## Customization

The file is plain markdown with clearly labeled sections. Strip or replace any section that doesn't fit your workflow. Common edits:

- **Git conventions** — swap out Conventional Commits for your team's format
- **External knowledge tools** — remove the context-mode section if you're not using that plugin
- **Communication style** — adjust verbosity rules to taste

## Why Not Just Use Claude Code?

Claude Code embeds these behaviors natively. OpenCode doesn't — it's more of a blank slate. This file makes OpenCode behave closer to Claude Code's defaults while keeping you on whichever model or provider OpenCode is configured to use.

## License

MIT
