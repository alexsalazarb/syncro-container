# Hub Resolution

Resolve the work-plans hub directory for skills that read or write master plan data.

Requires `LAYOUT`, `PLANS_DIR`, `WORK_PLANS_PATH`, and `WORK_PLANS_BRANCH` from
[`config-defaults.md`](config-defaults.md).

---

## 4-Tier Priority

| Priority | Condition | Hub root | Mode |
|----------|-----------|----------|------|
| 1 | `WORK_PLANS_PATH` is set in `.ai-framework.config` | `$WORK_PLANS_PATH` | external |
| 2 | `git config --global --get ai-framework.workplanspath` returns a path | that path | external |
| 3 | `LAYOUT=container` and tiers 1–2 are empty | `$PLANS_DIR` (container self-hosts) | self-hosted |
| 4 | None of the above (single layout, no explicit path) | Hub unavailable | — |

Store the resolved path as `HUB_ROOT`.

## Hub Modes

- **external** — `HUB_ROOT` is a separate repo or directory (differs from `PLANS_DIR`). Requires git fetch/push for sync.
- **self-hosted** — `HUB_ROOT` equals `PLANS_DIR` (container self-hosting). No push needed; commits stay local.

## State Directories

Master plans are organized by lifecycle state inside the hub:

```
{HUB_ROOT}/plans/
  backlog/
  planned/
  active/
  completed/
  deprioritized/
  cancelled/
```

To locate a plan by slug, scan all six state directories:

```bash
for state_dir in backlog planned active completed deprioritized cancelled; do
  if [ -d "$HUB_ROOT/plans/$state_dir/$slug" ]; then
    CURRENT_STATE="$state_dir"
    break
  fi
done
```

## Tier 4 Handling

When hub is unavailable, the calling skill decides the behavior:

- **Read-only skills** (e.g. plans-status): degrade gracefully — skip hub sections.
- **Write skills** (e.g. create-master-plan): abort with guidance to set `WORK_PLANS_PATH`.
- **Automatic callers** (e.g. execute-task → sync-master-plan): log warning, never block.

## Usage

Reference from a skill's Step 0:

```markdown
Resolve hub directory per [`common/hub-resolution.md`](../common/hub-resolution.md).
```
