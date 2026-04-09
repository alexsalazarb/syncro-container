
# Cleanup Sessions

## Context Required
META: no project context needed

## Triggers

### Automatic
- None (manual only)

### Manual
- `/cleanup-sessions`
- "run cleanup-sessions"
- "clean up old sessions"
- "delete old session files"

### Suggested
- When `{CONTAINER_KB_DIR}/ai-patterns/sessions/` has 10+ files
- During periodic maintenance
- When KB health check flags stale sessions

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `CONTAINER_KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

If config is absent, use default: `CONTAINER_KB_DIR=docs/kb-container`.

---

### Step 1: List session files
```bash
find {CONTAINER_KB_DIR}/ai-patterns/sessions -maxdepth 1 -type f -name '*.md' ! -name 'README.md' | sort
```

### Step 2: Identify old sessions (older than `--days`, default 30)
- Session files use format: `YYYY-MM-DD-topic-slug.md`
- Parse date from filename
- Calculate age from today's date and compare it to the configured threshold

### Step 3: Scan for unlogged patterns (before deletion)
- Quick scan each old session for:
  - Corrections mentioned but not in mistake-log
  - Solutions found but not documented
- If `--force` is set, skip this scan and proceed directly to Step 4
- If found, ask user: "Session X has undocumented patterns. Log them first?"

### Step 4: Delete old sessions
Show the sessions selected for deletion and ask for confirmation.
Delete sessions older than the threshold only after explicit approval.
If `--dry-run` is set, stop after reporting the candidate deletions and do not delete files or update the index, even when `--force` is also set.
Otherwise, if `--force` is set, skip confirmation and delete immediately.

```bash
for session in $old_sessions; do
  echo "DELETE: $session"
done

if [ "${DRY_RUN:-false}" = "true" ]; then
  exit 0
fi

if [ "${FORCE:-false}" != "true" ]; then
  printf "Delete these sessions? (y/N): "
  read -r confirm
  [ "$confirm" = "y" ] || [ "$confirm" = "Y" ] || exit 0
fi

deleted_sessions=""
for session in $old_sessions; do
  if rm "$session"; then
    deleted_sessions="${deleted_sessions}${deleted_sessions:+ }$session"
  else
    echo "Failed to delete $session — skipping" >&2
  fi
done
```

### Step 5: Update sessions index
Steps 4 and 5 must run in the same shell session so that `deleted_sessions` remains in scope.
Only run this step after real deletions (never on `--dry-run`).
- Verify `{CONTAINER_KB_DIR}/ai-patterns/sessions/README.md` exists and is readable
- Build a pattern of all deleted session basenames, then rewrite README in a single atomic pass to remove all matching lines
- If the rewrite fails, the original README is left untouched

```bash
readme="{CONTAINER_KB_DIR}/ai-patterns/sessions/README.md"
[ -r "$readme" ] || { echo "Missing or unreadable $readme" >&2; exit 1; }

# Build a pipe-separated ERE pattern of all deleted basenames (special chars escaped)
pattern=""
for session in $deleted_sessions; do
  base=$(basename "$session" | sed 's/[][(){}.^$*+?|\\]/\\&/g')
  pattern="${pattern}${pattern:+|}${base}"
done

# Single-pass rewrite: remove lines referencing any deleted session
tmp="${readme}.tmp"
if awk -v pat="$pattern" 'BEGIN{split(pat,a,"|")} { ok=1; for(i in a) if(index($0,a[i])) ok=0; if(ok) print }' "$readme" > "$tmp"; then
  mv "$tmp" "$readme"
else
  rm -f "$tmp"
  echo "Failed updating $readme" >&2
  exit 1
fi
```

### Step 6: Report results
```text
Session cleanup complete:
- Scanned: X sessions
- Deleted: Y sessions (older than N days)
- Kept: Z sessions (recent)

Oldest remaining session: YYYY-MM-DD
```

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `--days N` | 30 | Delete sessions older than N days |
| `--dry-run` | false | Preview deletions without executing |
| `--force` | false | Skip pattern scan and confirmation, then delete immediately unless `--dry-run` is also set |

## Output

- Deleted session files
- Updated: `{CONTAINER_KB_DIR}/ai-patterns/sessions/README.md` (only when deletions actually run)
- Console report of cleanup results
- Warning if undocumented patterns found

## Examples

User: "clean up old sessions"

```text
Scanning {CONTAINER_KB_DIR}/ai-patterns/sessions/...

Found 5 session files:
- YYYY-MM-DD-servicetrade-sync.md (38 days old) → DELETE
- YYYY-MM-DD-report-refactor.md (33 days old) → DELETE
- YYYY-MM-DD-api-endpoints.md (13 days old) → KEEP
- YYYY-MM-DD-component-model.md (5 days old) → KEEP
- YYYY-MM-DD-ai-framework.md (today) → KEEP

Session cleanup complete:
- Scanned: 5 sessions
- Deleted: 2 sessions
- Kept: 3 sessions
```
