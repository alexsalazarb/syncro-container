# Bug Triage Flowchart

## When to Use /create-bug-plan

```
Bug discovered
  │
  ├─ Reported externally (ticket, customer, monitoring)?
  │   │
  │   ├─ Quick fix (1 file, obvious cause, <30 min)?
  │   │   └─ Yes → Fix inline. Branch + PR + regression test. No plan needed.
  │   │
  │   └─ Needs investigation or touches multiple files?
  │       └─ Yes → Use /create-bug-plan (this skill)
  │
  ├─ Found during active plan execution (mid-task)?
  │   │
  │   ├─ Bug is in the code you're currently building?
  │   │   ├─ Small (fix + test fits in current task) → Fix inline, document in status.md
  │   │   └─ Larger → Add a defect task via execute-task Step 5a
  │   │
  │   ├─ Bug is in existing code you're building on top of?
  │   │   ├─ Blocks your task → Add a defect task to current plan (Step 5a)
  │   │   └─ Doesn't block → Log in plan Defects table. Triage separately.
  │   │
  │   └─ Bug is unrelated to the plan?
  │       └─ Always → Log a ticket. Do NOT add to current plan.
  │
  └─ Found during QA / code review of plan deliverables?
      ├─ Regression in plan's own work → /add-defect {plan-slug}
      └─ Pre-existing bug surfaced by QA → Log ticket, triage separately.
```

## How /create-bug-plan Differs from /create-plan

| Aspect | /create-plan (Feature) | /create-bug-plan (Bug Fix) |
|--------|----------------------|---------------------------|
| First step | Explore codebase | **Investigate root cause** |
| Investigation | Optional context | **Mandatory documented artifact** |
| Regression test | Optional | **Mandatory task** |
| Phases | Multi-phase with contracts | Usually 1-2 phases, flat layout |
| Consumer verification | N/A | **Verify all consumers handle the fix** |
| Cross-project detection | Explicit `--master-plan` flag | **Auto-detected during investigation** |
| Typical size | 5-15 tasks | 2-5 tasks |
| Kill criteria | Business/technical risks | Fix worse than bug, wrong root cause |
