# Research: System Prompt Best Practices for Code Harnesses

This document maps academic and industry research findings to the prompt files in this project. It explains *why* each file says what it says, grounded in evidence rather than intuition.

## Project Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/prompts/custom.txt` | Shared base prompt for all providers | 48 |
| `src/prompts/build-specific.txt` | Build-mode additions | 11 |
| `src/prompts/plan-specific.txt` | Plan-mode additions | 72 |
| `src/AGENTS.md` | Distributed workflow instructions | 127 |

---

## 1. Negation Processing

**Core finding:** Naming forbidden concepts activates them rather than suppressing them.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Semantic Gravity Wells" (2601.08070) | arXiv preprint | 87.5% priming failure rate across 40K samples. Naming forbidden words activates them in middle-layer attention. |
| "When Prohibitions Become Permissions" (2601.21433) | arXiv preprint | 77% endorsement of prohibited actions under simple negation across 16 models. Commercial models show 19-128% polarity swings. |
| "Don't Think of the White Bear" (2511.12381) | NeurIPS 2025 Workshop | Ironic rebound confirmed via circuit tracing; middle-layer amplification of forbidden tokens. |

### Where this applies

**`custom.txt`** — Behavioral rules use declarative framing throughout:
- Line 13: URLs policy stated as what *is* done, not what is forbidden
- Line 47: Library verification stated as a process, not a prohibition
- Line 50: Secrets handling stated as a property of output, not a command
- Line 56: Test framework determined by codebase inspection, not assumed
- Line 58: Commits framed as requiring explicit request
- Lines 84-86: Critical Reminders stated as conditions, not negations

**`AGENTS.md`** — Non-negotiables use declarative framing:
- Line 9: "Irreversible actions require confirmation"
- Line 10: "Git operations are opt-in and precise"
- Line 12: "Unverified work is never presented as fact"
- Line 89: "Code is reviewed for command injection, XSS, SQL injection..."

### Confidence

**Medium.** The negation failure mechanism is strong (activation patching, n=40K). The declarative framing improvement is demonstrated in a cross-linguistic context (22 probes, 4 models); English-only benefit is extrapolated. Commercial models are less affected than open-source models.

### Caveats

- The Social Register paper (2603.25015) tested cross-linguistic topology, not English-only prompts. Spillover effects are shown across languages, not within English.
- "When 'Better' Prompts Hurt" (2601.22025) shows generic prompt additions can degrade performance on specific tasks. Declarative rewrites could theoretically change behavior in unexpected ways.

---

## 2. Position Effects

**Core finding:** Instructions at the beginning and end of a prompt receive more weight than those in the middle (primacy and recency effects).

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Lost in the Middle" (Liu et al., TACL 2023) | Peer-reviewed | U-shaped performance curves across context positions. Foundational paper on position bias. |
| "Order Matters" (Zeng et al., Findings of ACL 2025) | Peer-reviewed | Hard-to-easy constraint ordering generalizes across architectures (24K samples). |
| "Confirmation, Framing, and Position Biases" (CHIIR 2026, ACM) | Peer-reviewed | LLMs favor initial or prominent elements within a prompt. |
| "Lost in the Middle at Birth" (2603.10123) | arXiv preprint | U-shape is an inherent geometric property of causal decoders, present at initialization. |

### Where this applies

**`custom.txt`** — Critical rules are positioned at the top and bottom of the file:
- Lines 1-8: Identity, Instruction Layers, and conflict resolution (top)
- Lines 83-86: Critical Reminders covering confirmation, commits, and verification (bottom)

### Confidence

**High.** Position bias is well-established across multiple papers. The "Lost in the Middle at Birth" finding (arxiv 2603.10123) demonstrates the U-shape is an *architectural* property of causal decoders with residual connections — present at initialization before training. This is not a learned bias that can be trained away; it is a geometric consequence of the architecture. The specific prescription "critical rules at top and bottom" is a direct inference from this architectural constraint.

### Caveats

- Position bias is an inherent geometric property of causal decoders (Lost in the Middle at Birth). It cannot be eliminated through prompt engineering or fine-tuning, only worked around.
- The U-shaped curve means middle content is weakest, but the exact drop-off varies by model and context length.
- The "Order Matters" paper studied constraint ordering difficulty (hard-to-easy), not importance-based positioning — related but distinct claims.

