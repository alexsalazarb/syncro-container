# Severity Classification

Classify the bug before investigating — this determines how deeply to investigate.

| Severity | Criteria | Workflow |
|----------|----------|----------|
| **P0 — Critical** | Data loss/corruption, security breach, system down, all users affected | **Hotfix track**: Skip Feature History (Step 2b). Minimal plan: 1 fix task + 1 regression test. |
| **P1 — High** | Feature broken for many users, no workaround, or SLA breach | Full investigation, expedited. |
| **P2 — Medium** | Feature degraded, workaround exists, limited impact | Standard flow — full investigation + plan. |
| **P3 — Low** | Cosmetic, single user, edge case with easy workaround | Standard flow, lower priority. |

**P0 shortcut**: Skip Steps 2b (Feature History) and 2d (Cross-Project Assessment). Speed matters more than completeness. Backfill investigation after the hotfix ships.
