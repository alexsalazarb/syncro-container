# CodeRabbit Status Handling & Triage Protocol

Used by [pr-cleanup](../SKILL.md) Steps 2 and 4.

---

## Step 2: CodeRabbit status handling

**ALWAYS sort by `updatedAt`, not `createdAt`.** CodeRabbit edits its first
summary comment with new statuses.

```bash
CR_COMMENT=$(gh pr view "$PR_NUMBER" --json comments \
  --jq '[.comments[] | select(.author.login == "coderabbitai")] | sort_by(.updatedAt) | last | .body // ""')
```

Check the result in this priority order:

### 2a. Paused

If `$CR_COMMENT` contains "paused":

```bash
gh pr comment "$PR_NUMBER" --body "@coderabbitai resume"
sleep 30
```

Then re-fetch and re-check.

### 2b. Rate limited

If `$CR_COMMENT` contains "rate limit" or "rate limited":

Parse the wait time from the message. CodeRabbit typically says something like
"rate limited for 30 minutes" or "please wait 10 minutes". Extract the number
and unit:

```bash
WAIT_STR=$(echo "$CR_COMMENT" | grep -oiE '[0-9]+ (minute|hour|second)s?' | head -1)
WAIT_NUM=$(echo "$WAIT_STR" | grep -oE '[0-9]+')
WAIT_UNIT=$(echo "$WAIT_STR" | grep -oiE 'minute|hour|second')

case "$WAIT_UNIT" in
  minute*) WAIT_SECS=$((WAIT_NUM * 60)) ;;
  hour*)   WAIT_SECS=$((WAIT_NUM * 3600)) ;;
  second*) WAIT_SECS=$WAIT_NUM ;;
  *)       WAIT_SECS=600 ;;  # default 10 min if unparseable
esac

ELAPSED=$(( $(date +%s) - START_TIME ))
REMAINING=$(( TIMEOUT_SECONDS - ELAPSED ))
if [ "$REMAINING" -le 0 ]; then
  echo "Timeout budget exhausted during rate-limit wait. Reporting current state."
  TIMED_OUT=true
else
  CAPPED_WAIT=$(( WAIT_SECS < REMAINING ? WAIT_SECS : REMAINING ))
  echo "CodeRabbit rate limited. Sleeping ${CAPPED_WAIT}s (budget: ${REMAINING}s remaining)..."
  sleep "$CAPPED_WAIT"
fi
```

After sleeping, only trigger a new review if the timeout budget has not been exhausted:

```bash
if [ "${TIMED_OUT:-false}" != "true" ]; then
  gh pr comment "$PR_NUMBER" --body "@coderabbitai review"
  ELAPSED=$(( $(date +%s) - START_TIME ))
  REMAINING=$(( TIMEOUT_SECONDS - ELAPSED ))
  REVIEW_WAIT=$(( 60 < REMAINING ? 60 : REMAINING ))
  if [ "$REVIEW_WAIT" -gt 0 ]; then
    echo "Waiting ${REVIEW_WAIT}s for review to begin..."
    sleep "$REVIEW_WAIT"
  fi
else
  echo "Skipping review-start wait (timeout budget exhausted)."
fi
```

Re-check status from Step 2.

### 2c. Currently processing

If `$CR_COMMENT` contains "Currently processing new changes in this PR":

```bash
echo "CodeRabbit is still reviewing. Waiting 60s..."
sleep 60
```

Re-fetch and re-check Step 2. **Do not proceed to Step 3 until processing is
complete.** Before each wait cycle, check whether the total elapsed time has
reached `TIMEOUT_SECONDS`. If it has, stop waiting and proceed to Step 7.

### 2d. Ready

If none of the above, CodeRabbit is idle and ready. Proceed to Step 3.

---

## Step 4: Triage protocol

For every thread where `isResolved: false`:

### 4a. Outdated threads

If `isOutdated: true`:
1. Mark resolved via GraphQL:

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: { threadId: $threadId }) {
    thread { id isResolved }
  }
}' -F threadId="<THREAD_ID>"
```

2. Post reply: `"Outdated thread — resolved."`

### 4b. Actionable code fixes

If the comment describes a genuine issue (bug, type error, missing guard, wrong
logic, incorrect documentation):
- Read the referenced file and line
- Implement the fix
- Note it for the commit message

### 4c. Disagreements / false positives

If the issue is not real (misunderstanding, style preference, inapplicable
suggestion):
- Post an inline reply explaining why it's not an issue
- Do NOT mark it resolved — wait for CodeRabbit to respond

### 4d. Resolving disagreement threads after CodeRabbit replies

During the poll loop (Step 7), re-fetch threads and check for new CodeRabbit
replies on disagreement threads you replied to:
- CodeRabbit **agrees** → mark resolved
- CodeRabbit **pushes back** with a valid point → re-evaluate: fix (4b) or
  reply with stronger justification
