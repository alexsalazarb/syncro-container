# State Machine and Transition Validation

## Allowed Transitions

| From | Allowed To |
|------|-----------|
| backlog | planned, cancelled |
| planned | active, deprioritized, cancelled |
| active | completed, deprioritized, cancelled |
| deprioritized | planned, cancelled |
| completed | *(terminal — no transitions allowed)* |
| cancelled | *(terminal — no transitions allowed)* |

## Validation Rules

- If `CURRENT_STATE` equals `TARGET_STATE`, report "Plan is already in {state}" and stop.
- If `CURRENT_STATE` is `completed` or `cancelled`, report "Plan is in terminal state '{state}' — no transitions allowed" and stop.
- If `TARGET_STATE` is not in the allowed set for `CURRENT_STATE`, report the error with the list of allowed transitions and stop.
- If `TARGET_STATE` is `deprioritized` or `cancelled` and `--reason` was not provided, prompt the user for a reason before continuing.
