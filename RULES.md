# Rules — authors and agents

These rules apply to **humans** editing this documentation and to **AI agents** asked to maintain or extend it.

## Scope and edits

- Keep changes **focused** on the documentation request. Do not refactor unrelated files or add speculative code unless explicitly requested.
- Prefer **small, reviewable diffs** over sweeping rewrites unless a doc is structurally wrong.

## Structure and navigation

- **New subsystem topics** must declare their place in the tree: link **up** to the parent doc, **down** to child detail (if any), and add a row to [docs/00-index.md](docs/00-index.md) if the page is new.
- Use **relative links only** between repository files (portable on GitHub and locally).
- Prefer **descriptive link text** over raw URLs in prose. Bare URLs belong primarily in [docs/90-references.md](docs/90-references.md).

## Claims and proof

- **Strong or vendor-specific claims** (tool formats, security properties, product behavior) require a supporting entry in [docs/90-references.md](docs/90-references.md) with a stable, primary URL where possible.
- If no primary source exists, label the claim as **heuristic** or **opinion** in the narrative.

## Diagrams (Mermaid)

- Use **flowchart** for policy and control flow, **sequenceDiagram** for tool and IO loops, **stateDiagram-v2** for session lifecycle.
- **Node IDs** must be alphanumeric or underscores—no spaces in IDs.
- **Edge labels** that contain parentheses, commas, or colons must be quoted in Mermaid.
- Do **not** use Mermaid `style` / `classDef` coloring in this corpus; rely on the default theme for dark-mode safety.
- When a section is architecture-heavy, put a **diagram before** a long prose wall.

## Glossary

- On **first substantive use** of a glossary term in a doc, link to [docs/91-glossary.md](docs/91-glossary.md) once; afterward use the plain term.
- Adding a term requires a **glossary entry** with at least one backlink to the defining section.

## Agent behavior in this repo (when using automation)

- Match the tone and structure of existing docs: summary first, diagram second, detail third.
- After substantive edits, **check internal links** mentally or with a quick search for broken `](docs/` targets.
