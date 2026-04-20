# Chapter 03 — Workflow (end-to-end user flow)

## Simple explanation

From a user’s eyes: **paste a Figma link** → wait while the system fetches and thinks → **preview a website** → **request tweaks** (“make the hero tighter on mobile”) → the system updates → you **approve** and deploy.

**Neighbors**: [Chapter 02 — Architecture](../02-architecture/README.md) · [Chapter 04 — Agent design](../04-agent-design/README.md) · [Chapter 08 — Feedback loop](../08-feedback-loop/README.md) · [Multi-step orchestration](../05-prompts/multi-step-orchestration.md) · **Canonical algorithm:** [README.md](../../README.md)

## Deep technical breakdown

Implement the flow as a **state machine** with **async jobs**:

- **States**: `received`, `fetching_figma`, `building_ir`, `generating_code`, `running_checks`, `awaiting_review`, `repairing`, `completed`, `failed`.  
- **Async**: Figma fetch and image export can take seconds; codegen may take minutes—use a **job id** and webhooks or polling.  
- **Retries**: exponential backoff on `429` from Figma; retry codegen once on **schema validation failure**; cap repair loops (e.g. max 3) to prevent runaway cost.

Persist **idempotency keys** per `(fileKey, frameId, promptVersion)` so retries do not duplicate artifacts.

## Mermaid diagram

Product-facing **states** (what the UI shows):

```mermaid
stateDiagram-v2
  [*] --> received
  received --> fetching_figma
  fetching_figma --> building_ir: ok
  fetching_figma --> failed: hard_error
  building_ir --> generating_code
  generating_code --> running_checks
  running_checks --> awaiting_review: pass
  running_checks --> repairing: fail
  repairing --> generating_code: retries_left
  repairing --> failed: no_retries
  awaiting_review --> completed: approve
  awaiting_review --> repairing: user_change_request
  completed --> [*]
  failed --> [*]
```

### Orchestrator algorithm (synced with README)

The diagram below is the **same branch-level logic** as diagram **§2 Main job algorithm** in [README.md](../../README.md). Use it when implementing the worker; use the **state diagram** above for UX copy and dashboards.

```mermaid
flowchart TB
  startNode([start_job])
  startNode --> vIn{"inputs_valid_fileKey_frame_config"}
  vIn -->|no| badIn([terminal_invalid_input])
  vIn -->|yes| loadPol[load_policy_secrets_and_limits]
  loadPol --> fetchTry[figma_GET_v1_files_key]
  fetchTry --> http429{"http_status_429"}
  http429 -->|yes| cntF{"fetch_attempt_lt_R_figma"}
  cntF -->|yes| sleepB[sleep_exponential_backoff_jitter]
  sleepB --> fetchTry
  cntF -->|no| limF([terminal_figma_rate_limited])
  http429 -->|no| httpOk{"http_status_200"}
  httpOk -->|no| figErr([terminal_figma_http_error])
  httpOk -->|yes| parseD[deterministic_parse_FigmaJSON_to_IR]
  parseD --> irOk{"jsonschema_IR_valid"}
  irOk -->|no| irFail([terminal_IR_build_failed])
  irOk -->|yes| layCall[LLM_layout_analyzer_plus_schema_validate]
  layCall --> layOk{"layout_JSON_valid"}
  layOk -->|no| layRetry{"layout_attempt_lt_R_llm"}
  layRetry -->|yes| layCall
  layRetry -->|no| layFail([terminal_layout_failed])
  layOk -->|yes| mapCall[LLM_component_mapper_plus_schema_validate]
  mapCall --> mapOk{"mapper_JSON_valid"}
  mapOk -->|no| mapRetry{"mapper_attempt_lt_R_llm"}
  mapRetry -->|yes| mapCall
  mapRetry -->|no| mapFail([terminal_mapper_failed])
  mapOk -->|yes| genCall[LLM_code_generator_emit_PatchBundle]
  genCall --> genOk{"patches_schema_and_path_allowlist_ok"}
  genOk -->|no| genRetry{"codegen_attempt_lt_R_llm"}
  genRetry -->|yes| genCall
  genRetry -->|no| genFail([terminal_codegen_failed])
  genOk -->|yes| applyP[git_apply_patches_atomic_to_workspace]
  applyP --> staticV[run_tsc_eslint_unit_fast_host_or_sandbox]
  staticV --> stOk{"static_checks_exit_zero"}
  stOk -->|no| fbA[feedback_engine_build_RepairBrief_from_logs]
  fbA --> repA{"repair_count_lt_R_repair"}
  repA -->|yes| incR[increment_repair_count_append_brief_to_context]
  incR --> genCall
  repA -->|no| escA([terminal_needs_human_escalation])
  stOk -->|yes| sbx[sandbox_pnpm_install_build_test_isolated]
  sbx --> sbxOk{"sandbox_exit_zero"}
  sbxOk -->|no| fbB[feedback_engine_from_sandbox_logs]
  fbB --> repB{"repair_count_lt_R_repair"}
  repB -->|yes| incR
  repB -->|no| escB([terminal_needs_human_escalation])
  sbxOk -->|yes| pack[write_artifact_bundle_dist_and_meta]
  pack --> prv[publish_preview_URL_optional]
  prv --> waitH{await_human_review_or_auto_approve}
  waitH -->|approve| doneOk([terminal_success_publish_allowed])
  waitH -->|change_request_text| humanBrief[merge_human_brief_into_repair_context]
  humanBrief --> incR
```

#### Mapping states to algorithm regions

| UI / DB state | Region in flowchart (approximate) |
|---------------|-----------------------------------|
| `received` | before `fetchTry` |
| `fetching_figma` | `fetchTry` … `httpOk` |
| `building_ir` | `parseD` … `mapOk` (IR + layout + mapper) |
| `generating_code` | `genCall` … `applyP` |
| `running_checks` | `staticV` … `sbxOk` |
| `repairing` | `fbA`/`fbB` → `incR` → `genCall` |
| `awaiting_review` | `waitH` |
| `completed` | `doneOk` |
| `failed` | any `terminal_*` node |

## Real example

1. User submits job `J-1001`.  
2. Worker hits Figma `GET /v1/files/abc` (retry x3 on 429).  
3. IR built; codegen produces `Hero.tsx`.  
4. Sandbox fails TypeScript: missing import. Validator returns structured error `{ "rule": "tsc", "message": "..." }`.  
5. Feedback engine re-prompts codegen with error context; second build passes; UI shows **Awaiting review**.

## Challenges and pitfalls

- **Blocking UI** on long steps: always show **progress** and partial logs.  
- **Unbounded repair**: without caps, the agent can loop forever on a fundamental IR bug.

## Tips and best practices

- Store **structured validator output** in the DB; it is gold for evals and prompt tuning.  
- Let users **pin** a `promptVersion` when comparing quality across releases.

## What most people miss

**Awaiting review** is a first-class state, not an afterthought. The best systems treat human review as part of the **control loop**, not a manual export step.
