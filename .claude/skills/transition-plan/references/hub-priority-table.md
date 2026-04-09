# Hub Priority Table

Resolve the hub location using this 4-tier priority:

| Priority | Condition | Hub root | Mode |
|----------|-----------|----------|------|
| 1 | `WORK_PLANS_PATH` is set in `.ai-framework.config` | `$WORK_PLANS_PATH` | external |
| 2 | `git config --global --get ai-framework.workplanspath` returns a path | that path | external |
| 3 | `LAYOUT=container` and tiers 1–2 are empty | `$PLANS_DIR` (container self-hosts) | self-hosted |
| 4 | None of the above | **Abort** — no hub available | — |
