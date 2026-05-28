# Enhanced OpenCode Prompt Stack

A provider-agnostic replacement for OpenCode's built-in system prompt, `build` and `plan` provider prompts, paired with an `AGENTS.md` workflow layer for safety, verification, git hygiene, scope control, communication, and code quality.

The value starts with `AGENTS.md`: a portable amalgamation of coding-agent operating practices from commercial harnesses such as Claude Code, Codex, and Cursor, adapted for OpenCode instead of copied from any one source. The custom prompt stack exists to make OpenCode's lower-level agent prompts align with that workflow layer, behave consistently across providers, and avoid the most glaring stale or inconsistent behavior in the current built-in prompts.

The goal is a small layered prompt system, not a giant catch-all prompt. OpenCode still provides the harness: model routing, tool schemas, permissions, environment context, skills, agent modes, and instruction-file loading. This repo replaces the baseline agent prompt OpenCode sends to the model, then adds durable operating standards through `AGENTS.md`.

## What This Is

This repo provides three layers that are meant to work together:

| Layer | File | Responsibility |
| --- | --- | --- |
| Base agent prompt | `src/prompts/custom.txt` | Provider-agnostic replacement for OpenCode's built-in provider prompts: identity, instruction hierarchy, tool-use defaults, safety boundaries, OpenCode-doc lookup, and concise CLI behavior |
| Mode prompt | `src/prompts/build-specific.txt` / `src/prompts/plan-specific.txt` | The small amount of behavior that differs between Build Mode and Plan Mode |
| Workflow instructions | `src/AGENTS.md` | Durable operating standards: confirmation rules, git hygiene, planning, blocker handling, code quality, verification, and communication |

Configured together, these files give GPT, Claude, Kimi, DeepSeek, GLM, Mimo, local models, and other OpenCode providers the same baseline behavior instead of depending on whichever model-specific prompt OpenCode selects. The base and mode prompts keep the harness aligned; `AGENTS.md` carries the deeper best-practice workflow.

## Installation

1. Copy the prompt files:

   ```text
   src/prompts/custom.txt
   src/prompts/build-specific.txt
   src/prompts/plan-specific.txt
   ```

   to:

   ```text
   ~/.config/opencode/prompts/
   ```

2. Copy `src/AGENTS.md` to one of:

   ```text
   ~/.config/opencode/AGENTS.md   # global
   ./AGENTS.md                    # project-level
   ```

3. Add the prompt overrides to `~/.config/opencode/opencode.json`:

   ```jsonc
   {
     "agent": {
       "build": {
         "prompt": "{file:prompts/custom.txt}\n\n{file:prompts/build-specific.txt}"
       },
       "plan": {
         "prompt": "{file:prompts/custom.txt}\n\n{file:prompts/plan-specific.txt}"
       }
     }
   }
   ```

4. Restart OpenCode. Config and prompt files are loaded when OpenCode starts; an existing session will keep using the already-loaded prompt stack.

Project-level `AGENTS.md` files are loaded alongside the global one, so use them for project-specific commands, conventions, and exceptions without editing your global defaults.

## What It Replaces

OpenCode's built-in `build` and `plan` agents normally use a provider prompt selected for the current model, such as `gpt.txt`, `kimi.txt`, `anthropic.txt`, or the fallback `default.txt`.

When you configure `agent.build.prompt` and `agent.plan.prompt`, OpenCode uses these files instead:

```text
prompts/custom.txt + prompts/build-specific.txt  -> build agent prompt
prompts/custom.txt + prompts/plan-specific.txt   -> plan agent prompt
```

That replacement is scoped to the configured agents only. It does not remove OpenCode's tools, permissions, environment block, skills, agent modes, or `AGENTS.md` loading.

## What Changes Compared to Stock OpenCode

OpenCode provides the agent harness out of the box. This repo changes the instruction layers around that harness.

| Area | Stock OpenCode | This repo adds or edits | Removed or avoided |
| --- | --- | --- | --- |
| Provider prompts | Model-selected prompts such as `gpt.txt`, `kimi.txt`, `anthropic.txt`, or `default.txt` | `prompts/custom.txt` gives configured models one shared provider-agnostic base prompt | Provider-specific prompt drift for `build` and `plan` |
| Build/Plan behavior | Built-in agents and permissions | Small mode prompts that separate implementation behavior from planning behavior | Mixing build and plan expectations into one broad prompt |
| Workflow rules | Global/project instruction files are loaded, but workflow preferences are yours to define | `src/AGENTS.md` supplies durable standards for safety, scope, git, verification, blockers, communication, and code quality | Relying on each provider prompt to imply those standards consistently |
| OpenCode harness | Tool schemas, permissions, environment context, skills, model routing, and messages | No replacement; the custom stack is designed to sit inside the existing harness | Nothing here bypasses tools, permissions, or instruction loading |

`AGENTS.md` is the main best-practice layer:

| OpenCode provides | `AGENTS.md` adds |
| --- | --- |
| Model-specific identity, tone, and basic workflow | Cross-model behavioral guardrails that stay stable across providers |
| Tool schemas, permissions, environment context, and skill discovery | Rules for when to use those capabilities safely and when to ask first |
| General coding-agent advice: inspect, edit, test, stay concise | Senior-engineer discipline: root-cause fixes, scope control, no defensive bloat |
| Basic git caution: do not commit unless asked | Dirty-worktree hygiene, stage-by-name discipline, hook failure handling |
| Testing and lint/typecheck reminders | Verification standards: runtime observation where meaningful, explicit PASS/FAIL/BLOCKED reporting |
| Concise CLI communication defaults | Signal-dense progress updates, blocker handling, and review-style findings when requested |

