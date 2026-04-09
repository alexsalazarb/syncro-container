---
name: pr-cleanup
description: >-
  Fetch the current PR, triage all unresolved CodeRabbit review threads (fix
  actionable issues, resolve outdated threads, reply to disagreements), commit
  all fixes in a single commit, push, then loop every 60 seconds until all CI
  checks pass and no unresolved threads remain. Handles CodeRabbit rate limits
  (parses wait time, sleeps, re-triggers review) and processing delays (waits
  for CodeRabbit to finish before triaging). Summarises results in a final PR
  comment.
---

# PR Cleanup

## Context Required

LOW-CONTEXT: needs only `gh` CLI + repo

## Triggers

### Automatic
None (manual only — too destructive to run automatically)

### Manual
- `/pr-cleanup` — run with default 30-minute timeout
- `/pr-cleanup 15` — custom timeout in minutes

### Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `timeout_minutes` | `30` | Total wall-clock minutes before giving up and reporting current status. Unlike a fixed iteration count, the loop runs until fully clean or this timeout expires. |

---

## Instructions

### Step 1: Find the PR

Detect the current PR and set up timeout tracking.
→ See [PR discovery and timeout setup scripts](references/pr-scripts.md#step-1-find-the-pr)

If no PR found, exit with: "No open PR found for current branch."

---

### Step 2: Check CodeRabbit status

**ALWAYS sort by `updatedAt`, not `createdAt`.** CodeRabbit repeatedly edits
its first summary comment — the most-recently-updated comment is the current
state.

Check in this priority order:
1. **Paused** → `@coderabbitai resume`, sleep 30s, re-check
2. **Rate limited** → parse wait time, sleep (capped by budget), `@coderabbitai review`, re-check
3. **Currently processing** → sleep 60s, re-check (never triage during processing)
4. **Ready** → proceed to Step 3

→ See [full status-handling scripts and logic](references/coderabbit-triage.md#step-2-coderabbit-status-handling)

---

### Step 3: Fetch all review threads (GraphQL)

Fetch threads via GraphQL (up to 500) and PR-level review bodies. Filter to
unresolved threads: `select(.isResolved == false)`.

→ See [GraphQL query and fetch scripts](references/pr-scripts.md#step-3-fetch-review-threads)

---

### Step 4: Triage each unresolved thread

For every thread where `isResolved: false`, classify and act:

| Classification | Action |
|----------------|--------|
| **Outdated** (`isOutdated: true`) | Resolve via GraphQL mutation + reply "Outdated thread — resolved." |
| **Actionable code fix** (bug, type error, missing guard) | Read file, implement fix, note for commit message |
| **Disagreement / false positive** | Reply explaining why, do NOT resolve — wait for CodeRabbit |

During the poll loop, re-check disagreement threads for CodeRabbit replies:
agrees → resolve; pushes back with valid point → fix or reply with stronger justification.

→ See [full triage protocol and GraphQL mutation](references/coderabbit-triage.md#step-4-triage-protocol)

---

### Step 5: Run local tests

Auto-detect and run tests appropriate for this repo before committing.
→ See [test detection scripts](references/pr-scripts.md#step-5-run-local-tests)

If tests fail: fix them before committing.

---

### Step 6: Run pre-commit-check, then commit and push

Run `/pre-commit-check` on all changed files. If violations are reported:
fix them, then re-run. Do **not** commit until pre-commit-check is clean.
Pre-commit violations are always blocking — `/pr-cleanup` runs autonomously
and cannot pause for mid-execution user confirmation. If violations cannot be
auto-fixed, stop and report them so the user can fix and re-run `/pr-cleanup`.

→ See [commit and push scripts](references/pr-scripts.md#step-6-commit-and-push)

**Rules**: Stage specific files only — NEVER use `git add -A` or `git add .`.

---

### Step 7: Poll loop — run until clean or timeout

After pushing, enter the poll loop. Runs until the PR is fully clean or the
timeout expires — no fixed iteration count.

→ See [full polling loop logic](references/polling-loop.md)

**Done condition** — exit when ALL of:
- `unresolved_threads == 0`
- All CI checks pass (`gh pr checks`)
- CodeRabbit is not currently processing

---

### Step 8: Post final PR comment

Post a summary comment with counts of fixes, resolved threads, disagreements,
and CI status.

→ See [summary comment template](references/pr-scripts.md#step-8-post-final-pr-comment)

---

## Key Notes

- **Thread limit** — the query fetches up to 500 review threads
- **Sort by `updatedAt`** — always use `sort_by(.updatedAt) | last` for current state
- **Rate limit handling** — parse wait duration, sleep, post `@coderabbitai review`; default 10 min if unparseable
- **Processing gate** — never triage during "Currently processing new changes"
- **Loop until clean** — no iteration cap; runs until done or wall-clock timeout
- **Single commit per cycle** — collect all fixes into one commit
- **Targeted staging** — never `git add -A` or `git add .`
- **Pre-commit-check** — always run before committing

---

## Output

- Console progress during each step
- Final PR comment with summary table
- Exit message: "Cleanup complete — N fixes, N threads resolved, CI: passing/failing/timeout"

---

## Example Invocation

→ See [example session](references/pr-scripts.md#example-invocation) for a full walkthrough.
