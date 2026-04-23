
# Log Mistake

## Context Required
LOW-CONTEXT: AGENTS.md only

## Triggers

### Automatic (MUST detect proactively - do NOT wait for explicit request)

**Detection Phrases** - If user says ANY of these, trigger immediately:
- "that's wrong", "that's not right", "that's incorrect"
- "no, it should be...", "actually...", "not quite..."
- "you forgot to...", "you missed...", "you need to..."
- "we already discussed...", "I told you earlier...", "like I said..."
- "that won't work because...", "the problem with that is..."
- User provides a fix or correction to your output
- User points out a convention/pattern you violated
- User explains why your approach is wrong

**Do NOT wait for**: "log this mistake", "remember this", or explicit logging requests.
**Do**: Detect correction patterns and log proactively.

### Manual
- `/log-mistake`
- "run log-mistake"
- "log this correction"
- "remember this for next time"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `CONTAINER_KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

If config is absent, use default: `CONTAINER_KB_DIR=docs/kb-container`.

---

### Step 1: Identify the mistake pattern
- What did the agent do wrong?
- What type of error? (convention, rx-pattern, ios-compat, architecture, performance, etc.)
- What files/code were affected?

### Step 2: Identify the correction
- What should have been done instead?
- Why is the correction right?
- What rule or pattern was violated?

### Step 3: Categorize the mistake

Use one of these generic categories, or add a stack-specific category if AGENTS.md defines one:

- `architecture` — Violated the project's layering/pattern rules (wrong layer, wrong ownership)
- `convention` — Violated AGENTS.md convention (wrong file location, naming, style)
- `thread-safety` — Mutation on wrong thread or async context
- `performance` — Unnecessary work, inefficient queries, redundant reloads
- `api` — Wrong endpoint, missing auth, bad response handling
- `testing` — Missing test, wrong assertion, wrong test target
- `config` — Hardcoded value that should come from a configuration/environment constant
- `dependency` — Wrong import, missing dependency, circular reference

Stack-specific categories to add in AGENTS.md as needed:
- Swift: `rx-pattern`, `ios-compat`, `realm`, `force-unwrap`
- Rails: `activerecord`, `n-plus-one`, `migration`
- React: `hook-rules`, `prop-types`, `re-render`
- Python: `type-annotation`, `async-pattern`

> **Convention vs. mistake?** If the correction reveals a project-specific rule the agent couldn't have known (e.g. "this project uses pnpm, not npm"), that rule also belongs in `conventions.md` via `/capture-convention`. Log the mistake here AND capture the convention — they serve different purposes.

### Step 4: Check for duplicates before appending

- Scan the existing mistake log for an entry with the same category and similar mistake/correction
- If a matching entry already exists, append a `**Recurrence:** {YYYY-MM-DD}` line to the existing entry
- If new, create a fresh entry

### Step 5: Append to mistake log

File: `{CONTAINER_KB_DIR}/ai-patterns/mistake-log.md`

```markdown
## [YYYY-MM-DD] - [Category]

**Mistake:** [What was done wrong]

**Correction:** [What should have been done]

**Prevention:** [How to catch this next time]

**Files involved:** [List of files]

---
```

### Step 5b: Update KB index
- Run `check-kb-index` after modifying the mistake log

### Step 6: Check for repeated patterns
- If 3+ occurrences of same category/pattern:
  ```text
  ⚠️ Pattern Alert: This is the 3rd [category] mistake logged.

  Consider adding to AGENTS.md "Things to Avoid" section.

  Would you like me to propose an update?
  ```

### Step 7: Framework contribution check (automatic)

After logging, evaluate whether the mistake and its correction reveal a **universal pattern** — a rule that any developer on this stack could benefit from knowing.

Ask yourself:
- Could this mistake happen on any {STACK} project, not just this one?
- Is the correction a general rule (not tied to project-specific classes or business logic)?
- Is this correction already in `.ai-framework/knowledge-base/{STACK}/best-practices/`?

**If the correction is a universal rule**, proactively tell the user:

```text
💡 This correction looks like a universal {STACK} standard.
   Consider contributing it to the framework: /contribute-pattern
```

**If the mistake is project-specific** (relies on project structure or custom code), skip silently.

### Step 8: Confirm to user
```text
Mistake logged: {CONTAINER_KB_DIR}/ai-patterns/mistake-log.md

Category: [category]
[If new entry]: Appended new entry.
[If duplicate]: Updated existing entry — appended 'Recurrence: {YYYY-MM-DD}'.
This will help prevent similar errors in future sessions.
```

## Output

- Created or updated entry in `{CONTAINER_KB_DIR}/ai-patterns/mistake-log.md`
- Alert if pattern is recurring
- Confirmation message