---

## 3. Constraint Overload

**Core finding:** Compliance degrades as the number of constraints increases. Models struggle with more than ~10 constraints per prompt.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "MulDimIF" (2505.07591) | arXiv preprint | Accuracy drops from 80.82% (Level I) to 36.76% (Level IV) as constraint count increases. 9,106 samples, 18 LLMs. |
| "RECAST" (2505.19030) | ICLR 2026 (accepted) | Models struggle with >10 constraints per prompt. 30K instances, 19 constraint types. |
| "Step-by-Step Mastery" (Findings of ACL 2025) | Peer-reviewed | DPO + curriculum learning improves soft constraint following. |

### Where this applies

**`AGENTS.md`** — Non-negotiables are consolidated into 8 rules (lines 9-16), merging related concerns:
- Line 9: Combines irreversible-action confirmation with git precision
- Line 12: Combines verification requirement with unverified-work handling

### Confidence

**Medium.** The general principle that more constraints reduce compliance is well-supported. The specific application (8 behavioral rules) is a reasonable inference. The threshold at which compliance degrades is not established by the cited papers for system-prompt-level rules.

### Caveats

- MulDimIF tested constraint counts in user instructions, not system-prompt-level behavioral rules. Applicability is uncertain.
- MulDimIF notes that training with multi-constraint data *improves* instruction following — the solution may be training, not prompt reduction.

---

## 4. Instruction Hierarchy

**Core finding:** Models fail to reliably enforce instruction priority. System/user prompt separation does not establish a reliable hierarchy.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Control Illusion" (AAAI 2026) | Peer-reviewed | System/user prompt separation fails across 6 frontier models. Social hierarchy framings show stronger influence. |
| "Many-Tier Instruction Hierarchy" (2604.09443) | arXiv preprint | Frontier models achieve ~40% accuracy on multi-tier conflicts. 853 tasks, 12 privilege levels, 46 agents. |
| "Where Instruction Hierarchy Breaks" (2606.07808) | arXiv preprint | Three failure stages: instruction identification, conflict resolution, response realization. Self-monitoring reduces non-compliance by 81-99%. |

### Where this applies

**`custom.txt`** — Instruction Layers section (lines 3-8):
- Lines 3-6: Describe the priority hierarchy (environment block > AGENTS.md > skills > tool descriptions > user messages)
- Line 8: Explicit conflict resolution — "When instructions from different layers conflict, the higher-priority source wins. If you cannot satisfy both, follow the higher-priority one and note the conflict to the user."

**`src/AGENTS.md`** — Line 3 establishes local priority: "Follow them on every turn; when context is tight, the Non-negotiables come first."

### Confidence

**High.** The strongest recommendation. Peer-reviewed AAAI paper with multiple corroborating sources. The research confirms the problem is real; the proposed fix is reasonable but may be insufficient — models fail to *execute* hierarchy, not to *understand* it.

### Caveats

- "Where IH Breaks" found that self-monitoring mechanisms reduce non-compliance by 81-99% — more actionable than just restating the hierarchy. Future work could add self-monitoring prompts.
- The Control Illusion paper found that social hierarchy framings work better than system/user separation. The current prompt uses a simple hierarchy description, not a social framing.

---

## 5. Code-First Output Order

**Core finding:** Generating code before explanation improves planning accuracy more than reasoning first.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Revisiting CoT in Code Generation" (ICML 2025) | Peer-reviewed | Code first then explanation is 9.86% more effective than reasoning first. |
| "CodeAgents" (2507.03254) | arXiv preprint | Pseudocode-style prompting improves planning by 3-36 percentage points and reduces tokens by 55-87%. |
| "RoutingGen" (AAAI 2026) | Peer-reviewed | Dynamic routing reduces tokens by 46%. |

### Where this applies

**`build-specific.txt`** — Line 9:
> "Understand the problem and codebase before writing. When writing code, prefer generating the implementation before explaining your reasoning — this helps some models and is optional, not a rule."

The guidance is a soft hint rather than a hard imperative because the underlying finding is a training-time result extrapolated to inference, and CoT effects are architecture-dependent.

### Confidence

