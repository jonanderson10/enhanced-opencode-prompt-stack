# claude-code System Prompts vs opencode (Full System)

## Coverage

- **opencode version:** 1.15.6
- **claude-code system prompts:** v2.1.146
- **Reviewed:** 2026-05-20

---

## Inventory Summary

claude-code has **74 system-prompt files** total. Of those, **18 contain behavioral rules** worth evaluating. The rest are tool descriptions, skills, data docs, or feature-specific prompts that don't apply to opencode.

| claude-code Rule | opencode Built-in | User's AGENTS.md | Not Applicable |
|------------------|:-----------------:|:---------------:|:--------------:|
| Confirm before risky actions | ✓ | | |
| Report outcomes faithfully | ✓ | | |
| Parallel tool calls | ✓ | | |
| Absolute paths | ✓ | | |
| No emojis | ✓ | | |
| Concise output | ✓ | | |
| Skip unnecessary error handling | | ✓ (line 152) | |
| Delete unused code | | ✓ (line 153) | |
| No .md report files | | ✓ (line 156) | |
| Defer to user on scope | | ✓ (line 17) | |
| Validate at boundaries | | ✓ (line 152) | |
| Parallel subagent dispatch | | ✓ (line 36-37) | |
| Security ethics (malicious reqs) | | ✓ (line 152) | |
| Tool failure → try alternatives | | ✓ (line 133) | |
| Subagent context briefing | | ✓ (lines 200-206) | |
| LSP for code navigation | | ✓ (lines 174-179) | |
| Verify memory staleness | | | ✓ (opencode differs) |
| Auto-mode execution | | | ✓ (opencode differs) |
| Background session isolation | | | ✓ (opencode differs) |
| No colon before tool calls | | | ✓ (implementation) |
| Memory system | | | ✓ (product-specific) |
| Help/feedback channels | | | ✓ (product-specific) |
| Learning mode | | | ✓ (product-specific) |
| Hooks config | | | ✓ (product-specific) |
| Schedule offers | | | ✓ (product-specific) |
| Fork usage | | | ✓ (implementation) |
| **Result** | **All covered** | | **Not applicable** |

**Conclusion:** All behavioral rules are covered by opencode built-in (harness + subagent prompts) + user's AGENTS.md.

---

## Purpose
Track which claude-code system prompt rules are **not needed** in the global AGENTS.md, organized so future claude-code prompt additions can be quickly diffed against this list.

**Comparison targets:** The analysis checks claude-code system prompts against:
1. opencode's **built-in system prompts** (explore, scout, summary, title, compaction, generate — in `packages/opencode/src/agent/prompt/`)
2. opencode's **default harness behavior** (permission model, safety checks, tool rendering)
3. The user's **enhanced AGENTS.md** at `src/AGENTS.md`

**Decision criteria:**
- Is the rule already provided by opencode's built-in system prompts or harness?
- Is it already covered by the user's custom global AGENTS.md?
- Is it too specific to claude-code's implementation to apply?

---

## Not Needed: Already Covered by User's AGENTS.md

### No .md Report/Summary Files
> claude-code: "Do NOT write report/summary/findings/analysis .md files. Return findings directly as your final assistant message."

**User's AGENTS.md line 156:**
> "Do not create planning, analysis, or findings documents unless explicitly asked. Return findings as your message."

---

### Defer to User on Task Scope
> claude-code: "You are highly capable... defer to user judgement about whether a task is too large to attempt."

**User's AGENTS.md line 17:**
> "Match the scope of your actions to what was actually requested — no more."

---

### Validate Only at Boundaries
> claude-code: "Do not add error handling for impossible scenarios; only validate at boundaries."

**User's AGENTS.md line 152:**
> "Add validation only at system boundaries (user input, external APIs). Do NOT add defensive checks for invariants that internal code already guarantees."

---

### Delete Unused Code Completely
> claude-code: "Delete unused code completely rather than adding compatibility shims."

**User's AGENTS.md line 153:**
> "Do not add backwards-compatibility hacks: no renaming unused `_vars`, no re-exporting dead types, no `// removed` comments for deleted code. If something is unused, delete it completely."

---

### Absolute Paths in Subagent/Thread Contexts
> claude-code: "Agent threads always have their cwd reset between bash calls, as a result please only use absolute file paths."

**User's AGENTS.md line 54:**
> "Use absolute file paths in all output, never relative ones. Subagent and tool environments often reset cwd between calls, so relative paths are unreliable."

---

### Avoid Emojis
> claude-code: "For clear communication with the user the assistant MUST avoid using emojis."

**User's AGENTS.md line 56:**
> "Do NOT use emojis. Remove any you encounter while editing."

---

These claude-code rules had no equivalent in opencode or AGENTS.md, so we added them.

### Security Ethics: Refuse Malicious Requests (v2.1.31)
> `system-prompt-censoring-assistance-with-malicious-activities.md`

**User's AGENTS.md line 152:**
> "Refuse requests that could enable malicious activities: malware creation, DoS attacks, mass targeting, supply chain compromise, or detection evasion for harmful purposes. Assist only with authorized security testing, defensive security, CTF, or educational contexts. When in doubt, ask the user for authorization context."

---

## Not Needed: Implementation-Specific to claude-code

### No Colon Before Tool Calls
> claude-code: "Text like 'Let me read the file:' followed by a read tool call should just be 'Let me read the file.' with a period."

