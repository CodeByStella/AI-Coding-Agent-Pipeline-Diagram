# AI builder pipeline — AI coding agent design corpus

This repository holds a **structured, diagram-first description** of how an **AI coding agent** works: from high-level intent down to components, lifecycle, governance, and operations. The docs are **implementation-agnostic** at the core, with a **Cursor-focused appendix** that maps the same ideas to rules, skills, hooks, and MCP.

## How to read

Pick a path by time and role:

| Path | Time | Audience | Start here |
|------|------|----------|------------|
| **Executive** | ~30 min | Product, leads | [docs/10-overview.md](docs/10-overview.md) → [docs/20-architecture.md](docs/20-architecture.md) |
| **Builder** | ~2 hr | Engineers shipping agents | [docs/20-architecture.md](docs/20-architecture.md) → [docs/30-lifecycle.md](docs/30-lifecycle.md) → [docs/40-components.md](docs/40-components.md) |
| **Safety & ops** | ~2 hr | Security, platform, SRE | [docs/50-governance.md](docs/50-governance.md) → [docs/60-operations.md](docs/60-operations.md) |

Full map, conventions, and reading order: **[docs/00-index.md](docs/00-index.md)**.

## Core links

- [docs/00-index.md](docs/00-index.md) — documentation map and cross-link conventions  
- [RULES.md](RULES.md) — normative rules for authors and agents working in this repo  
- [docs/91-glossary.md](docs/91-glossary.md) — terms and backlinks to first definitions  
- [docs/90-references.md](docs/90-references.md) — external proof links (vendors, standards)  
- [docs/appendix-cursor.md](docs/appendix-cursor.md) — mapping to Cursor (rules, skills, hooks, MCP)

## What this is not

This repo does not ship a runnable agent runtime by default. It is a **design and teaching corpus** you can align product code, evaluations, and editor configuration against.
