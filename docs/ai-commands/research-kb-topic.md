
# Research KB Topic

## Context Required
LOW-CONTEXT: AGENTS.md, dependency manifest files

## Triggers

### Automatic
- After `audit-and-plan` completes: if framework KB gaps detected, suggest `/research-kb-topic --scan`
- After `cleanup-kb` runs: if used libraries lack KB docs, suggest `/research-kb-topic`

### Manual
- `/research-kb-topic`
- `/research-kb-topic --scan`
- `/research-kb-topic [technology]`
- "research KB for this project"
- "populate KB for [technology]"
- "what KB docs are we missing"
- "fill KB gaps"

## Instructions

### Step 0: Resolve context

Follow [`common/resolve-project-context.md`](../common/resolve-project-context.md):

Read `.ai-framework.config` to get `STACK`. Resolve repo root.

Resolved variables:
- **`FRAMEWORK_KB_DIR`** — `.ai-framework/knowledge-base/{STACK}/`
- **`PROJECT_CODE_ROOT`** — where source code lives
- **`STACK`** — from config

**Framework-repo mode**: If `.ai-framework.config` is absent but `knowledge-base/` exists at
repo root, you are inside the framework repo. Ask for the target stack.
Set `FRAMEWORK_KB_ROOT = knowledge-base/{stack}/`.

### Step 1: Determine mode

**Single mode** (default): User specified a technology, or skill was suggested after a
specific KB miss. Go to Step 4.

**Scan mode** (`--scan` flag, or "what KB docs are we missing", "fill KB gaps"):
Go to Step 2.

---

### Step 2: Scan dependencies (scan mode)

Read the project's dependency manifest under `{PROJECT_CODE_ROOT}`.

→ See [dependency-tech-mapping.md](references/dependency-tech-mapping.md) for the per-stack
  file table, name normalization rules, and standard-library skip list.

Extract a flat list of technology names with normalized aliases.

### Step 3: Diff against framework KB

Read `{FRAMEWORK_KB_DIR}/_index.md` trigger keywords AND list all `.md` files
in `{FRAMEWORK_KB_DIR}/`.

For each technology from Step 2, check:
1. Does any file name contain the technology name or alias?
   (e.g., `persistence-patterns.md` covers CoreData)
2. Do any `_index.md` trigger keywords match?
3. Is the technology in the core-platform skip list?
   (see [dependency-tech-mapping.md § Standard Library](references/dependency-tech-mapping.md))

Classify each:
- **COVERED** — matching KB doc exists
- **MISSING** — no matching doc found
- **CORE** — core platform (too basic for a dedicated doc)

Present the gap report and wait for confirmation:

```text
KB Gap Report — {PROJECT_DIR} ({STACK})
========================================

MISSING (no framework KB doc found):
  [1] CoreData         → expected: integrations/coredata-patterns.md
  [2] Kingfisher       → expected: integrations/kingfisher-patterns.md

COVERED (already in framework KB):
  - RxSwift            → integrations/rxswift-patterns.md ✓
  - Realm              → integrations/realm-patterns.md ✓

CORE (standard library — skip):
  - Foundation, UIKit, SwiftUI

Research all MISSING items? (y)
Research selected items? (list numbers)
Cancel? (n)
```

### Step 4: Research a technology

For each confirmed technology (scan) or the user-specified technology (single):

**4a. Web research** (MUST do):

Use WebSearch/WebFetch to gather current best practices, API patterns, architecture
recommendations, and common pitfalls.

→ See [research-template.md](references/research-template.md) for required research areas,
  search query patterns, and quality criteria.

**4b. Project code analysis** (if in consumer repo):

Read project code under `{PROJECT_CODE_ROOT}` to understand how the technology is used:
- Which APIs/patterns appear in practice
- How it integrates with the project's architecture
- Common files involving this technology

This makes the KB doc context-aware — not just a generic web summary.

**4c. Determine category**:

| Category | When |
|----------|------|
| `integrations/` | Third-party library or complex platform SDK |
| `architecture/` | Design pattern or core framework feature (concurrency, networking) |
| `ui-patterns/` | UI framework, layout library, or styling tool |
| `best-practices/` | Append to existing standards doc (rare — only for broad rules) |

### Step 5: Generate KB content

Compose the doc following the template in
[`add-kb-doc/references/kb-doc-template.md`](../add-kb-doc/references/kb-doc-template.md):

- Title, last updated, context/trigger keywords
- Overview — what it is, when to use it
- Core patterns — with code examples in the stack's language
- Gotchas — non-obvious pitfalls from web research + project code
- Related — links to related KB docs in the same stack

→ See [research-template.md § Quality Criteria](references/research-template.md) for
  minimum content requirements.

### Step 6: Delegate to `add-kb-doc`

Pass the generated content to `/add-kb-doc`, which handles:
1. Writing the file to `knowledge-base/{STACK}/{category}/{filename}.md`
2. Generating `.compact.md` variant if over 100 lines
3. Updating `_index.md` with trigger keywords
4. Updating `README.md` catalog

Do NOT duplicate these steps — `add-kb-doc` owns the full lifecycle.

### Step 7: Report

**Single mode:**
```text
KB Topic Researched: {technology}
  → knowledge-base/{STACK}/{category}/{filename}.md
  → Delegated to add-kb-doc for compact variant and indexing.

Sources consulted: {N} web pages + project code analysis
```

**Scan mode:**
```text
KB Gap Scan Complete — {PROJECT_DIR} ({STACK})
===============================================

Researched and populated:
  ✓ CoreData      → integrations/coredata-patterns.md
  ✓ Kingfisher    → integrations/kingfisher-patterns.md
  ✗ SnapKit       → skipped (user declined)

{N} new KB docs added to framework.
All future {STACK} projects will benefit.

Commit in .ai-framework/ submodule:
  cd .ai-framework && git add knowledge-base/ && git commit -m "feat({stack}): add KB docs"
```

## Output

- Gap report (scan mode) showing technologies vs. KB coverage
- KB docs created via `add-kb-doc` delegation (with compact variants and indexing)
- Commit instructions for the framework submodule

_Recommended: Run with a high-capability model for best research quality._