**Low.** The research finding (code first → better in SFT data) is solid and peer-reviewed, but the application as an inference-time prompt instruction is an extrapolation from a training-time finding. CoT effects are architecture-dependent (LESS, BETTER): some models gain from CoT, others lose. A soft hint, not an imperative, is the appropriate weight for a shared base prompt across providers.

### Caveats

- "Revisiting CoT" is about supervised fine-tuning data ordering, not inference-time prompting. The 9.86% improvement comes from training models to generate code first.
- The "LESS, BETTER" paper shows CoT effects are architecture-dependent: Qwen gains +13.4% from CoT as a base model but loses -15.2% as an instruct model.
- The exploration caveat ("understand the problem and codebase before writing") prevents misinterpretation as "start coding immediately."

---

## 6. False-Termination and Halt Authority

**Core finding:** Coding agents declare done prematurely by *simulating* verification internally instead of running it. Reasoning models are worse. Without an external halt authority and criteria-in-view, agents told to "keep improving" destroy already-correct work.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Shepherd" (OpenReview, ZBOFr4ryBk) | Peer-reviewed (ICLR-scale) | FALSE-TERMINATION is a primary failure mode across 18 models / 3,908 SWE-Bench trajectories. Agents hallucinate verification rather than running it; reasoning models (QwQ) show significantly higher rates than non-reasoning (Qwen2.5). |
| "Goal-Autopilot" (arxiv 2606.11688) | arXiv preprint | Externalized gated FSM with hard floor against false terminal claims. No-False-Success theorem. Autopilot fabricates 0.95% vs StateFlow 25.05%. The mechanism is the gate, not the model — deterministic guardrails outperform prompt-only approaches. |
| "Vigil" (arxiv 2605.08747) | arXiv preprint | World completion vs self-termination separation. Models achieve task state but fail to convert into correct terminal reports. GPT-5.4: 54% unsupported commitment rate — models declare done without evidence. |
| "Halt Authority" (Armalo Labs, 2026-06) | Industry research | Told to keep improving verified-correct work, a reasoning model destroyed it 76% of the time when success criteria were out of view, 0% when criteria stayed in view (McNemar p=0.000244). The model's self-audit caught none of the damage; a deterministic keep-best gate prevented all of it. |
| "Check Yourself Before You Wreck Yourself" (OpenReview) | Peer-reviewed | "Simple quit" prompts (option to stop, no safety emphasis) gave only +0.17 safety; "specified quit" (explicit when/why) gave +0.39. Agents have a compulsion to act; mere permission to stop is insufficient. |
| "CaRT" (OpenReview, zHMwFgJsmk) | Peer-reviewed | Base models maintain consistently low termination tendencies regardless of information gathered — they fail to recognize when sufficient information has been obtained. |

### Where this applies

**`src/prompts/build-specific.txt`** — Rules covering:
- Acceptance criteria stated up front and kept in view while implementing (Armalo: 0% damage with criteria in view vs 76% without)
- Executable check preferred over self-audit; never claim success from reasoning alone (Shepherd: simulated verification is the named failure)
- Halt when scope is met and checks are green; do not keep iterating unprompted (Armalo halt authority)

**`src/prompts/custom.txt`** — Final Critical Reminder uses `YOU MUST` emphasis: run the real check and cite its output; simulated verification is a false-termination failure.

**`src/AGENTS.md`** — Verification section includes: "Verification is executed, not simulated. Citing a check you did not run is a false-termination failure." and a halt rule.

### Confidence

**High.** Shepherd is peer-reviewed with 3,908 trajectories across 18 models — the false-termination pattern is well-established. Goal-Autopilot's No-False-Success theorem and measured 0.95% vs 25.05% fabrication rate provides the strongest empirical validation of the gate-over-prompt approach. Vigil's 54% unsupported commitment rate in GPT-5.4 confirms the pattern persists in frontier models. Armalo's criteria-in-view result has a strong statistical signal (p=0.000244). The convergence across peer-reviewed, preprint, and industry sources is compelling.

### Caveats

- Goal-Autopilot and Vigil are arXiv preprints, not yet peer-reviewed. The directional findings are consistent with peer-reviewed Shepherd.
- Armalo tested one reasoning model on 17 outputs; the 76% figure is dramatic but narrow in sample. The criteria-in-view effect (0% vs 76%) is the robust part.
- "Check Yourself" is about safety quitting, not coding completion — the mechanism (compulsion to act) transfers, the domain does not directly.
- A prompt rule cannot substitute for a deterministic keep-best gate (Goal-Autopilot's and Armalo's strongest recommendation is a harness-level gate, not a prompt). The prompt rules here reduce incidence; the harness must enforce.

