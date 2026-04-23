---
name: capture-convention
description: >-
  Appends a short project convention to conventions.md at the appropriate KB layer (L2 or L3).
  Lighter than add-kb-doc (no full doc, no index update). Different from log-mistake (project
  rules, not AI errors). Use when a project-specific rule is discovered during a session.
---

# Capture Convention

## Context Required
LOW-CONTEXT: AGENTS.md only

## Triggers

### Automatic (MUST detect proactively - do NOT wait for explicit request)

**Detection Phrases** - If user says ANY of these, trigger immediately:
- "remember that we [rule]"
- "note that this project [rule]"
- "add this as a convention"
- "we always [rule] here"
- A correction reveals a project-specific rule that couldn't have been known from existing docs

**Note**: When a correction reveals a project-specific rule, run BOTH this skill AND `log-mistake` — they serve different purposes. `capture-convention` records the project rule for future agents; `log-mistake` records the AI error pattern.

**Do NOT wait for**: "capture this", "add a rule", or explicit skill invocations.
**Do**: Detect convention-discovery phrases and log proactively.

### Manual
- `/capture-convention`
- "capture this convention"
- "add a project rule"
- "add a convention"

## Instructions

### Step 1: Resolve project context

Read `.ai-framework.config` and resolve `CONTAINER_KB_DIR` and `KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

If config is absent, use defaults: `CONTAINER_KB_DIR=docs/kb-container`, `KB_DIR=docs/kb-container`.

---

### Step 2: Determine target layer

Ask if ambiguous:

- **L2** (`CONTAINER_KB_DIR/conventions.md`): rule applies across all projects in this repo
- **L3** (`KB_DIR/conventions.md`): rule is specific to one project

Prompt when unclear:
```
Is this rule shared across all projects, or specific to [project name]?
```

---

### Step 3: Determine category and severity

Infer from context, or ask:

**Category** (choose one):
- `code-quality` — style, naming, structure, formatting rules
- `security` — auth, secrets, data handling requirements
- `testing` — test coverage, assertion, test organization rules
- `architecture` — layering, ownership, module/pattern constraints
- `git` — branching, commit message, PR, review rules
- `performance` — caching, query, load-time constraints

**Severity** (choose one):
- `critical` — violating this breaks things or causes data loss
- `high` — violating this causes rework or significant bugs
- `medium` — good to know; missing it degrades quality
- `low` — nice to have; missing it is acceptable

---

### Step 4: Ensure conventions.md exists

Check whether `{target_path}/conventions.md` exists. If absent, create the stub:

```markdown
# Project Conventions

Short project-specific rules captured during sessions. Append via `/capture-convention`.
For complex patterns, use `/document-solution`. For AI error patterns, use `/log-mistake`.

---
```

---

### Step 5: Append the entry

Add a single line to `{target_path}/conventions.md`:

```
- **[{category}|{severity}]** {convention text} _({YYYY-MM-DD})_
```

Example:
```
- **[architecture|critical]** Never call repository methods directly from a SwiftUI view — always go through a ViewModel. _(2026-04-22)_
```

---

### Step 6: Confirm to user

```
Convention captured: {target_path}/conventions.md
[{category}|{severity}] {convention text}
```

---

### Step 7: Do NOT run check-kb-index

`conventions.md` is a lightweight append-only log, not a full KB document. It does not need an index update.

## Output

- Appended entry in `{target_path}/conventions.md` (created if absent)
- Confirmation message with file path and entry details
- No index update required
