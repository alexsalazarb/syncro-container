
# Session End Checklist

## Context Required
LOW-CONTEXT: AGENTS.md only

## Triggers

### Automatic (MUST detect and run)

Trigger when user says ANY of:
- "done", "done for now", "that's all", "thanks"
- "ending session", "wrapping up", "stopping here"
- "let's commit", "commit everything", "push this"
- "bye", "talk later", "end of day"
- Any indication session is ending

### Manual
- `/session-end-checklist`
- "run session-end-checklist"
- "end session"
- "wrap up"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `CONTAINER_KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

If config is absent, use default: `CONTAINER_KB_DIR=docs/kb-container`.

---

Run through this checklist and take action on any "Yes":

### Step 1: Complex Problem Check
```text
Did we solve something that required:
- Reading 3+ files?
- 5+ back-and-forth exchanges?
- A non-obvious Swift/iOS/RxSwift pattern?
- KB lookup that found nothing?

→ If YES: Run `document-solution`
```

### Step 2: Correction Check
```text
Did the user correct me at any point?
- Said "that's wrong", "actually...", "no..."?
- Pointed out a Swift convention I missed?
- Corrected a RxSwift pattern or iOS compatibility issue?

→ If YES: Run `log-mistake` (skip only exact duplicates from same session/incident)
```

### Step 3: KB Modification Check
```text
Did I create or modify any file in {CONTAINER_KB_DIR}/ (or {KB_DIR}/)?

→ If YES: Run `check-kb-index`
```

### Step 4: Uncommitted Changes Check
```text
Are there uncommitted changes that should be committed?

→ If YES: Run `pre-commit-check`, then offer to commit
```

### Step 5: Session Length Check
```text
Was this a long session (20+ turns)?

→ If YES: Run `save-session`
```

## Output

Report to user:

```text
Session End Checklist
=====================

[x] Complex problem documented: [filename or "None needed"]
[x] Corrections logged: [count or "None"]
[x] KB index updated: [Yes/No/Not needed]
[x] Pre-commit check: [Passed/Issues found/No changes]
[x] Session saved: [filename or "Not needed"]

Session complete. Ready for next session.
```
