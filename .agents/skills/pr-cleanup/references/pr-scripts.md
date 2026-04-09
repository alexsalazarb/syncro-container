# PR Cleanup — Scripts Reference

Bash scripts and templates used by [pr-cleanup](../SKILL.md).

---

## Step 1: Find the PR

```bash
PR_NUMBER=$(gh pr list --head "$(git branch --show-current)" --json number --jq '.[0].number')
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
OWNER=$(echo "$REPO" | cut -d/ -f1)
REPO_NAME=$(echo "$REPO" | cut -d/ -f2)
echo "PR #$PR_NUMBER in $REPO"
```

Record start time for timeout tracking:

```bash
START_TIME=$(date +%s)
TIMEOUT_SECONDS=$((${timeout_minutes:-30} * 60))
```

---

## Step 3: Fetch review threads

Write the query to a temp file to avoid Unicode/quoting issues:

```bash
cat <<'GRAPHQL' > /tmp/pr_cleanup_threads.graphql
query($owner: String!, $repo: String!, $pr: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      reviewThreads(first: 500) {
        nodes {
          id
          isResolved
          isOutdated
          comments(first: 5) {
            nodes {
              author { login }
              body
              path
              line
              originalLine
            }
          }
        }
      }
    }
  }
}
GRAPHQL

gh api graphql \
  -F owner="$OWNER" \
  -F repo="$REPO_NAME" \
  -F pr="$PR_NUMBER" \
  -f query="$(cat /tmp/pr_cleanup_threads.graphql)"
```

Also fetch PR-level review bodies:

```bash
gh pr view "$PR_NUMBER" --json reviews,comments
```

---

## Step 5: Run local tests

```bash
TESTS_RAN=false

# Shell/bash repos (promptcraft, framework scripts)
if [ -d "promptcraft/tests" ]; then
  ./promptcraft/tests/test_all_tasks.sh || exit 1
  if [ -f "promptcraft/tests/test_utils_functions.sh" ]; then
    ./promptcraft/tests/test_utils_functions.sh || exit 1
  fi
  TESTS_RAN=true
fi

# Ruby / Rails
if [ -f "Gemfile" ]; then
  bundle exec rspec --format progress || exit 1
  TESTS_RAN=true
fi

# Node / TypeScript
if [ -f "package.json" ] && grep -q '"test"' package.json; then
  npm test -- --passWithNoTests 2>/dev/null || exit 1
  TESTS_RAN=true
fi

# Python
if [ -f "pyproject.toml" ] || [ -f "setup.py" ] || [ -f "pytest.ini" ]; then
  python3 -m pytest -q || exit 1
  TESTS_RAN=true
fi

if [ "$TESTS_RAN" = false ]; then
  echo "No test suite detected — skipping local tests."
fi
```

---

## Step 6: Commit and push

```bash
# Stage specific changed files — NEVER use git add -A or git add .
git add <specific files>

git commit -m "fix: address CodeRabbit review feedback on PR #$PR_NUMBER

$(git diff --cached --stat)"

git push
```

---

## Step 8: Post final PR comment

```bash
gh pr comment "$PR_NUMBER" --body "$(cat <<'EOF'
## PR Cleanup Summary

| Category | Count |
|----------|-------|
| Code fixes applied | N |
| Outdated threads resolved | N |
| Disagreement threads resolved (CodeRabbit agreed) | N |
| Threads replied (disagreement, awaiting response) | N |
| CI checks passing | ✅ / ❌ |

### Fixes applied
- fix 1 description

### Threads resolved as outdated
- thread description

### Disagreement threads resolved (CodeRabbit agreed)
- thread description

### Threads replied (disagreement, awaiting CodeRabbit response)
- thread description + reason
EOF
)"
```

---

## Example Invocation

```text
User: /pr-cleanup
```

```text
Agent:
PR #21 found in InspectPoint/ai-agent-framework

Checking CodeRabbit status (sort by updatedAt)...
→ Idle. Proceeding.

Fetching review threads...
4 unresolved threads found

Thread 1 (feature_planning.sh:112): Actionable — fixing .plans/ directory structure docs
Thread 2 (README.md:208): Actionable — fixing /plan → /create-plan
Thread 3 (test_utils_functions.sh:6): Actionable — guarding cd with || exit 1
Thread 4 (SKILL.md:119): Actionable — same .plans/ structure fix

Running tests... ✓ 10/10 + 32/32 passed
Running pre-commit-check... ✓ Clean
Committing: "fix: address CodeRabbit review feedback on PR #21"
Pushing...

Poll 1 — Elapsed: 0:01 / 30:00
  CodeRabbit status: Currently processing new changes in this PR
  → Waiting for review to complete...
  Sleeping 60s

Poll 2 — Elapsed: 0:02 / 30:00
  CodeRabbit status: Review complete
  Unresolved threads: 0
  CI: Validate Promptcraft Scripts ✅, apply-label ✅, Mend ✅
  → Done condition met.

Cleanup complete — 3 fixes applied, 0 threads outdated, 4 threads resolved, CI: passing
```