---

## 7. Acceptance Criteria and Work-Order Prompts

**Core finding:** Effective coding-agent prompts are work orders, not conversation. They carry four parts: goal, scope, acceptance criteria, stop condition. "State the outcome, not the steps." Give the agent a check it can run.

### Sources

| Source | Type | Finding |
|--------|------|---------|
| Anthropic Claude Code best practices (code.claude.com/docs) | Official docs | "State the outcome, not the steps." Give Claude a check it can run — "the difference between a session you watch and one you walk away from." |
| SurePrompts "Complete Guide to Prompting AI Coding Agents" (2026) | Industry guide | A spec has four parts: what to build, what to avoid, context (which files), acceptance criteria (when done). Transfers across Claude Code, Cursor, Aider, Devin. |
| eesel AI "How to prompt Claude Code" (2026) | Industry guide | Test-driven prompting works: write tests first, confirm they fail, implement until they pass, show evidence. If you've corrected Claude more than twice on the same issue, `/clear` and start fresh. |
| claudecodeguides "Prompt Engineering Tips" (2026) | Industry guide | Prompt anatomy: action+target, file path/scope, constraints/patterns, acceptance criteria. Anti-patterns: vague scope, too many tasks at once, no success criteria. |

### Where this applies

**`src/prompts/build-specific.txt`** — Includes: "Before coding, state the task's acceptance criteria. Keep them in view while implementing; if none were given, ask or define them first." This is the single highest-leverage rule per the Armalo criteria-in-view finding.

**`src/prompts/plan-specific.txt`** — Phase 4 (Final Plan) requires an explicit acceptance-criteria + stop-conditions section alongside the verification section. Anti-patterns include: declaring the plan complete without stating how "done" will be verified externally.

### Confidence

**Medium-High.** Convergent across Anthropic's own docs and three independent 2026 industry guides. The underlying mechanism (criteria-in-view) is backed by Armalo's controlled experiment. The specific prompt phrasing is an inference, not a measured result.

### Caveats

- Industry guides are practitioner reports, not controlled experiments. They describe what works, not why.
- "State the outcome, not the steps" can conflict with the code-first output order finding (section 5). The resolution: state the outcome up front, then generate code before explaining reasoning.

---

## 8. Anthropic CLAUDE.md Best Practices

**Core finding:** Anthropic's own guidance for CLAUDE.md (the reference implementation AGENTS.md derives from) gives concrete editorial rules: target under 200 lines; apply a cut test to every line; exclude self-evident practices; use emphasis tokens for adherence.

### Sources

| Source | Type | Finding |
|--------|------|---------|
| Anthropic "Best practices for Claude Code" (code.claude.com/docs) | Official docs | Target <200 lines. For each line ask: "Would removing this cause Claude to make mistakes? If not, cut it." Exclude self-evident practices ("write clean code"). Use "IMPORTANT" or "YOU MUST" to improve adherence. If Claude ignores a rule, the file is probably too long. |
| Anthropic "Using CLAUDE.md files" (claude.com/blog, 2025-11) | Official docs | CLAUDE.md becomes part of the system prompt every conversation. Keep concise. Break up information into separate markdown files and reference them. Each addition should solve a real problem encountered, not theoretical concerns. |
| llmbestpractices "System Prompts" (2026) | Industry guide | Hard cap 200–800 tokens for most agents. The last line gets the most attention — put the rule that must hold no matter what there. Treat the system prompt as code: commit, review, tag, run evals on every diff. |

### Where this applies

**`src/prompts/custom.txt`** — The file holds only universal, structural rules that define what the agent is (identity, instruction hierarchy, epistemic stance, capability routing, the task loop, tool policy, output format, and the false-termination rule at the last line). Preferences and policy that a user or project chooses — tone, proactiveness level, commit policy, secrets handling, code-comment style, irreversible-action confirmation — live in `src/AGENTS.md` where users can tune them per-project. The file reflects the cut test:
- No opencode /help or feedback URL block — meta, not behavioral
- No trivial examples illustrating conciseness — the rule is self-explanatory
- No "Security best practices are followed" sentence — self-evident per Anthropic's exclude list
- No tone, proactiveness, commit, secrets, or code-comments rules — all are preferences, all are covered in `src/AGENTS.md`
- `YOU MUST` emphasis on the single highest-stakes rule (false-termination)
- The false-termination rule at the literal last line (recency)

