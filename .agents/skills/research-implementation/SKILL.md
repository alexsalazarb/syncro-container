---
name: research-implementation
description: >-
  Researches how to implement an unknown feature pattern or design approach.
  Does web search + codebase analysis, then captures findings to the KB via
  add-kb-doc. Use before create-plan or execute-task when the implementation
  approach is unclear. Triggered automatically when create-plan or execute-task
  detects a KB gap.
---

# Research Implementation

## Context Required
LOW-CONTEXT: KB _index.md for the active stack

## Triggers

### Automatic
- Called from `create-plan` Step 2d when KB gaps detected after codebase exploration
- Called from `execute-task` Step 2 pre-flight when task touches unknown implementation territory

### Manual
- `/research-implementation [topic]`
- "research how to implement X"
- "how should we implement X"
- "I don't know how to approach X"

## Instructions

### Step 1: Resolve context

Read `.ai-framework.config` for `STACK`. Resolve `FRAMEWORK_KB_DIR`, `CONTAINER_KB_DIR`, and `KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

Resolved variables:
- **`FRAMEWORK_KB_DIR`** — `.ai-framework/knowledge-base/{STACK}/` (or `knowledge-base/{STACK}/` in framework-repo mode)
- **`CONTAINER_KB_DIR`** — container-level KB root
- **`KB_DIR`** — project-level KB root
- **`PROJECT_CODE_ROOT`** — where source code lives

### Step 2: Check KB first

Search all three KB layers for the topic before researching:

1. **Framework KB** — `{FRAMEWORK_KB_DIR}/_index.md` trigger keywords
2. **Container KB** — `{CONTAINER_KB_DIR}/README.md` (or `_index.md` if present) for shared project patterns
3. **Project KB** — `{KB_DIR}/README.md` (or `_index.md` if present) for project-specific patterns
4. **Conventions** — `{CONTAINER_KB_DIR}/conventions.md` and `{KB_DIR}/conventions.md` if present

If the topic is already documented in any layer:
- Report "already documented at {path}" and stop — do not re-research.

### Step 3: Web research

Use WebSearch/WebFetch to find:
- Current best practices and recommended approaches
- Common architectural patterns for this problem type
- Known pitfalls and anti-patterns
- Authoritative references (official docs, well-known engineering blogs)

Minimum 3 authoritative sources. Prefer official documentation and established engineering references over tutorials.

### Step 4: Codebase scan

Search `{PROJECT_CODE_ROOT}` for any existing partial implementation or related patterns:
- Grep for relevant keywords, types, protocols, or file names
- Identify how similar patterns are already handled in the project
- Note any conventions the project already follows that should be respected

This grounds the research in the project's actual context rather than producing a generic web summary.

### Step 5: Synthesize

Combine web findings and codebase context into a coherent pattern summary:
- What approach to use and why
- How it fits the project's existing patterns
- Key gotchas and non-obvious pitfalls
- Any project-specific adaptations to the general approach

### Step 6: Route output

Decide based on what was found:

| Finding type | Route |
|-------------|-------|
| Reusable pattern (architecture, integration, multi-project value) | Delegate to `add-kb-doc` |
| Short project rule (one-liner, session-discovered, project-specific) | Delegate to `capture-convention` |
| Both apply | Split: pattern to `add-kb-doc`, rule to `capture-convention` |

**Never write KB files directly** — always delegate to `add-kb-doc` or `capture-convention`.

For `add-kb-doc` routing, use the standard category table:

| Category | When |
|----------|------|
| `architecture/` | Design patterns, data flows, system-level approaches |
| `integrations/` | Third-party library or complex platform SDK patterns |
| `ui-patterns/` | UI framework patterns or styling approaches |
| `best-practices/` | Universal rules (append to existing standards doc) |

### Step 7: Report

Output the following:

```text
Topic researched: {topic}
Where captured:   {KB doc path or conventions.md entry}

Key findings:
  - {finding 1}
  - {finding 2}
  - {finding 3}
  - {finding 4 (optional)}
  - {finding 5 (optional)}

Sources consulted: {N} web sources + codebase analysis
```

## Important Distinction

This skill differs from `research-kb-topic`:

| Skill | Purpose |
|-------|---------|
| `research-kb-topic` | Scans dependency manifests for technology coverage gaps — answers "what libraries do we use that lack KB docs?" |
| `research-implementation` | Targets feature/pattern questions before planning or building — answers "how do we implement offline sync?", "what's the right token refresh pattern?" |

Use `research-implementation` when the question is about **how to build something**, not about what technology coverage is missing.

## Output

- KB doc created via `add-kb-doc` delegation (full doc + compact variant + index update), or
- Conventions entry via `capture-convention` delegation, or both
- Research summary returned to caller for use in plan/task context
