# Review Report Template

Use this structure for every `framework-review` output.
Replace `{placeholders}` with actual content. Remove sections that don't apply.

---

```markdown
# Framework Review — {scope description}

**Date**: {YYYY-MM-DD}
**Scope**: {single component path | component type | full framework | diff range}
**Reviewer**: framework-reviewer
**Components reviewed**: {count} agents, {count} skills, {count} KB docs, {count} plans

---

## Executive Summary

{2-3 sentences: what was reviewed, overall health assessment (green/yellow/red),
count of BLOCKERs and WARNINGs.}

---

## What Works Well

- {Strong design choice — be specific, cite the component}
- {Max 5 bullets. If nothing stands out, write "No standout strengths identified — see findings."}

---

## Findings

### Agent Design (Dimension A)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| A-{N} | {BLOCKER/WARNING/NOTE/QUESTION} | `{path}` | {problem} | {fix} | {impact} |

{If no findings: "No agent design issues found."}

### Skill Design (Dimension B)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| B-{N} | {severity} | `{path}` | {problem} | {fix} | {impact} |

### Knowledge Base (Dimension C)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| C-{N} | {severity} | `{path}` | {problem} | {fix} | {impact} |

### Plan System (Dimension D)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| D-{N} | {severity} | `{path}` | {problem} | {fix} | {impact} |

### Token & Context Efficiency (Dimension E)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| E-{N} | {severity} | `{path}` | {problem} | {fix} | {impact} |

**Token metrics:**

| Component | Current lines | Budget | Status |
|-----------|--------------|--------|--------|
| `{path}` | {N} | {target} | {over/under/at budget} |

### Maintainability & Scalability (Dimension F)

| ID | Severity | Component | Problem | Fix | Impact |
|----|----------|-----------|---------|-----|--------|
| F-{N} | {severity} | `{path}` | {problem} | {fix} | {impact} |

---

## Architecture Recommendations

{Cross-cutting recommendations that span multiple dimensions.
Each should be 2-4 sentences: the pattern observed, why it matters, and the proposed change.}

1. {Recommendation}
2. {Recommendation}

---

## Priority Actions

Ranked by impact. Execute in order.

| Rank | Finding ID | Action | Effort | Expected outcome |
|------|-----------|--------|--------|-----------------|
| 1 | {ID} | {specific action} | {low/medium/high} | {what improves} |
| 2 | {ID} | {specific action} | {effort} | {outcome} |
| 3 | {ID} | {specific action} | {effort} | {outcome} |

---

## Follow-ups

- [ ] {Skill or action to run next — e.g., `/framework-builder improve {path}`}
- [ ] {e.g., `/check-kb-index` to fix index drift}
- [ ] {e.g., `/create-plan framework-optimization` for systemic issues}
```