**`src/AGENTS.md`** — 127 lines, under the 200-line cap. Owns all preference-level rules: Communication (tone), Planning and scope (proactiveness), Non-negotiables (commit policy, irreversible-action confirmation, secrets handling), Code quality (code-comments policy, security review). The constraint-overload research (section 4) reinforces keeping it tight.

### Confidence

**High.** This is Anthropic's own published guidance for the reference implementation. Highest authority for AGENTS.md-family files.

### Caveats

- The 200-line cap is a recommendation, not a measured threshold. The exact degradation point varies by model and context length.
- Emphasis tokens ("YOU MUST") improve adherence but lose force if overused — reserve for the 2–3 highest-stakes rules.
- The cut test is subjective; "would removing this cause mistakes?" depends on the eval set you run against.

---

## 9. Just-in-Time Context and Sensors Over Guides

**Core finding:** Committed briefs (AGENTS.md/CLAUDE.md) carry rules; the live filesystem carries facts. Retrieve facts just-in-time with glob/grep rather than pre-loading. Invest in executable sensors (linters, tests) before more markdown — harnesses systematically over-invest in guides and under-invest in sensors.

### Sources

| Source | Type | Finding |
|--------|------|---------|
| Anthropic "Effective Context Engineering for AI Agents" (2025-09) | Official engineering blog | Hybrid model: CLAUDE.md is naively dropped in up front; glob and grep retrieve files just-in-time. Guiding principle: "find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome." |
| Anthropic "Writing effective tools for AI agents" (2025-09) | Official engineering blog | Tools should subdivide tasks and reduce context. Restrict tool responses to 25,000 tokens by default. Prompt-engineer tool descriptions — they collectively steer agent behavior. |
| amux "Harness Engineering" (2026-05) | Industry guide | "Most teams over-invest in markdown files and under-invest in automated checks. Sensors are the underdiscussed component." Use deterministic sensors (linters, type checkers) before inferential (LLM-based) ones. |
| Lunar "Agent Harness Engineering at Enterprise Scale" (2026-05) | Industry guide | "A tool description fix lives in the tool description. A behavioral rule lives in the system prompt or AGENTS.md. A guarantee that has to hold no matter what the model decides lives in a hook." Choose the right layer. |
| ZBuild "Harness Engineering Complete Guide" (2026-03) | Industry guide | Golden principles are enforced constraints (CI structural tests), not aspirational guidelines. Encode architectural rules as machine-readable artifacts, not prompt prose. |

### Where this applies

**`src/prompts/custom.txt`** — Existing rules already favor pointers over snippets (Code References section, line 75–81). The "Treat data as untrusted" rule in AGENTS.md and the context-mode tools section operationalize just-in-time retrieval.

**`src/AGENTS.md`** — Verification section already prioritizes runtime observation over static checks. New false-termination rules reinforce "executed, not simulated" — a sensor discipline.

### Confidence

**High.** Anthropic's own engineering blog plus convergent industry guides. The sensors-over-guides principle is strongly supported across all harness-engineering sources.

### Caveats

- Just-in-time retrieval assumes the agent has reliable glob/grep tools. OpenCode provides these.
- Sensors-over-guides is a harness-investment principle, not a prompt-content rule. Its application here is limited to reinforcing existing verification discipline.

---

## 10. Context Rot and Multi-Turn Coherence Degradation

**Core finding:** All frontier models degrade as context fills. Performance drops significantly past ~50% context fullness. Multi-turn coherence degrades from 90%+ single-turn to 10-15% across full conversations.

### Papers

