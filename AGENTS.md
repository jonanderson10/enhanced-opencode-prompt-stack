# Repository: Enhanced OpenCode Prompt Stack

A provider-agnostic prompt replacement for OpenCode's built-in `build` and `plan` agent prompts, paired with a workflow-layer `AGENTS.md` for safety, verification, git hygiene, and code quality.

## Repo Structure

- `src/prompts/custom.txt` — shared base prompt for all providers (replaces OpenCode's `gpt.txt`, `kimi.txt`, `anthropic.txt`, `default.txt`)
- `src/prompts/build-specific.txt` — build-mode additions
- `src/prompts/plan-specific.txt` — plan-mode additions
- `src/AGENTS.md` — the **distributed AGENTS.md** that ships to users (copied to `~/.config/opencode/AGENTS.md` or project root)
- `docs/research.md` — research sources and how they map to prompt files
- `README.md` — canonical project documentation

## Key Facts for Agents

- **No build, test, lint, or CI pipeline.** This repo is prompt and markdown content only. There are no automated checks to run.
- **`src/AGENTS.md` is the product.** Edits to it affect what all downstream users receive. Understand the three-layer architecture (base prompt, mode prompt, workflow instructions) before modifying it.
- **Verification is manual.** After editing prompts, the user confirms with `opencode debug agent build` and `opencode debug agent plan` to inspect resolved agent prompts. There is no script to run.
- **No code to typecheck or lint.** The `.txt` prompt files and `.md` files have no automated format enforcement. Match the existing style: concise, direct, no fluff.
- **The global `AGENTS.md` at `~/.config/opencode/AGENTS.md` may differ from `src/AGENTS.md`.** The global one is the installed copy; `src/AGENTS.md` is the source of truth for this repo.
- **`.claude/settings.local.json` exists** — Claude Code settings are present. Respect them.

## Editing Conventions

- Prompt files (`src/prompts/*.txt`) are terse and instruction-dense. Preserve that style.
- `src/AGENTS.md` is structured in sections with horizontal rules. Add new rules within the appropriate section; don't create new top-level sections without reason.
- When changing prompt behavior, consider all three files: the change may belong in `custom.txt` (universal) or in a mode-specific file (build or plan only).
- `README.md` documents the architecture, installation, and rationale. Update it if your change alters how the system works.
