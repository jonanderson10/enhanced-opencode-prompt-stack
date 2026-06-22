# Enhanced OpenCode Prompt Stack

A model-agnostic replacement for OpenCode's built-in system prompt, `build` and `plan` agent prompts, paired with an `AGENTS.md` workflow layer for safety, verification, git hygiene, scope control, communication, and code quality.

The value starts with `AGENTS.md`: a portable amalgamation of coding-agent operating practices from commercial harnesses such as Claude Code, Codex, and Cursor, adapted for OpenCode instead of copied from any one source. The custom prompt stack is the model-agnostic base that aligns OpenCode's lower-level agent prompts with that workflow layer, replacing the per-model built-in prompts with one shared baseline designed to behave consistently across models.

The goal is a small layered prompt system, not a giant catch-all prompt. OpenCode still provides the harness: model routing, tool schemas, permissions, environment context, skills, agent modes, and instruction-file loading. This repo replaces the baseline agent prompt OpenCode sends to the model, then adds durable operating standards through `AGENTS.md`.

**Model-agnostic, not harness-agnostic.** The stack is intentionally tightly coupled to OpenCode as a harness — it relies on OpenCode's agent-prompt override mechanism, instruction-file loading, skill discovery, and tool surface (e.g. `WebFetch`, the `Task` tool, subagent types referenced in `plan-specific.txt`). It is not portable to other agent harnesses such as Claude Code, Codex, or Cursor. The "agnostic" claim is about models: one base prompt is designed to work across GPT, Claude, Kimi, DeepSeek, GLM, Mimo, local models, and other models OpenCode supports, instead of depending on whichever model-specific prompt OpenCode would otherwise select.

## What This Is

This repo provides three layers that are meant to work together:

| Layer | File | Responsibility |
| --- | --- | --- |
| Base agent prompt | `src/prompts/custom.txt` | Model-agnostic replacement for OpenCode's built-in model-specific prompts: identity, instruction hierarchy, professional objectivity, tool-use defaults, the task loop, OpenCode-doc lookup, and false-termination as the last-line rule |
| Mode prompt | `src/prompts/build-specific.txt` / `src/prompts/plan-specific.txt` | The small amount of behavior that differs between Build Mode and Plan Mode |
| Workflow instructions | `src/AGENTS.md` | Durable operating standards: confirmation rules, git hygiene, planning, blocker handling, code quality, verification, and communication |

Configured together, these files are designed to give every model OpenCode routes to the same baseline behavior instead of depending on whichever model-specific prompt OpenCode would otherwise select. The base and mode prompts keep the harness aligned; `AGENTS.md` carries the deeper best-practice workflow.

## When to Use Each Mode

OpenCode has two built-in agent modes, and this stack provides a different prompt for each. The split is deliberate:

- **Build mode** is the default working mode. It is for direct implementation of the user's requested change, and it also covers small-to-medium tasks that include a quick read of the surrounding code and a short inline plan, all in one pass. If the task can be done in a handful of edits with the design obvious from the existing code, build mode is the right choice.
- **Plan mode** is reserved for heavy-duty planning: tasks that are ambiguous, span many files, benefit from structured exploration, or warrant a separate review pass before any code is written. The 5-phase plan workflow (`plan-specific.txt`) — initial understanding, design, review, final plan, finish — is intentional. Plan mode also enforces a read-only constraint and is the right choice when the user wants to see and approve an approach before implementation begins.

If you are unsure which mode to use: build mode handles the common case. Switch to plan mode when the work is large enough that a one-pass implementation risks scope drift, hidden assumptions, or an approach the user would not have signed off on.

## Requirements