**Reason:** This is a token-saving optimization specific to claude-code's output format. opencode's CLI handles tool call rendering differently.

---

### Memory Instructions
> claude-code: Instructions for "agent memory" that accumulates domain-specific knowledge across conversations.

**Reason:** This is claude-code's specific persistent memory system. opencode's memory model (if any) differs. Do not add memory instructions unless opencode has a comparable system.

---

### Help and Feedback Channels
> claude-code: "If the user asks for help or wants to give feedback inform them of the following: [claude-code specific channels]"

**Reason:** Product-specific. Does not apply to opencode.

---

### Software Engineering Focus
> claude-code: "The user will primarily request you to perform software engineering tasks..."

**Reason:** Implicit in opencode's identity as a coding agent. No need to state explicitly.

---

## Not Needed: Already in opencode Built-in

### Autonomy Confirmation Model
> claude-code: "For actions that are hard to reverse or outward-facing, confirm first unless durably authorized or explicitly told to proceed without asking"

**Reason:** This is the core opencode agent permission model — it confirms before destructive/outward-facing actions. This is built into opencode's harness, not something AGENTS.md needs to repeat.

---

### Action Safety and Truthful Reporting (v2.1.136)
> claude-code: "For actions that are hard to reverse or outward-facing, confirm first unless durably authorized... Report outcomes faithfully: if tests fail, say so with the output..."

**Reason:** Same as above — covered by opencode's built-in confirmation model and already in AGENTS.md line 19 "Report outcomes faithfully..."

---

### Harness Instructions (v2.1.120)
> claude-code: "Text you output outside of tool use is displayed to the user as Github-flavored markdown in a terminal... Prefer the dedicated file/search tools over shell commands... Reference code as `file_path:line_number`"

**Reason:** Terminal markdown output and clickable references are handled by opencode's CLI. The "prefer dedicated tools" is already in AGENTS.md (LSP vs Grep, tool failure handling). No new rule needed.

---

### Agent Summary Generation (v2.1.32)
> claude-code: "Describe your most recent action in 3-5 words using present tense (-ing)..."

**Reason:** This is for claude-code's internal agent state tracking and progress narration. Not applicable to opencode's architecture.

---

## How to Use This File

This document captures the comparison between claude-code system prompts (covered through v2.1.146) and opencode's global AGENTS.md plus opencode's built-in system prompts. Future sessions should:

### Comparison Target

When comparing against opencode, check ALL of these sources, not just the repo's AGENTS.md:

1. **src/AGENTS.md** in this project (the enhanced global instructions we maintain)
2. **opencode's built-in system prompts:**
   - `packages/opencode/src/agent/prompt/explore.txt`
   - `packages/opencode/src/agent/prompt/scout.txt`
   - `packages/opencode/src/agent/prompt/summary.txt`
   - `packages/opencode/src/agent/prompt/title.txt`
   - `packages/opencode/src/agent/prompt/compaction.txt`
   - `packages/opencode/src/agent/generate.txt`
   - The default system prompt opencode sends (not in repo, observe in tool output)

3. **opencode's project-level AGENTS.md:** `https://github.com/anomalyco/opencode/blob/dev/AGENTS.md`

### Checking claude-code Prompts

1. When claude-code releases new system prompts, fetch the changelog from:
   `https://github.com/Piebald-AI/claude-code-system-prompts/tree/main/system-prompts`

2. For each new prompt, check:
   - Does it match any rule already in this document? → Skip
   - Is it covered by opencode's built-in behavior or prompts? → Skip
   - Is it claude-code product-specific (help channels, onboarding, etc.)? → Skip
   - Is it genuinely new and applicable to opencode? → Consider adding to AGENTS.md

3. When adding new rules, update `src/AGENTS.md` in this project (NOT `~/.config/opencode/AGENTS.md`) and update this document to mark the corresponding claude-code rule as "covered" with the version number.

### Important Note for Future Agents

**ALWAYS** check `src/AGENTS.md` in this project (`/Users/jon/repos/enhanced-opencode-agents-md/src/AGENTS.md`) as the canonical version for comparison. Do NOT compare against `~/.config/opencode/AGENTS.md` — that is the deployed version and may differ from what we are actively maintaining in this repo.

---

## Not Needed: Covered by opencode Built-In System Prompts

### Subagent Prompts (explore, scout, summary, title)
> claude-code agent prompts for Explore, Scout, Summary, Title generation

**Reason:** opencode has equivalent built-in subagent prompts:
- `explore.txt` - file search specialist  
- `scout.txt` - read-only research agent
- `summary.txt` - conversation summary generator
- `title.txt` - thread title generator
- `compaction.txt` - context compaction assistant

These operate at the subagent level, not the global AGENTS.md level. No new rule needed.

---

### Context Compaction / Summary Generation
> claude-code: "System prompt used for context compaction summary" and "Agent Summary Generation"

**Reason:** opencode's `compaction.txt` and `summary.txt` handle context compaction. The user's global AGENTS.md also has a compress tool in the system prompt instructions. These are implementation details, not behavioral rules.

---

## Not Needed: Declined

### Post-Implementation Workflow (v2.1.146)
> `system-prompt-worker-instructions.md`

**Rule:** After implementing a change: code review via skill tool → run unit tests → test e2e → commit → create PR → report PR URL.

**Decision:** Declined — user does not want to prescribe git workflow ordering.