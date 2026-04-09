# Audit Report Template

Report format for Phase 9 of the `audit-and-plan` skill.
Present this to the user after completing the audit.

---

```
✅ Audit Complete
=================

Projects audited: {list of folder (stack) pairs}
Codebase class:   {A / B / C}

What it does:
{2-3 sentence plain-language product summary}

── Per-project KB docs ──────────────────────────

{For each project:}
{folder}/ ({stack})
  + technical/architecture/project-overview.md
  + technical/architecture/architecture-patterns.md
  + technical/architecture/coding-conventions.md
  + technical/integrations/{...}.md   (platform-specific)
  {if Class B/C:}
  + ai-patterns/known-issues.md

── Shared KB docs (container root) ─────────────

  + product/domain/product-overview.md
  + product/domain/{domain-concept}.md  (one per concept)
  + technical/api/{shared-api}.md
  + technical/ble/{ble-spec}.md         (if applicable)
  + technical/data-models/{model}.md    (if applicable)
  + engineering/best-practices/{standard}.md  (if applicable)

── Per-project problems ─────────────────────────

{For each project:}
{folder}/
  🔴 Critical: {N}  — {brief list or "none"}
  🟠 High:     {N}  — {brief list or "none"}
  🟡 Medium:   {N}
  🟢 Low:      {N}

─────────────────────────────────────────────────

AGENTS.md:  {list of updated files}
KB indexes: updated ✓
{if plan created:}
Improvement plan: {plan-slug}
  Phase 1: {N} tasks
  Phase 2: {N} tasks

Next steps:
{if plan: /execute-plan {plan-slug}}
{if no plan: Start working — agents are now onboarded to this project.}
```