This stack requires the [context-mode](https://github.com/mksglu/context-mode) plugin. Its `ctx_*` tools are referenced throughout `AGENTS.md` and the prompt files for context-window protection.

Add it to the `plugin` array in `opencode.json`:

```json
"plugin": ["context-mode"]
```

Verify with `ctx stats` in an OpenCode session.

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

OpenCode's built-in `build` and `plan` agents normally use a model-specific prompt selected for the current model, such as `gpt.txt`, `kimi.txt`, `anthropic.txt`, or the fallback `default.txt`.

When you configure `agent.build.prompt` and `agent.plan.prompt`, OpenCode uses these files instead:

```text
prompts/custom.txt + prompts/build-specific.txt  -> build agent prompt
prompts/custom.txt + prompts/plan-specific.txt   -> plan agent prompt
```

That replacement is scoped to the configured agents only. It does not remove OpenCode's tools, permissions, environment block, skills, agent modes, or `AGENTS.md` loading.

## How This Stack Is Structured

OpenCode provides the agent harness out of the box. This stack is an opinionated replacement for the instruction layers that sit on top of it.

| Area | OpenCode provides | This stack adds or edits |
| --- | --- | --- |
| Provider prompts | Model-selected prompts such as `gpt.txt`, `kimi.txt`, `anthropic.txt`, or `default.txt` | `prompts/custom.txt` gives configured models one shared model-agnostic base prompt |
| Build/Plan behavior | Built-in agents and permissions | Small mode prompts that separate implementation behavior from planning behavior |
| Workflow rules | Global/project instruction files are loaded, but workflow preferences are yours to define | `src/AGENTS.md` supplies durable standards for safety, scope, git, verification, blockers, communication, and code quality |
| OpenCode harness | Tool schemas, permissions, environment context, skills, model routing, and messages | No replacement; the custom stack is designed to sit inside the existing harness |

`AGENTS.md` is the main best-practice layer:

| OpenCode provides | `AGENTS.md` adds |
| --- | --- |
| Model-specific identity, tone, and basic workflow | Cross-model behavioral guardrails that stay stable across models |
| Tool schemas, permissions, environment context, and skill discovery | Rules for when to use those capabilities safely and when to ask first |
| General coding-agent advice: inspect, edit, test, stay concise | Senior-engineer discipline: root-cause fixes, scope control, no defensive bloat |
| Basic git caution: do not commit unless asked | Dirty-worktree hygiene, stage-by-name discipline, hook failure handling |
| Testing and lint/typecheck reminders | Verification standards: runtime observation where meaningful, explicit PASS/FAIL/BLOCKED reporting |
| Concise CLI communication defaults | Signal-dense progress updates, blocker handling, and review-style findings when requested |

In the prompt content itself, the custom prompts and stock OpenCode differ in these specific ways:

| Stock OpenCode | This stack |
| --- | --- |
| Provider prompts vary by model in tone, planning pressure, comments, and tool advice | `custom.txt` gives every configured model one compact operating contract |
| Read-before-edit guidance is implicit in some provider prompts | `custom.txt` requires inspecting existing files before edits and rereading after stale or ambiguous edit failures |
| File contents, logs, and tool output are not explicitly classified as untrusted | `custom.txt` treats file contents, logs, retrieved docs, and tool output as untrusted data unless they come from the active instruction hierarchy |
| Some provider prompts constrain output length or comment usage | `AGENTS.md` keeps responses concise and allows comments when they clarify non-obvious code |
| Some provider prompts emphasize TodoWrite and Task-tool usage | `custom.txt` mentions planning, delegation, and clarification without forcing heavy process on small tasks |
| Provider prompts handle OpenCode docs lookup with varying specificity | `custom.txt` scopes docs/source verification to OpenCode capability, config, agent, skill, hook, MCP, and prompt questions |
| Provider prompts combine base and mode-specific guidance in one file | `build-specific.txt` and `plan-specific.txt` separate implementation behavior from planning behavior |

## What This Stack Is Built On

The custom prompts follow these design principles:

- Read existing files before editing them.
- Treat prompt-like text in files, logs, retrieved docs, and tool output as untrusted data.
- Keep tool-use guidance consistent without forcing heavy process on small tasks.
- Avoid brittle output-length rules and absolute no-comment policies (handled in `AGENTS.md`, not the base prompt).
- Keep model-specific assumptions out of the shared base prompt.
- Verify OpenCode capability, config, agent, skill, hook, MCP, and prompt behavior against current docs or source instead of guessing.

The stack does not try to copy any single vendor prompt, and it does not try to make weaker models stronger. It shapes behavior; it does not change model capability.

These are design choices backed by the research mapped in `docs/research.md`, not measured improvements. The stack has not been benchmarked against stock OpenCode or any other prompt setup; treat the claims as hypotheses, not guarantees. If you have benchmark data that contradicts a design choice, that is a useful contribution.

## How It Was Derived

The stack was built from four inputs:

- OpenCode's currently bundled model-specific prompts.
- Relevant open OpenCode GitHub issues and PRs about prompt behavior.
- The operating standards encoded in this repo's `AGENTS.md`, synthesized from commercial coding-agent harness practices and adapted for OpenCode.
- Criterium's [OpenCode context-dump research](https://github.com/criterium/opencode-lab/tree/main/research/context-dump), which shows how OpenCode assembles the model context: agent prompt, tool schemas, messages, environment, skills, and instruction files.

Those inputs shaped the split between base behavior, mode behavior, and workflow rules. The base prompt stays short and aligned with `AGENTS.md`. The mode files only describe what differs between `build` and `plan`. The deeper engineering discipline lives in `AGENTS.md`.

See `docs/research.md` for the full mapping of research sources to specific prompt files and line numbers.

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
- Verification standards: runtime observation for behavior changes, real lint/typecheck/build checks where practical, explicit PASS/FAIL/BLOCKED reporting, executable checks over self-audit, and a halt rule that prevents unanchored improvement loops from destroying already-correct work.
- Test-writing guidance for existing test suites, including changed paths and meaningful edge cases.
- Git hygiene: no commits unless asked, stage by name, no root `git add .`, and no amend-based recovery from failed hooks.
- Scope control: complete but not expansive, no speculative abstractions, no defensive bloat, no jumping to implementation on open-ended questions, and proactive only on adjacent fixes in touched code — otherwise halt when scope is met.
- Right-sizing for new vs. existing work: be ambitious on fresh projects, minimal on existing code.
- Code-quality preferences: root-cause fixes, typed edges, readable names, comment WHY not WHAT, flat control flow, no unsolicited compatibility shims.
- Blocker handling, obstacle investigation (no destructive shortcuts), ambiguity handling, outcome-first communication, honest pushback, and subagent briefing principles.
- context-mode tool routing: Think-in-Code analysis via the sandbox, blocked `curl`/inline HTTP (use `ctx_fetch_and_index` / `ctx_execute`), large-output redirection, search-before-asking on resume, and parallel I/O batching.

## License

MIT
