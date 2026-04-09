# PR Cleanup — Polling Loop

Used by [pr-cleanup](../SKILL.md) Step 7.

---

## Poll loop — run until clean or timeout

After pushing, enter the poll loop. This loop runs **until the PR is fully
clean or the timeout expires** — it does not exit early based on a fixed
iteration count.

On each iteration:

```bash
NOW=$(date +%s)
ELAPSED=$((NOW - START_TIME))
if [ $ELAPSED -ge $TIMEOUT_SECONDS ]; then
  echo "Timeout reached (${timeout_minutes}m). Reporting current status."
  break
fi
```

### Per-iteration sequence

1. **Re-check CodeRabbit status** (Step 2 logic — sort by updatedAt):
   - If rate limited → parse wait time, sleep, `@coderabbitai review`, continue
   - If processing → sleep 60s, continue (do not triage yet)
   - If paused → `@coderabbitai resume`, sleep 30s, continue

2. **Re-fetch unresolved threads** (Step 3 GraphQL)
   - If new unresolved threads appeared → go back to Step 4
   - If disagreement threads have new CodeRabbit replies → handle per Step 4d

3. **Check CI**:

   ```bash
   gh pr checks "$PR_NUMBER"
   ```

   - If CI is failing on a non-flaky check → investigate, fix, re-run Step 5/6
   - If CI is failing on a flaky/infra check → note it, keep polling

4. **Done condition** — exit the loop when ALL of:
   - `unresolved_threads == 0`
   - All CI checks pass (exit code 0 from `gh pr checks`)
   - CodeRabbit is not currently processing

5. If not done: `sleep 60` and repeat.
