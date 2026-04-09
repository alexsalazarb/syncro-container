# Component Templates

Quick-reference templates for the `framework-builder` skill.
These are structural skeletons — the skill fills in role-specific content.

---

## Agent Definition Template

```markdown
---
name: {slug}
description: >-
  {Role summary in 1-3 sentences. End with "Use for..." or "Use when..."}
# Capability tier: {tier} ({justification})
# Claude Code: model: {model} | Other tools: {guidance}
model: {opus|sonnet|haiku}
# Access level: {access description}
tools:      # omit for full-access agents
  - {Tool1}
---

# {Human Title}

## Role
{2-4 word role title}

## Responsibilities

- {4-8 bullet points, each starting with a verb}

## Workflow

1. **Read `.ai-framework.config`** — resolve variables per `resolve-project-context.md`
2. **Read `{AGENTS_MD}`** — load project conventions
3. **Load KB (3-layer, tiered)**:
   a. **Layer 1 — Framework**: Read `{FRAMEWORK_KB_DIR}/_index.md`. Match triggers. Compact first.
   b. **Layer 2 — Container**: Read `{CONTAINER_KB_DIR}/README.md`
   c. **Layer 3 — Project**: Read `{KB_DIR}/README.md`
   d. Load only matching docs
4. **Plans awareness**: Read `{PLANS_DIR}/README.md`
5. {Role-specific steps — typically 3-6 additional steps}

## KB Escalation Protocol

During Step 3, if no KB doc matches the task topic:
1. {How this agent handles the gap}
2. {Whether it backfills or delegates backfill}

## Guidelines

- {3-8 role-specific guidelines}
```

**Budget**: Target ≤ 80 lines. Hard limit 120 lines.

---

## Skill Definition Template

```markdown
---
name: {slug}
description: >-
  {What the skill does in 1-3 sentences. End with "Use when..." trigger.}
---

# {Human Title}

## Context Required
{META|LOW-CONTEXT|HIGH-CONTEXT|FULL-CONTEXT}: {what is needed}

## Triggers

### Automatic
{Bullet list of conditions, or "None (manual only)"}

### Manual
- `/{slug}`
- `/{slug} [arguments]`
- "{natural language phrases}"

## Instructions

### Step 0: Load Project Config
Read `.ai-framework.config` from the project root. Extract variables per
[`common/resolve-project-context.md`](../common/resolve-project-context.md).

### Step 1: {Action}
{Clear instructions with inputs and outputs per step}

### Step N: Report
{Present results to the user}

## Output

- {Bullet list of artifacts and updates}
```

**Budget**: Target ≤ 150 lines. Hard limit 250 lines (use references/ for overflow).

---

## KB Index Entry Template (`_index.md` row)

```markdown
| {doc-name} | {2-5 trigger keywords} | `{path}.compact.md` | `{path}.md` | {one-line description} |
```

---

## Plan Overview Template

See `skills/create-plan/references/plan-overview.md` for the canonical template.

---

## Audit Report Template

```markdown
# Framework Audit Report — {YYYY-MM-DD}

## Summary
- Agents: {count} ({issues})
- Skills: {count} ({issues})
- KB stacks: {count} ({issues})
- Overall health: {green|yellow|red}

## Token Optimization Opportunities
| Component | Lines | Issue | Est. Savings |
|-----------|-------|-------|-------------|

## Consistency Issues
| Component | Issue | Fix |
|-----------|-------|-----|

## Missing Components
- {gaps}

## Recommendations (prioritized)
1. {action} — {effort estimate}
```
