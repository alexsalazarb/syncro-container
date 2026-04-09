# Bug Plan Directory Structure

Bug plans use **flat layout** (no phases for ≤5 tasks).

## Single-project or `LAYOUT=single`

```
{PLANS_DIR}/{slug}/
  overview.md        ← from references/bug-overview.md
  investigation.md   ← findings from Step 2
  task-01-{fix}/
    task.md          ← from references/bug-task.md
    status.md        ← from references/bug-task-status.md
  task-02-regression-tests/
    task.md
    status.md
```

## Container layout (multi-project bug)

```
{PLANS_DIR}/{master-slug}/         ← master plan at container root
  overview.md

{PLANS_DIR}/{project-dir-A}/{slug}/  ← local bug plan for project A
  overview.md   (Master Plan: {master-slug})
  investigation.md
  task-01-...

{PLANS_DIR}/{project-dir-B}/{slug}/  ← local bug plan for project B
  overview.md   (Master Plan: {master-slug})
  investigation.md
  task-01-...
```

## Examples

### Single-project bug

```
.plans/bugfix-login-timeout/
  overview.md
  investigation.md
  task-01-fix-timeout/
    task.md
    status.md
  task-02-regression-tests/
    task.md
    status.md
```

### Cross-project bug (container layout with iOS + Rails)

```
.plans/bugfix-sync-overlap/           ← master plan
  overview.md

.plans/rails/bugfix-sync-overlap/     ← Rails local plan
  overview.md  (Master Plan: bugfix-sync-overlap)
  investigation.md
  task-01-fix-api-validation/
  task-02-regression-tests/

.plans/ios/bugfix-sync-overlap/       ← iOS local plan
  overview.md  (Master Plan: bugfix-sync-overlap)
  investigation.md
  task-01-fix-error-handling/
```
