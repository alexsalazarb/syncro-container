# Hub Resolution

Resolve the hub path using this priority order:

| Priority | Condition | Hub root | Mode |
|----------|-----------|----------|------|
| 1 | `WORK_PLANS_PATH` is set in `.ai-framework.config` | `$WORK_PLANS_PATH` | external |
| 2 | `git config --global --get ai-framework.workplanspath` returns a path | that path | external |
| 3 | `LAYOUT=container` and tiers 1–2 are empty | `$PLANS_DIR` (container self-hosts) | self-hosted |
| 4 | None of the above (single layout, no explicit path) | Hub not configured | — |

Store the resolved path in `HUB_PATH`.

**Hub types:**
- **external** — `HUB_PATH` differs from `{PLANS_DIR}/` (a separate repo or directory)
- **self-hosted** — `HUB_PATH` equals `{PLANS_DIR}/` (container self-hosting; no push needed)