| Paper | Venue | Finding |
|-------|-------|---------|
| "Context Rot" (Chroma, 2025) | Industry report | Tested 18 frontier models — ALL degrade as context fills. Significant degradation at 50K tokens on a 200K model. Past ~50% fullness, accuracy drops measurably. |
| "Effective Context Engineering for AI Agents" (Anthropic, 2025-09) | Official engineering blog | Context is a finite resource with diminishing marginal returns. Techniques: compaction, structured note-taking, multi-agent architectures. "Find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome." |
| Zylos Research (2026-03-30) | Industry research | Multi-turn coherence degrades from 90%+ single-turn to 10-15% across full conversations. Instruction adherence drops as conversation length increases. |

### Where this applies

**`src/AGENTS.md`** — Non-negotiables include "Context is finite" rule directing use of context-mode tools over raw data reads. Blockers section includes: "Long conversations degrade instruction adherence. When blocked or pivoting after many turns, restate the non-negotiables before continuing."

**`src/prompts/custom.txt`** — The context-mode tools section operationalizes context management mechanically (sandbox execution, routing large output).

**Subagent delegation pattern** — Multi-agent context isolation (subagents explore in own windows, lead agent sees condensed results) mitigates both context rot and multi-turn degradation. Every major coding agent has converged on this pattern.

### Confidence

**High.** Chroma tested 18 frontier models with measured accuracy curves. Anthropic (the maker of Claude) published on it. The Zylos multi-turn degradation finding is model-specific (Claude Code) but directionally consistent with the broader context rot literature. The mechanism (attention entropy increases with sequence length) is well-understood.

### Caveats

- The specific degradation thresholds (50K tokens, 50% fullness) vary by model and architecture.
- The Zylos 10-15% figure is from analysis of Claude Code specifically; other models may degrade at different rates.
- Context-mode tools provide mechanical mitigation, but the prompt-level "context is finite" rule reinforces behavioral discipline.

---

## 11. Additional Research Themes (Not Directly Implemented)

### Chain-of-Thought for Code

| Paper | Venue | Finding |
|-------|-------|---------|
| "RoutingGen" (AAAI 2026) | Peer-reviewed | Dynamic routing reduces tokens by 46%. |
| "LESS, BETTER" (OpenReview) | Preprint | CoT effects are architecture-dependent. Qwen gains +13.4% as base model but loses -15.2% as instruct model. |

**Status:** Partially implemented via code-first guidance in `build-specific.txt`. Full CoT restructuring deferred due to architecture-dependent effects.

### Few-Shot Examples

| Paper | Venue | Finding |
|-------|-------|---------|
| "CodePromptEval" (FSE 2026) | Peer-reviewed | Few-shot + function signature is the most effective combination. 7,072 prompts, 5 techniques, 3 LLMs. |
| "Many-Shot Paradox" (2510.16809) | ICSE 2026 RECODE workshop | Correctness peaks at 5-25 examples, degrades beyond. 90K+ translations, 30 language pairs. |

**Status:** Deferred. Adding examples increases prompt length, conflicting with context length research. The model already has access to codebase files as implicit examples.

### Context Length

| Paper | Venue | Finding |
|-------|-------|---------|
| "Context Rot" (Chroma, 2025) | Industry report | Performance degrades with context length. |
| "Context Length Alone Hurts" (Findings of EMNLP 2025) | Peer-reviewed | Longer contexts reduce performance even when relevant. |
| "Effective Context Engineering" (Anthropic, 2025-09) | Official engineering blog | Hybrid model: CLAUDE.md up front, glob/grep just-in-time. Smallest set of high-signal tokens maximizes desired outcome. See section 9. |

**Status:** Implicitly managed. DCP (context-mode tools) handles context window management automatically, routing large output to the sandbox. Anthropic's hybrid model validates the existing pointers-over-snippets approach in `custom.txt`.

### Persona Prompting

| Paper | Venue | Finding |
|-------|-------|---------|
| "Playing Pretend" (Wharton, 2025) | Preprint | Expert personas do not reliably improve accuracy on hard factual questions. |
| "Persona is a Double-Edged Sword" (IJCNLP 2025 Findings) | Peer-reviewed | Persona prompting can hurt; ensemble methods mitigate. |
| "When A Helpful Assistant Is Not Really Helpful" (Findings of EMNLP 2024) | Peer-reviewed | 162 personas show "no or small negative effects" on performance. |

**Status:** The persona in `custom.txt` line 1 ("You are an expert software engineer") is minimal and non-elaborate. Research suggests elaborate personas add overhead without benefit.

