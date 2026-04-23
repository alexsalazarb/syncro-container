---
name: check-agent-drift
description: >-
  Verifies that AGENTS.md accurately reflects the current codebase state.
  Use when user says "check for drift", "verify documentation", or during
  periodic maintenance. Identifies stale paths, missing patterns, outdated
  commands, and undocumented conventions. Suggests targeted fixes.
---

# Check Agent Drift

## Context Required
FULL-CONTEXT: AGENTS.md + relevant KB

## Triggers

### Automatic
- None (resource-intensive, run on request)

### Manual
- `/check-agent-drift`
- "run check-agent-drift"
- "verify agents.md"
- "is our documentation current?"
- "check for drift"
- Periodically during long projects

## Instructions

### Step 0: Resolve project context

Follow [`common/resolve-project-context.md`](../common/resolve-project-context.md):

Read `.ai-framework.config` → resolve:
- **`AGENTS_MD`** — `{PROJECT_CONTEXT_ROOT}AGENTS.md` (per-project conventions file)
- **`KB_DIR`** — per-project Layer 3 KB root
- **`CONTAINER_KB_DIR`** — container-level Layer 2 KB root

If config is absent, use defaults: `AGENTS_MD=AGENTS.md`, `CONTAINER_KB_DIR=docs/kb-container`.

---

### Step 1: Parse `{AGENTS_MD}` sections
- Commands and their expected behavior
- File paths and patterns
- Conventions and constraints
- Integration details

### Step 2: Verify Commands section
- For each documented slash command / skill:
  - Verify the referenced skill or doc file exists
  - Verify the command name matches the deployed skill/command mapping
  - Flag stale aliases, missing files, or renamed commands

### Step 3: Verify File Patterns section
- Check that listed paths exist
- Flag paths that don't exist or have moved

### Step 4: Verify Conventions against recent code
- Sample recent commits (last 20)
- Check if commits follow documented patterns
- Flag patterns that diverge from documentation

### Step 5: Scan for undocumented patterns
- Look for new directories not in `{AGENTS_MD}`
- Look for new conventions in recent PRs
- Check for new integrations or services

### Step 6: Check constraints accuracy
- Verify documented constraints are still valid
- Check for changed infrastructure or requirements

### Step 7: Check for stale AI tool shims

Some editors maintain their own config files that mirror AGENTS.md content. Check each of the following paths if they exist:

| Shim file | Tool |
|-----------|------|
| `.cursor/rules/agent.mdc` | Cursor |
| `.github/copilot-instructions.md` | GitHub Copilot |
| `.windsurfrules` | Windsurf |

For each file found:
1. Compare its last-modified timestamp against `{AGENTS_MD}` (via `git log -1 --format="%ci" -- <file>`)
2. Check whether it contains the same Skills/Commands section headings as `{AGENTS_MD}`
3. Flag as `[STALE_SHIM]` if `{AGENTS_MD}` is newer or key headings are missing/diverged

Add to the drift report under a `STALE SHIMS:` section:

```text
[STALE_SHIM] .cursor/rules/agent.mdc
  AGENTS.md last updated: 2026-04-15
  Shim last updated:      2026-03-10
  SUGGESTION: Regenerate via upgrade-framework (Step 5d re-runs shim generation)
```

---

### Step 8: Generate drift report

```text
{AGENTS_MD} Drift Check
========================

Checking against codebase state...

DRIFT DETECTED:

[STALE_PATH] Line 45
  Documented: app/workers/report_worker.rb
  Reality: File moved to app/workers/reports/generate_worker.rb
  SUGGESTION: Update path reference

[MISSING_PATTERN] Recent commits
  Pattern found: New convention in codebase
  Not documented in `{AGENTS_MD}`
  SUGGESTION: Add documentation

[OUTDATED_COMMAND] Line 23
  Documented: old command
  Reality: Command changed
  SUGGESTION: Update command

NO DRIFT:

✓ Section 1 - accurate
✓ Section 2 - confirmed

=====================
N drift issues found, M sections verified OK

Would you like me to apply suggested fixes?
```

### Step 9: Offer to apply fixes
- Show proposed `{AGENTS_MD}` changes
- Apply with user confirmation

## Output

- Detailed drift report to console
- Specific `{AGENTS_MD}` line references
- Actionable suggestions with proposed text
- Option to auto-apply fixes

## Examples

User: "Let's make sure our docs are up to date"

Agent runs the drift check and reports findings with specific line references and suggestions for each issue found.
