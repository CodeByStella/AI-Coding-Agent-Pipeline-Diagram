# Chapter 09 — Model selection

## Simple explanation

Different **AI models** have different strengths: some are better at long code, some at strict JSON, some are cheaper. **Model selection** means picking the right worker for each step and budget.

**Neighbors**: [Chapter 11 — Scaling](../11-scaling/README.md) · [Chapter 15 — Cost optimization](../15-cost-optimization/README.md)

## Deep technical breakdown

**Closed models (example families: GPT)**: strong general reasoning, good tool adherence with schema prompting; cost scales with tokens; good for **codegen** and **feedback** when quality matters.  
**Open-weight models (examples: Llama, Mistral, Qwen families)**: lower per-token cost self-hosted; need tighter prompts and more eval; good for **high-volume triage** or **sandboxed internal** use.

**Heuristic routing**:

| Step | Prefer | Why |
|------|--------|-----|
| IR normalization (LLM assist) | smaller / cheaper | structured, short |
| Layout ambiguity resolution | mid-size | needs reasoning |
| Codegen | strongest affordable | long context + correctness |
| Validator triage | small | classification |

Track **quality metrics** per route (pass rate, retries) weekly.

## Mermaid diagram

```mermaid
flowchart TD
  job[Job_metadata]
  r[Router_policy]
  a[Model_A]
  b[Model_B]

  job --> r
  r -->|"complex_or_high_risk"| a
  r -->|"bulk_or_low_risk"| b
```

## Real example

Policy YAML: `codegen.model = gpt-4.1` for customer jobs; `triage.model = local-llama-3-8b` only when `errors.length>30`.

## Challenges and pitfalls

- **Model drift**: vendor updates behavior—pin **snapshot evals** and version prompts.  
- **Hidden costs**: long system prompts repeated every call—cache static prefixes where API allows.

## Tips and best practices

- A/B test **one step at a time**; do not change parser+codegen simultaneously.  
- Log **tokens in/out** per step for finance attribution.

## What most people miss

The cheapest win is often **smaller context** (better IR filtering), not a cheaper model.
