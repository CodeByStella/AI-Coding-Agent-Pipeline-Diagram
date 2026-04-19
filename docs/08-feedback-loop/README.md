# Chapter 08 — Feedback loop (iteration and self-repair)

## Simple explanation

The first generated site is rarely perfect. A **feedback loop** means: detect problems, tell the agent **exactly** what failed, and regenerate **only what must change**—like revising an essay with teacher comments.

**Neighbors**: [Chapter 03 — Workflow](../03-workflow/README.md) · [Chapter 05 — Prompts / Feedback engine](../05-prompts/feedback-engine.md) · [Chapter 12 — Common issues](../12-common-issues/README.md)

## Deep technical breakdown

**Automated loop**: `validator.errors[]` → `RepairBrief` → `PatchBundle` (delta) → re-run checks until pass or max retries.  
**Human loop**: UI shows diff + preview URL; user comments create a new `RepairBrief` with `source: human`.  
**Diff-based updates**: prefer **unified diffs** or structured patches to avoid rewriting entire files (smaller prompts, fewer regressions). Track `baseCommit` so retries apply on consistent parent.

## Mermaid diagram

```mermaid
flowchart LR
  gen[Generate_v1]
  val[Validate]
  fb[Feedback_brief]
  gen2[Generate_v2_delta]

  gen --> val
  val -->|"errors"| fb
  fb --> gen2
  gen2 --> val
  val -->|"pass"| done[Ship]
```

## Real example

TypeScript error `TS2307` missing CSS module → feedback brief goal: “add `Hero.module.css` with class names referenced in TSX.” Codegen issues **add** patch only for CSS file.

## Challenges and pitfalls

- **Wobble**: fix A unfixes B—use `nonGoals` and snapshot tests on critical sections.  
- **Unbounded scope**: user asks for new feature during bugfix—route to new job.

## Tips and best practices

- Keep a **patch budget** (max files touched per repair).  
- Show users a **side-by-side** diff with Figma frame thumbnail.

## What most people miss

Successful loops depend on **error causality**: validators must attribute errors to **specific files and nodes**; otherwise the LLM guesses.