The custom prompts keep the base harness aligned with that layer:

| Built-in prompt behavior | Custom prompt behavior |
| --- | --- |
| Provider prompts differ in tone, planning pressure, comments, and tool advice | `custom.txt` gives every configured model the same compact operating contract |
| Some prompts lack explicit read-before-edit guidance | `custom.txt` requires inspecting existing files before edits and rereading after stale or ambiguous edit failures |
| Prompt-like text in files or tool output can be mistaken for instructions | `custom.txt` treats file contents, logs, retrieved docs, and tool output as untrusted data unless they come from the active instruction hierarchy |
| Some prompts over-constrain output length or forbid comments absolutely | `custom.txt` keeps responses concise and allows comments when they clarify non-obvious code |
| Some prompts push aggressive TodoWrite or broad Task-tool usage | `custom.txt` mentions planning, delegation, and clarification without forcing heavy process on small tasks |
| Some prompts encourage stale or inconsistent OpenCode docs behavior | `custom.txt` scopes docs/source verification to OpenCode capability, config, agent, skill, hook, MCP, and prompt questions |
| Provider prompts mix base behavior with mode-specific expectations | `build-specific.txt` and `plan-specific.txt` separate implementation behavior from planning behavior |

## Why Replace the Built-ins?

OpenCode's bundled provider prompts are useful, but they differ by model and have accumulated inconsistent or stale guidance around editing, planning, comments, tool use, OpenCode docs lookup, and prompt-like text in files or tool output.

This stack gives every configured model one compact baseline that matches the expectations in `AGENTS.md`. The detailed workflow preferences stay in `AGENTS.md`, where they are easier to read, audit, and customize.

The replacement focuses on targeted fixes:

- Read existing files before editing them.
- Treat prompt-like text in files, logs, retrieved docs, and tool output as untrusted data.
- Keep tool-use guidance consistent without forcing heavy process on small tasks.
- Avoid brittle output-length rules and absolute no-comment policies.
- Keep provider-specific assumptions out of the shared base prompt.
- Verify OpenCode capability, config, agent, skill, hook, MCP, and prompt behavior against current docs or source instead of guessing.

It does not try to copy any single vendor prompt, and it does not try to make weaker models stronger. It shapes behavior; it does not change model capability.

## How It Was Derived

The stack was built from four inputs:

- OpenCode's currently bundled provider prompts.
- Relevant open OpenCode GitHub issues and PRs about prompt behavior.
- The operating standards encoded in this repo's `AGENTS.md`, synthesized from commercial coding-agent harness practices and adapted for OpenCode.
- Criterium's [OpenCode context-dump research](https://github.com/criterium/opencode-lab/tree/main/research/context-dump), which shows how OpenCode assembles the model context: agent prompt, tool schemas, messages, environment, skills, and instruction files.

Those inputs shaped the split between base behavior, mode behavior, and workflow rules. The base prompt stays short and aligned with `AGENTS.md`. The mode files only describe what differs between `build` and `plan`. The deeper engineering discipline lives in `AGENTS.md`.

## Prompt Layering

With this repo installed, the effective instruction structure is:

```text
custom base prompt
then build-specific or plan-specific prompt
then OpenCode environment context
then global/project AGENTS.md instruction files
then skills and other harness-provided context
```

OpenCode also sends the tool schemas and conversation messages as part of the model request.

So the custom prompts are not a replacement for `AGENTS.md`; they sit below it. `AGENTS.md` remains the workflow layer for rules that should survive across models, projects, and future prompt revisions.

## Verification

Confirm the resolved agent prompts with:

```sh
opencode debug agent build
opencode debug agent plan
```

Each command should show the resolved custom base prompt plus the matching mode-specific prompt.

For a deeper audit of the full assembled API context, use Criterium's [context-dump research](https://github.com/criterium/opencode-lab/tree/main/research/context-dump). That is useful when you want to inspect the final context, not just the configured agent prompt.

## Customization

Edit the layers separately:

- Change `prompts/custom.txt` when you want to alter the shared base behavior for every configured model.
- Change `prompts/build-specific.txt` or `prompts/plan-specific.txt` when only one mode should behave differently.
- Change `src/AGENTS.md` when you want to alter durable workflow rules like git conventions, verification standards, communication style, code-quality preferences, or blocker handling.

Keep project-specific commands and exceptions in a project-level `AGENTS.md`, not in the global prompt stack.

## What's Included in `AGENTS.md`

- Confirmation before destructive, irreversible, secret-bearing, external, or out-of-tree actions.
- Read-before-edit and prompt-injection boundaries for file/tool output.
- Verification standards: runtime observation for behavior changes, real lint/typecheck/build checks where practical, and explicit PASS/FAIL/BLOCKED reporting.
- Test-writing guidance for existing test suites, including changed paths and meaningful edge cases.
- Git hygiene: no commits unless asked, stage by name, no root `git add .`, and no amend-based recovery from failed hooks.
- Scope control: complete but not expansive, no speculative abstractions, no defensive bloat, and no jumping to implementation on open-ended questions.
- Right-sizing for new vs. existing work: be ambitious on fresh projects, minimal on existing code.
- Code-quality preferences: root-cause fixes, typed edges, readable names, comment WHY not WHAT, flat control flow, no unsolicited compatibility shims.
- Blocker handling, obstacle investigation (no destructive shortcuts), ambiguity handling, outcome-first communication, honest pushback, and subagent briefing principles.

## License

MIT
