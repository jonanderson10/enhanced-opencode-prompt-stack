# MiniMax - Prompting Best Practices

**Source:** https://platform.minimax.io/docs/token-plan/prompting-best-practices#m-series-usage-tips

Extracted principles for use in global agent instructions. See source for detailed examples.

---

## General Principles

### Be clear and direct
State the task, constraints, and desired output explicitly. Show your work to a colleague who has no context — if they would be confused, the model will be too.

### Add context explaining why
When explaining a constraint, include the intent behind it. Context is especially valuable for formatting, safety, accessibility, and workflow constraints.

### Use examples for hard pattern matching
For classification, structured extraction, or edge-case handling, include 3–5 diverse examples. Repeating similar examples wastes tokens without teaching the model anything new.

### Use prompt templates
For repeated tasks, use named variables to make the task, inputs, and guardrails visible. This helps when debugging prompt regressions.

### Match output language explicitly
When input mixes languages or a specific output language is needed, say so explicitly. Without this, the model follows the dominant input language.

---

## Output and Formatting

### Structure prompts with clear sections
Use short labels — Task, Context, Source, Constraints, Output format. A bold header or trailing colon marks each section. Keep structure flat; deep nesting hurts readability.

### Set role, format, and length
Role instructions define expertise, scope, and decision criteria. Output instructions specify sections, fields, and length limits that are easy to verify. Prefer concrete output contracts over vague asks like "be detailed."

---

## Long Context

### Place the task after the source
For long inputs, write the question or task after the source documents. The model is more likely to keep the task in focus when it is closest to its own response.

### Index and delimit source material
For very large inputs, delimit with headers including source name and date. Ask the model to quote or summarize relevant parts before answering.

---

## Tool Use

### Define tool contracts
Define each tool with name, purpose, inputs, return shape, and failure behavior. The model should understand the tool contract before deciding to call it.

### Parallelize independent tool calls
Use parallel calls for independent read-only lookups. Use sequential calls when one result determines the next query or action.

### Avoid overeagerness
Use tools only when they materially improve the answer. Set clear stopping rules.

---

## Thinking and Reasoning

### Control reasoning depth
Ask for deeper analysis when the task involves planning, debugging, tradeoffs, or long-horizon execution. Ask for a direct answer when the task is extraction, rewriting, or formatting.

### Reduce hallucinations
Three patterns:
- **Permission to refuse** — explicitly tell the model what to say when it does not know
- **Reference grounding** — require the model to quote or cite the source it used
- **Boundaries before generation** — state allowed sources, time range, or product version before the task

---

## Agentic and Long-Task Workflows

### State tracking
For long-running tasks, keep the working plan, current status, and open questions visible in the prompt or project notes.

### Evaluate and iterate
Treat important prompts like product configuration. Keep a small evaluation set, compare prompt versions, and record what changed.