# Chapter 05 — Prompt engineering (overview)

## Simple explanation

**Prompt engineering** means writing instructions so the model does **predictable** work. In this system, each agent step gets: what role it plays, what JSON it must return, and what happens if it disobeys. Good prompts are **short, structured, and testable**.

**Neighbors**: [Chapter 04 — Agent design](../04-agent-design/README.md) · per-node pages: [figma-parser](figma-parser.md), [layout-analyzer](layout-analyzer.md), [component-mapper](component-mapper.md), [code-generator](code-generator.md), [validator](validator.md), [feedback-engine](feedback-engine.md) · [Chapter 06 — Code generation](../06-code-generation/README.md)

## Deep technical breakdown

Use a **three-layer prompt**:

1. **System**: immutable policy (output schema, banned APIs, stack assumptions).  
2. **Developer** (optional): tool and IR schema excerpts.  
3. **User**: task-specific slice (frame id, constraints, prior errors).

**Prompt chaining**: output of step N becomes **typed input** to step N+1—never free-form prose between machines. **Debugging**: log token counts, refusal patterns, schema validation failures; keep a **golden IR** fixture and run prompts in CI with `temperature=0` for regression.

**Validation rules**: every LLM step should post-process with **JSON Schema** validation and reject-and-retry once with the validation error appended.

## Mermaid diagram

```mermaid
flowchart LR
  sys[System_prompt]
  dev[Developer_context_IR_excerpt]
  usr[User_task_slice]
  llm[LLM]
  val[JSON_Schema_validate]
  fix[Single_retry_with_errors]

  sys --> llm
  dev --> llm
  usr --> llm
  llm --> val
  val -->|"fail"| fix
  fix --> llm
  val -->|"ok"| out[Structured_output]
```

## Real example

**Chaining**: `figma-parser` emits `NormalizedNode[]` → `layout-analyzer` consumes only the subtree for `frame=Hero` → `code-generator` receives `LayoutBox` objects, not raw Figma.

## Challenges and pitfalls

- **Bad prompt**: “Write React for this JSON” with 40k tokens of unfiltered Figma → inconsistent components.  
- **Good prompt**: “Emit **only** `components[]` matching schema v3; use design tokens IDs; no external libraries.”

## Tips and best practices

- Maintain a **prompt changelog** next to your IR schema version.  
- Keep **negative constraints** (“do not use inline styles except motion”) explicit.

## What most people miss

The best guardrail is not clever wording; it is a **strict schema + automatic repair loop**. Prompts set intent; schemas enforce reality.
