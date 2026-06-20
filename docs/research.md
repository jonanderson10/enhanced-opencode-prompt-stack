# Research: System Prompt Best Practices for Code Harnesses

This document maps academic and industry research findings to the prompt files in this project. It explains *why* each file says what it says, grounded in evidence rather than intuition.

## Project Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/prompts/custom.txt` | Shared base prompt for all providers | 86 |
| `src/prompts/build-specific.txt` | Build-mode additions | 15 |
| `src/prompts/plan-specific.txt` | Plan-mode additions | 73 |
| `src/AGENTS.md` | Distributed workflow instructions | 123 |

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

**Medium.** Position bias is well-established across multiple papers. The specific prescription "critical rules at top and bottom" is an inference from general position literature. The "Order Matters" paper studied constraint ordering difficulty (hard-to-easy), not importance-based positioning — related but distinct claims.

### Caveats

- Position bias is an inherent geometric property of causal decoders (Lost in the Middle at Birth). It cannot be eliminated through prompt engineering, only worked around.
- The U-shaped curve means middle content is weakest, but the exact drop-off varies by model and context length.

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
> "When writing code, generate the implementation first and explain your reasoning after — not the reverse. This applies to output order, not to exploration: understand the problem and codebase before writing."

### Confidence

**Medium.** The research finding (code first → better) is solid and peer-reviewed. The application as a prompt instruction is an extrapolation from a training-time finding (SFT data ordering) to inference-time behavior. CoT effects are architecture-dependent (LESS, BETTER): some models gain from CoT, others lose.

### Caveats

- "Revisiting CoT" is about supervised fine-tuning data ordering, not inference-time prompting. The 9.86% improvement comes from training models to generate code first.
- The "LESS, BETTER" paper shows CoT effects are architecture-dependent: Qwen gains +13.4% from CoT as a base model but loses -15.2% as an instruct model.
- The exploration caveat ("understand the problem and codebase before writing") prevents misinterpretation as "start coding immediately."

---

## 6. Additional Research Themes (Not Directly Implemented)

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

**Status:** Implicitly managed. DCP (context-mode tools) handles context window management automatically, routing large output to the sandbox.

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

## 7. Missing and Contradictory Research

These papers present findings that complicate the implemented recommendations:

| Paper | Finding | Implication |
|-------|---------|-------------|
| "When 'Better' Prompts Hurt" (2601.22025) | Generic prompt additions degrade performance on specific tasks | All recommendations are hypotheses, not guarantees |
| "LESS, BETTER" (OpenReview) | CoT effects are architecture-dependent | Code-first guidance may help some models and hurt others |
| "Playing Pretend" (Wharton, 2025) | Expert personas don't improve accuracy | The simple persona is fine; elaborate personas would be worse |
| "Persona is a Double-Edged Sword" (IJCNLP 2025) | Persona prompting can hurt | Keep persona minimal |

---

## 8. Venue Accuracy

Of the 26 sources cited across this research:

| Category | Count | Examples |
|----------|-------|---------|
| Peer-reviewed at top-tier venues (AAAI, ICML, ACL, EMNLP, FSE) | 8 | Control Illusion, Revisiting CoT, Order Matters, CodePromptEval |
| Peer-reviewed at Findings of top-tier venues | 3 | Order Matters, Step-by-Step Mastery, Context Length Alone Hurts |
| Workshop papers | 2 | White Bear (NeurIPS workshop), Many-Shot Paradox (ICSE workshop) |
| Industry research reports | 1 | Context Rot (Chroma) |
| arXiv preprints (no peer review) | 12 | Semantic Gravity Wells, When Prohibitions, Social Register, Many-Tier IH, Where IH Breaks, MulDimIF, RECAST, etc. |

The strongest evidence comes from the peer-reviewed papers. The arXiv preprints provide supporting evidence but have not undergone formal review.

---

## Summary

| Research Theme | Files | Key Lines | Confidence |
|----------------|-------|-----------|------------|
| Negation processing | custom.txt, AGENTS.md | 13, 47, 50, 56, 58, 84-86, 9, 10, 12, 89 | Medium |
| Position effects | custom.txt | 1-8, 83-86 | Medium |
| Constraint overload | AGENTS.md | 9-16 | Medium |
| Instruction hierarchy | custom.txt, AGENTS.md | 3-8, 3 | High |
| Code-first output order | build-specific.txt | 9 | Medium |

All recommendations are treated as hypotheses to be validated, not as proven improvements. The negation processing and instruction hierarchy findings are the most robust; the code-first recommendation is the weakest.
