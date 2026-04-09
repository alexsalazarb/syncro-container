---
name: list-skills
description: >-
  Lists all available skills with descriptions and trigger conditions. Use
  when user asks "what skills are available", "what automation do we have",
  or "list skills". Reads skill definitions from .claude/skills/ and presents
  a formatted summary.
---

# List Skills

## Context Required
META: no project context needed

## Triggers

### Automatic
- None (manual only)

### Manual
- `/list-skills`
- "run list-skills"
- "what skills are available?"
- "show me the skills"
- "what automation do we have?"

## Instructions

### Step 1: Locate skill files
Read SKILL.md files from `.ai-framework/.claude/skills/` directories.

### Step 2: Extract metadata
- Skill name (from frontmatter `name` field)
- Description (from frontmatter `description` field)
- Trigger type (Automatic vs Manual from Triggers section)

### Step 3: Format and present

```text
Available Skills for {Project Name}:
=====================================

AUTOMATIC (fire without being asked):
--------------------------------------

pre-commit-check
  Validates Swift files before any git commit: force unwraps, thread safety, hardcoded values
  Triggers: Any "commit", "stage", "git add", "create a PR"

log-mistake
  Logs correction patterns to prevent repeated errors
  Triggers: User provides correction (auto-detected)

check-kb-index
  Updates knowledge base README.md index
  Triggers: After any KB file created/modified/deleted

session-end-checklist
  Safety net checklist before ending session
  Triggers: User says "done", "thanks", "bye"

MANUAL (invoke with /skill-name):
-----------------------------------

document-solution     /document-solution
  Creates KB doc from complex problem solution
  Use: KB miss → solved, 3+ files, 5+ exchanges

save-session          /save-session
  Creates handoff doc for session continuity
  Use: Long session (20+ turns), switching tasks

create-plan           /create-plan [description]
  Creates a git-native development plan with tasks, phases, and dependencies
  Use: Starting a new feature or multi-step work

execute-task          /execute-task {plan-slug}/{task-path}
  Executes a single task from a development plan
  Use: Working on a specific plan task

execute-plan          /execute-plan {plan-slug}
  Executes all remaining plan tasks with parallel agent support
  Use: Run everything in a plan

check-test-coverage   /check-test-coverage
  Maps source files to expected test files, identifies coverage gaps
  Use: After implementing a feature or before creating a PR

archive-plan          /archive-plan {plan-slug}
  Archives a completed plan and updates the index
  Use: After all tasks are merged

list-skills           /list-skills
  Lists all available skills (this skill)
  Use: Discover automation
```

## Output

- Formatted list of all available skills
- Each entry includes name, description, and trigger summary
