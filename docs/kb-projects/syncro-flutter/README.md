# AI Knowledge Base — Shared

Cross-stack documents used by all projects regardless of technology.

## 3-Layer KB Architecture

Knowledge is split across three layers, from broadest to narrowest scope:

| Layer | Where | Scope | Skill to write | Tiered loading? |
|-------|-------|-------|----------------|-----------------|
| **Layer 1 — Framework** | `.ai-framework/knowledge-base/{stack}/` | All projects of a stack, everywhere | `add-kb-doc` | Yes (`_index.md` + `.compact.md`) |
| **Layer 2 — Container** | `docs/kb-container/` at repo root | All projects in this repo/product | `document-solution` | No (project-specific, smaller docs) |
| **Layer 3 — Project** | Isolated: `docs/kb-projects/{project}/`. Embedded: `{project}/docs/kb-project/` | One specific codebase | `document-solution` | No (project-specific, smaller docs) |

In single layout, Layer 2 and Layer 3 collapse to the same path.

### What goes where

- **"Is this true for ALL Swift/Rails/React projects?"** → Layer 1 (Framework)
  - RxSwift DisposeBag rules, Realm thread safety, Rails migration conventions
- **"Is this shared across projects in this repo?"** → Layer 2 (Container) → route to:
  - `product/domain/` — business rules, user model shape, domain concepts
  - `technical/api/` — shared REST API contracts
  - `technical/ble/` — BLE protocol specs
  - `technical/data-models/` — canonical data shapes
  - `engineering/best-practices/` — cross-stack engineering standards
- **"Is this specific to this one codebase?"** → Layer 3 (Project)
  - This app's coordinator pattern, bug in migration v12, session handoffs

### How agents load KB

```
1. Read .ai-framework/knowledge-base/{STACK}/_index.md    (Layer 1 — index only)
2. Match task keywords → load compact (.compact.md) first  (Layer 1 — compact)
3. Escalate to full doc only if compact is insufficient     (Layer 1 — full)
4. Read {CONTAINER_KB_DIR}/README.md                        (Layer 2 — index)
5. Read {KB_DIR}/README.md                                  (Layer 3 — index)
6. Load matching docs from Layers 2-3 as needed
```

Tiered loading (compact vs full) only applies to Layer 1, where docs tend to be larger
and technology-generic. Layers 2-3 are project-specific and typically smaller.

---

## Shared Index

### AI Patterns

| Document | When to Read | Last Updated |
|----------|--------------|--------------|
| [mistake-log](./ai-patterns/mistake-log.md) | Session start — check for patterns to avoid | |

---

## Stack-Specific KBs (Layer 1)

Each stack has its own `_index.md` entry point:

| Stack | Index | Docs |
|-------|-------|------|
| Swift | `knowledge-base/swift/_index.md` | 19 docs (16 with compact variants) |
| Android | `knowledge-base/android/_index.md` | 1 doc |
| Flutter | `knowledge-base/flutter/_index.md` | 1 doc |
| Rails | `knowledge-base/rails/_index.md` | 1 doc |
| React | `knowledge-base/react/_index.md` | 1 doc |
| Python | `knowledge-base/python/_index.md` | 1 doc |
| Docs | `knowledge-base/docs/_index.md` | 1 doc |
| Generic | `knowledge-base/generic/_index.md` | 1 doc (fallback) |

---

## Maintenance

The knowledge base is self-maintaining via skills:

| Trigger | Skill | Layer affected |
|---------|-------|---------------|
| KB gap found during work | `add-kb-doc` | Layer 1 (Framework) |
| Complex solution to document | `document-solution` | Layer 2 or 3 |
| Pattern reusable across projects | `contribute-pattern` → `add-kb-doc` | Promotes Layer 3 → Layer 1 |
| Correction by user | `log-mistake` | Layer 3 (Project) |
| Long session to hand off | `save-session` | Layer 3 (Project) |
| KB files changed | `check-kb-index` | Whichever layer was modified |

### Adding New Docs to Layer 1

1. Run `add-kb-doc` — it handles the full lifecycle:
   - Creates the doc in the appropriate stack and category
   - Generates a `.compact.md` variant if over 100 lines
   - Updates the stack's `_index.md` with triggers and line count
   - Updates the stack's `README.md` catalog

---

**Last Updated**: April 2026
