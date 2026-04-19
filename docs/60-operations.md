# Operations — observability, cost, evals, releases

## Summary

Running an [AI coding agent](91-glossary.md) in production (or at team scale) requires **observability**, **cost controls**, **evaluations** ([evals](91-glossary.md)), and a **release process** for prompts and policies. This complements [governance](50-governance.md): policy defines what must not happen; operations proves what actually happens.

## Observability

Minimum signal per session:

| Signal | Use |
|--------|-----|
| Model latency and tokens | Cost, SLAs, regression when switching models |
| Tool success and duration | Pinpoint flaky adapters |
| Denials and HITL blocks | Tune policies; training for users |
| Final verify status | Quality gate; connects to evals |

**Privacy**: treat transcripts as sensitive; align retention to policy ([REF-NIST-AI](90-references.md) for organizational framing).

## Cost and budgets

- Per-session **token ceilings** and **tool step ceilings** (see [30-lifecycle.md](30-lifecycle.md)).  
- **Model routing**: cheaper models for retrieval summarization, stronger model for final patch.  
- **Caching**: idempotent reads (file hashes) to avoid redundant tool chatter.

## Evaluations

**Levels**:

1. **Unit**: schema validation, path rules, prompt templates render.  
2. **Scenario**: fixed repos with golden tasks; score pass/fail on tests and diff quality.  
3. **Online shadow**: duplicate suggestions without auto-apply; human labels.  

Connect failures back to components in [40-components.md](40-components.md).

```mermaid
flowchart LR
  fix[Fixed_task_set]
  run[Agent_run]
  judge[Auto_judge]
  human[Spot_human_review]

  fix --> run --> judge --> human
```

## Releases of behavior

Version together:

- System prompts and rule packs  
- Tool manifests (names, descriptions, JSON Schema)  
- Policy tables (allowlists, regex blocks)  

Use semver or date stamps; keep **changelogs** so regressions map to a release.

## See also

- Up: [50-governance.md](50-governance.md)  
- Sideways: [30-lifecycle.md](30-lifecycle.md), [40-components.md](40-components.md)  
- Proof: [90-references.md](90-references.md)  
- Map: [00-index.md](00-index.md)  
