
# Save Session

## Context Required
FULL-CONTEXT: AGENTS.md + relevant KB

## Triggers

### Automatic
- Conversation exceeds ~20 turns
- Context usage exceeds 70%
- User says "let's pause here" or "save this for later"
- Before switching to unrelated major task

### Manual
- `/save-session`
- "run save-session"
- "save session" or "save progress"
- "create handoff doc"
- "let's pick this up later"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `CONTAINER_KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

If config is absent, use default: `CONTAINER_KB_DIR=docs/kb-container`.

---

### Step 1: Generate filename
- Format: `YYYY-MM-DD-topic-slug.md`
- Topic slug: 2-4 words describing main task, kebab-case
- Example: `2026-03-15-realm-migration-ios18.md`

### Step 2: Gather session context
- What task(s) were we working on?
- What decisions were made and why?
- What files were created or modified?
- What's still unfinished?
- Any blockers or open questions?
- Which country targets were affected?

### Step 3: Create handoff document

Save to `{CONTAINER_KB_DIR}/ai-patterns/sessions/[filename]` using this structure:

```markdown
# Session: [Topic Description]

**Date**: YYYY-MM-DD
**Duration**: ~X turns
**Status**: [In Progress / Paused / Completed]
**Country Targets Affected**: [e.g., ArgNewspaper, EspNewspaper, or "All"]

---

## Task Summary

[1-2 paragraphs describing what we were doing and why]

## Key Decisions

- **[Decision 1]**: [Rationale]
- **[Decision 2]**: [Rationale]

## Files Modified

| File | Change |
|------|--------|
| path/to/file | Description of change |

## Files Created

- `path/to/new_file` - [Purpose]

## Current State

[Where things stand right now]

## Open Questions / Blockers

- [ ] Question 1
- [ ] Blocker 2

## Next Steps

1. [Specific action]
2. [Specific action]

## Useful Commands

[xcodebuild / swiftlint / pod commands that were useful]

## Related Resources

- [Link to relevant KB doc]
```

### Step 4: Update sessions index
- Read `{CONTAINER_KB_DIR}/ai-patterns/sessions/README.md` (or create if missing)
- Add entry to "Active Sessions" table
- Save updated README

### Step 5: Run check-kb-index
Update the KB index to include the new session file.

### Step 6: Confirm to user
```text
Session saved: {CONTAINER_KB_DIR}/ai-patterns/sessions/YYYY-MM-DD-topic-slug.md

To continue this work:
1. Start new conversation
2. Say: "Continue session from YYYY-MM-DD-topic-slug.md"
3. Agent will load context and resume
```

## Output

- New file: `{CONTAINER_KB_DIR}/ai-patterns/sessions/YYYY-MM-DD-topic-slug.md`
- Updated: `{CONTAINER_KB_DIR}/ai-patterns/sessions/README.md` (if exists)
- Confirmation message with continuation instructions