### Prompt Structure

| Paper | Venue | Finding |
|-------|-------|---------|
| "PICCO Framework" (2604.14197) | arXiv preprint | Conceptual framework for prompt structure. No empirical validation. |

**Status:** Deferred. The paper explicitly states it does not claim empirical validation. High disruption for uncertain benefit.

### Prompt Positioning and Priming

| Paper | Venue | Finding |
|-------|-------|---------|
| "Prompt Positioning" (2507.22887) | arXiv preprint | Demos at beginning reliably outperform later placements. Primacy bias becomes less severe with larger models. |

**Status:** Partially implemented via critical-rules-at-top placement in `custom.txt`.

---

## 11. Missing and Contradictory Research

These papers present findings that complicate the implemented recommendations:

| Paper | Finding | Implication |
|-------|---------|-------------|
| "When 'Better' Prompts Hurt" (2601.22025) | Generic prompt additions degrade performance on specific tasks | All recommendations are hypotheses, not guarantees |
| "LESS, BETTER" (OpenReview) | CoT effects are architecture-dependent | Code-first guidance may help some models and hurt others |
| "Playing Pretend" (Wharton, 2025) | Expert personas don't improve accuracy | The simple persona is fine; elaborate personas would be worse |
| "Persona is a Double-Edged Sword" (IJCNLP 2025) | Persona prompting can hurt | Keep persona minimal |

---

## 12. Venue Accuracy

Of the 37 sources cited across this research:

| Category | Count | Examples |
|----------|-------|---------|
| Peer-reviewed at top-tier venues (AAAI, ICML, ACL, EMNLP, FSE, ICLR) | 10 | Control Illusion, Revisiting CoT, Order Matters, CodePromptEval, RECAST (ICLR 2026), Shepherd |
| Peer-reviewed at Findings of top-tier venues | 4 | Order Matters, Step-by-Step Mastery, Context Length Alone Hurts, Think Just Enough |
| Official vendor docs / engineering blogs (Anthropic, OpenAI) | 5 | Best practices for Claude Code, Using CLAUDE.md, Effective Context Engineering, Writing tools for agents, Context management |
| Workshop papers | 2 | White Bear (NeurIPS workshop), Many-Shot Paradox (ICSE workshop) |
| Industry research reports / guides | 9 | Context Rot (Chroma), Armalo Halt Authority, Zylos Research, SurePrompts, eesel, claudecodeguides, amux, Lunar, ZBuild, llmbestpractices |
| arXiv preprints (no peer review) | 14 | Semantic Gravity Wells, When Prohibitions, Social Register, Many-Tier IH, Where IH Breaks, MulDimIF, Goal-Autopilot, Vigil, etc. |

The strongest evidence comes from the peer-reviewed papers and Anthropic's official docs. The false-termination findings (Shepherd peer-reviewed; Goal-Autopilot/Vigil preprints; Armalo industry), the context rot evidence (Chroma 18-model study; Anthropic engineering blog), and Anthropic's CLAUDE.md editorial rules are the highest-authority additions since the original research.

---

## Summary

| Research Theme | Files | Key Lines | Confidence |
|----------------|-------|-----------|------------|
| Negation processing | custom.txt, AGENTS.md | 13, 47, 50, 56, 58, 84-86, 9, 10, 12, 89 | Medium |
| Position effects | custom.txt | 1-8, 83-86 | High |
| Constraint overload | AGENTS.md | 9-16 | Medium |
| Instruction hierarchy | custom.txt, AGENTS.md | 3-8, 3 | High |
| Code-first output order | build-specific.txt | 9 | Low |
| False-termination and halt authority | build-specific.txt, custom.txt, AGENTS.md | new | High |
| Acceptance criteria / work-order prompts | build-specific.txt, plan-specific.txt | new | Medium-High |
| Anthropic CLAUDE.md best practices (cut test, length, emphasis) | custom.txt | editorial | High |
| Just-in-time context and sensors over guides | custom.txt, AGENTS.md | reinforcing | High |
| Context rot and multi-turn coherence degradation | AGENTS.md, custom.txt | new | High |

All recommendations are treated as hypotheses to be validated, not as proven improvements. The false-termination and Anthropic-CLAUDE.md findings are the most robust new sources; the code-first recommendation is the weakest.
