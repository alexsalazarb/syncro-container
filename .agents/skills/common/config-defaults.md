# Config Defaults

Standard procedure for reading `.ai-framework.config` in plan-related skills.

---

## Read `.ai-framework.config`

Look for `.ai-framework.config` at the repository root.

```bash
cat .ai-framework.config
```

## Common Fields & Defaults

| Field | Default | Description |
|-------|---------|-------------|
| `PLANS_DIR` | `.plans` | Path to the plans directory |
| `PLANS_MODE` | `local` | `local` (plans live in the code repo) or `repo` (separate plans repo) |
| `LAYOUT` | `single` | `single` or `container` |
| `PROJECT_DIR` | `.` | Project folder (single layout only) |
| `STACK` | — | Tech stack identifier (single layout) |
| `PROJECTS` | — | Comma-separated `folder:stack[:mode]` entries (container layout) |

If the config file is missing, apply all defaults above.

## Hub-Related Fields

These are only needed by skills that interact with the work-plans hub:

| Field | Default | Description |
|-------|---------|-------------|
| `WORK_PLANS_PATH` | — | Explicit hub directory path |
| `WORK_PLANS_BRANCH` | `main` | Branch for hub git operations |

## Usage

Reference this doc from a skill's Step 0:

```markdown
Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md).
Extract: PLANS_DIR, PLANS_MODE, LAYOUT.
```

Then list any **skill-specific** fields inline (e.g. `PR_INTEGRATION`, `AUTO_MERGE_TASK_PRS`).
