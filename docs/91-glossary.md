# Glossary

Terms used across the corpus. Each entry links to the **first defining section** (or the closest equivalent).

| Term | Definition | First used in |
|------|------------|---------------|
| **AI coding agent** | A system that turns natural-language goals into repository changes via an LLM, tools, and policies, usually in a loop. | [10-overview.md#summary](10-overview.md#summary) |
| **Orchestrator** | The component that owns session state, calls the model, dispatches tools, and applies governance gates. | [20-architecture.md#container-view](20-architecture.md#container-view) |
| **LLM** | Large language model; the probabilistic core that proposes plans and tool calls. | [10-overview.md#summary](10-overview.md#summary) |
| **Tool calling** | Structured outputs from the model that name a tool and arguments; the host executes and returns observations. | [30-lifecycle.md#summary](30-lifecycle.md#summary) · [REF-OPENAI-FC](90-references.md), [REF-ANTHROPIC-TOOLS](90-references.md) |
| **Tool host** | Runtime that registers tools, validates arguments, executes with OS identity and sandbox, streams results. | [40-components.md#tool-host](40-components.md#tool-host) |
| **RAG** | Retrieval-augmented generation: fetch repo or docs chunks before generation to ground answers. | [40-components.md#retrieval-and-context](40-components.md#retrieval-and-context) |
| **MCP** | Model Context Protocol: standard way to expose tools, prompts, and resources to hosts like editors. | [appendix-cursor.md#mcp](appendix-cursor.md#mcp) · [REF-MCP-SPEC](90-references.md) |
| **Sandbox** | Isolation boundary for shell, network, and filesystem (containers, VMs, policy wrappers). | [20-architecture.md#trust-boundaries](20-architecture.md#trust-boundaries) |
| **HITL** | Human-in-the-loop: explicit approval before irreversible or high-risk actions. | [50-governance.md#summary](50-governance.md#summary) |
| **Prompt injection** | Attacker-controlled content in context that steers the model to misuse tools or leak secrets. | [50-governance.md#threats](50-governance.md#threats) · [REF-OWASP-LLM](90-references.md) |
| **Least privilege** | Grant tools and credentials only what the task needs, for only as long as needed. | [50-governance.md#policy-flow](50-governance.md#policy-flow) |
| **Telemetry** | Traces, logs, metrics, and cost accounting for model and tool usage. | [60-operations.md#observability](60-operations.md#observability) |
| **Eval** | Systematic test of agent behavior (fixed tasks, judges, regression suites). | [60-operations.md#evaluations](60-operations.md#evaluations) |

## See also

- [00-index.md](00-index.md) — map and reading paths  
- [10-overview.md](10-overview.md) — narrative start  
- [90-references.md](90-references.md) — proof links keyed to topics  
