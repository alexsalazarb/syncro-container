---
name: merge-framework
description: >-
  Merges AI framework conventions into an existing CLAUDE.md / AGENTS.md that
  predates the framework. Run once after adding .ai-framework as a submodule to
  a project that already has its own CLAUDE.md. Preserves all existing project
  content and adds framework sections (Quick Triggers, Skills, KB path).
---

# Merge Framework

## Context Required
LOW-CONTEXT: reads CLAUDE.md and framework template — no prior context needed

## Triggers

### Automatic
- None

### Manual
- `/merge-framework`
- "merge the framework into my CLAUDE.md"
- "my CLAUDE.md doesn't know about the framework"
- "wire up the AI framework"

## Instructions

### Step 1: Check if merge is needed

```bash
grep -l "pre-commit-check\|QUICK TRIGGERS\|\.claude/skills" CLAUDE.md AGENTS.md 2>/dev/null
```

If both files already contain framework references → tell the user the framework is already wired and stop.

---

### Step 2: Detect the stack

```bash
.ai-framework/scripts/init-repo.sh --stack   # check what stack was detected
```

Or detect manually:
- `Podfile` or `Package.swift` → `swift`
- `package.json` with `next` or `react` → `react`
- `Gemfile` → `rails`
- `requirements.txt` or `pyproject.toml` → `python`
- Otherwise → `generic`

---

### Step 3: Read both sources

1. Read the **existing** `CLAUDE.md` in full — this is the project's rules, must be fully preserved
2. Read `.ai-framework/templates/AGENTS.{stack}.md` — this is the framework template to merge from

---

### Step 4: Produce the merged CLAUDE.md

Merge rules:
- **Keep all existing content** — every rule, constraint, architecture note, command stays exactly as written
- **Add framework sections** that are missing. Sections to add if absent:

**At the top** — add Quick Triggers block:
```markdown
## QUICK TRIGGERS (Memorize These)

**Session Start**: Check `docs/kb-container/ai-patterns/mistake-log.md` for patterns to avoid.

**During Session**:
- User says "commit/stage" → run `pre-commit-check`
- You get corrected → run `log-mistake`
- KB lookup fails → after solving, run `document-solution`
- You modify KB files → run `check-kb-index`

**Session End** (user says "done/thanks/bye"): Run `session-end-checklist`
```

**Knowledge Base section** — add if absent:
```markdown
## Knowledge Base

Index: `docs/kb-container/README.md`

Check before exploring code. If KB lookup fails and you solve via code search, run `document-solution`.

**KB-First Rule**: Before exploring any subsystem, check `docs/kb-container/README.md`. If a doc exists, read it first.
```

**Skills & Agents section** — add if absent:
```markdown
## Skills & Agents

Skills: `.claude/skills/` | `.agents/skills/`
Agents: `.claude/agents/`

| Skill | When |
|-------|------|
| `pre-commit-check` | Before commit/staging |
| `session-end-checklist` | Session ending |
| `log-mistake` | User corrects you |
| `check-kb-index` | After KB file changes |
| `save-session` | Long session (20+ turns) |
| `document-solution` | KB miss → solved, 3+ files, 5+ exchanges |
| `list-skills` | Discover available automation |
| `create-plan` | Starting a new feature/project |
| `execute-task` | Working on a specific plan task |
| `merge-framework` | Wire framework into existing CLAUDE.md |

*Invoke with `/skill-name` or natural language.*
```

**Mandatory Auto-Triggers section** — add if absent:
```markdown
## Mandatory Auto-Triggers

| Trigger | Action | Why |
|---------|--------|-----|
| User says "commit", "stage", "git add" | `pre-commit-check` | Catch issues before they land |
| User corrects you | `log-mistake` | Build patterns |
| Modified KB/index files | `check-kb-index` | Keep index current |
| User ending session | `session-end-checklist` | Catch missed work |
```

**At the bottom** — add REMEMBER block:
```markdown
## REMEMBER (End of File)

Before responding, check:
1. **Am I being corrected?** → Run `log-mistake`
2. **Is user committing/staging?** → Run `pre-commit-check`
3. **Is user ending session?** → Run `session-end-checklist`
```

Do NOT duplicate sections that already exist in the project's CLAUDE.md.
Do NOT remove or rewrite any existing content.

---

### Step 5: Write merged files

Write the merged result to:
- `CLAUDE.md` — Claude Code entry point
- `AGENTS.md` — all other AI tools

Both files must be identical.

---

### Step 6: Initialize KB structure if missing

```bash
ls docs/kb-container/README.md 2>/dev/null || echo "missing"
```

If missing, create the base KB structure (Layer 2 + Layer 3 share this tree in **single** layout; matches `init-repo.sh` / `resolve-project-context`):
```bash
mkdir -p docs/kb-container/product/{domain,features}
mkdir -p docs/kb-container/technical/{architecture,api,ble,data-models,integrations}
mkdir -p docs/kb-container/engineering/{best-practices,security,testing}
mkdir -p docs/kb-container/ai-patterns/sessions
```

Copy KB README from framework:
```bash
cp .ai-framework/knowledge-base/README.md docs/kb-container/README.md
cp .ai-framework/knowledge-base/shared/ai-patterns/mistake-log.md docs/kb-container/ai-patterns/mistake-log.md
```

---

### Step 7: Commit

```bash
git add CLAUDE.md AGENTS.md docs/kb-container/
git commit -m "chore: merge AI framework conventions into CLAUDE.md"
```

---

### Step 8: Report

```
✅ Framework merged
===================

CLAUDE.md:  updated (framework sections added, existing content preserved)
AGENTS.md:  updated (identical to CLAUDE.md)
KB:         initialized at docs/kb-container/

Sections added:
+ QUICK TRIGGERS
+ Knowledge Base
+ Skills & Agents
+ Mandatory Auto-Triggers
+ REMEMBER

Next steps:
- Run /list-skills to verify skills are loaded
- Customize docs/kb-container/ with project-specific KB docs
- Run /check-kb-index after adding KB docs
```

## Output

- Merged `CLAUDE.md` and `AGENTS.md` with all framework conventions added
- Existing project content fully preserved
- KB structure initialized
- Changes committed
