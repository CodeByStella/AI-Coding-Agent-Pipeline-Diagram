# Chapter 02 — Architecture

**Build track:** implement the **orchestrator boundary** and container split during **M0–M3** ([build track](../00-build-track/README.md)); the diagrams below match the README “canonical” copies.

## Simple explanation

**Architecture** answers: what are the big boxes, and how do they talk? Here the boxes are: **ingest truth** (Figma API **or** brief/spec stores), **understand layout**, **map to React components**, **write files**, **check quality**, and **loop on feedback**. Each box can be a **service** or a **step** inside one app.

**Neighbors**: [Build track](../00-build-track/README.md) · [Chapter 01 — Overview](../01-overview/README.md) · [Chapter 03 — Workflow](../03-workflow/README.md) · [Chapter 04 — Agent design](../04-agent-design/README.md) · [Chapter 18 — Requirements-only intake](../18-greenfield-from-requirements/README.md) · [Chapter 17 — Build vs integrate](../17-build-vs-integrate/README.md) · **Canonical diagrams:** [README.md](../../README.md) (*Visual architecture — topology plus algorithms*)

## Deep technical breakdown

Use a **modular pipeline** behind a single orchestrator API:

| Container | Responsibility |
|-----------|----------------|
| **Figma client** | OAuth/token, `GET /v1/files/:key`, `GET /v1/images`, rate-limit handling (**when Figma intake is enabled**) |
| **Brief and spec store** | Raw markdown, `ProductBrief`, `UxSpec`, `DesignSpec` revisions + approval metadata (**requirements-only** jobs); treat uploads as **untrusted** |
| **IR builder** | Deterministic transform: Figma JSON → typed IR (frames, text styles, layout boxes) |
| **Spec-to-layout bridge** | Deterministic slicing: `DesignSpec` → same **layout input shape** your `layout_analyzer` expects (optional shared schema with IR) |
| **Agent workers** | LLM calls per task with strict JSON schema outputs |
| **Workspace writer** | Applies unified diffs to a git worktree |
| **Sandbox runner** | `pnpm install`, `pnpm test`, `pnpm build` in isolated environment |
| **Artifact store** | S3/Git bundle of outputs and logs |

Communication is **message-passing**: each step receives a **layout input slice** (from **IR** or from **`DesignSpec`**), plus `userConfig` and `priorErrors`, and returns structured outputs (e.g. patches) with telemetry. Avoid letting the LLM freely write to disk without schema validation.

### Requirements-only topology (extra boxes)

When `source` is **requirements-only**, the **Figma client** is unused; **brief/spec** containers feed the pipeline until `DesignSpec` is approved, then workers run the **shared agentic core** from **layout analysis** onward.

```mermaid
flowchart TB
  subgraph intake [Upstream_requirements_first]
    raw[Raw_brief_store]
    pb[ProductBrief_validator]
    ux[UxSpec_validator]
    ds[DesignSpec_validator]
    raw --> pb --> ux --> ds
  end
  subgraph workers [Shared_agent_workers_from_layout_onward]
    l[Layout_analyzer]
    m[Component_mapper]
    g[Code_generator]
  end
  ds --> l --> m --> g
```

**Prompt modules and planner loops** (how prompts are composed and when the LLM chooses several steps) live in [Modular prompt architecture](../05-prompts/modular-prompt-architecture.md) and [Multi-step orchestration](../05-prompts/multi-step-orchestration.md). **When to integrate sandboxes, gateways, and queues** instead of building them is covered in [Chapter 17 — Build vs integrate](../17-build-vs-integrate/README.md).

## Mermaid diagram

Container placement (how services sit relative to control vs data plane):

```mermaid
flowchart TB
  subgraph host [Control_plane]
    orch[Orchestrator_API]
    q[Job_queue]
  end
  subgraph data [Data_plane]
    figma[Figma_client]
    ir[IR_store]
    ws[Git_workspace]
  end
  subgraph workers [Agent_workers]
    p[Parser_normalizer]
    l[Layout_analyzer]
    m[Component_mapper]
    g[Code_generator]
    v[Validator]
    f[Feedback_engine]
  end
  sb[Sandbox_CI]

  orch --> q
  q --> p
  p --> ir
  p --> l
  l --> m
  m --> g
  g --> ws
  g --> v
  v --> sb
  v -->|"errors"| f
  f --> q
  figma --> p
```

### Canonical topology and algorithm (synced with README)

The following blocks mirror [README.md](../../README.md) (*Visual architecture — topology plus algorithms*). **Edit README and this subsection together** when the orchestrator algorithm changes.

#### Topology (compact)

```mermaid
flowchart LR
  usr[User] --> orch[Orchestrator]
  rev[Review_UI] --> orch
  orch --> fapi[Figma_API]
  orch --> pipe[Agent_pipeline]
  pipe --> ws[Workspace]
  ws --> qual[Validator_Sandbox]
  qual --> orch
  orch --> art[Artifacts_Preview_Deploy]
```

#### Main job algorithm (branch-level detail)

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

**Policy knobs (same names as README):** `R_figma` (Figma fetch retries), `R_llm` (per-stage schema retries), `R_repair` (codegen repair budget), transactional **`applyP`**, human gate **`waitH`**.

For the **time-ordered** view see [Chapter 04 — Agent design](../04-agent-design/README.md).

## Real example

A request `POST /jobs` with body `{ "fileKey": "abc", "frameId": "123:456", "stack": "vite-react-ts" }` enqueues work. Worker `p` fetches Figma JSON and writes `ir/landing.json`. Worker `g` emits `src/App.tsx` importing `./sections/Hero`.

## Challenges and pitfalls

- **Monolith creep**: one 5,000-line prompt tries to do parse+codegen; debugging becomes impossible.  
- **Shared mutable workspace**: parallel jobs overwrite each other—use **per-job worktrees**.

## Tips and best practices

- Define a **versioned IR schema** (`ir.schema.v2.json`) and validate every worker output against it.  
- Put **secrets** only in the control plane; sandboxes get short-lived tokens.

## What most people miss

The IR is your **real contract** between deterministic code and probabilistic LLM steps. If the IR is fuzzy, every downstream prompt inherits that ambiguity.
