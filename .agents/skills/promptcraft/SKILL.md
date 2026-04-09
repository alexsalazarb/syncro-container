---
name: promptcraft
description: Generate an optimized, structured prompt for a development task. Creates LLM-optimized prompts using primacy/recency effects, concrete examples, and mandatory language. Supports 8 task types. Use this when you want to create a high-quality prompt for a new session or to hand off to a teammate.
argument-hint: "<task-type> <description>"
---

# PromptCraft — AI Prompt Generator

## Context Required
META: no project context needed — generates prompts from user input alone.

## Triggers

### Automatic
None

### Manual
- `/promptcraft`
- `/promptcraft [type] [description]`
- "generate a prompt for..."
- "create an optimized prompt"

## Instructions

### Step 1: Parse Arguments

You are generating an optimized prompt for a development task. The user invoked `/promptcraft $ARGUMENTS`.

### Step 2: Identify Task Type

If the user did not specify a type, ask them to pick one:

1. **feature-research** — Investigating existing code, understanding patterns, or researching how something works
2. **bug-fix** — Hunting down and fixing a bug
3. **jira-docs** — Writing JIRA tickets, documentation, or technical specs
4. **sql** — Generating or optimizing SQL queries
5. **feature-plan** — Large-scale feature planning and architecture (supports `.plans/` formal plan output)
6. **test-writing** — Writing tests/specs for existing or new code
7. **general-dev** — General development/implementation tasks
8. **code-review** — Reviewing code changes for quality, bugs, and conventions

### Step 3: Apply Prompt Generation Rules (LLM Optimization Research)

Apply these principles from December 2025 LLM comprehension research:

1. **Primacy Effect**: Put the MOST CRITICAL instructions at the TOP (constraints, "do NOT" rules, required behaviors)
2. **Recency Effect**: Put REMEMBER/self-check section at the BOTTOM
3. **Lost in the Middle**: Keep total prompt under 200 lines. Use tables and bullet points, not paragraphs
4. **Concrete Examples**: Include explicit examples of what to do and what NOT to do
5. **Mandatory Language**: Use "MUST", "Do NOT", "ALWAYS", "NEVER" — not "should", "consider", "try to"
6. **Explore First**: Include an exploration preamble for implementation tasks (reduces rework ~70%)
7. **Web Research Trigger**: Inject web research instruction when external libraries are involved
8. **Plan Mode**: Suggest `/create-plan` for tasks spanning 3+ files

### Step 4: Generate Using Prompt Template Structure

Generate the prompt in this structure:

```markdown
# [Task Type]: [Brief Title]

## CRITICAL REQUIREMENTS
- [Constraint 1 — use MUST/NEVER language]
- [Constraint 2]
- Check `{KB_DIR}/README.md` for relevant KB docs BEFORE exploring code

## EXPLORE FIRST — THEN IMPLEMENT
Before writing any code or proposing solutions:
1. **Read the relevant code** — Use Read/Glob/Grep to understand the current implementation.
2. **Identify patterns** — Note how similar things are done in the codebase.
3. **Come back with questions** — If anything is unclear, ask BEFORE implementing.

## COMMON MISTAKES TO AVOID
- [Anti-pattern 1 with concrete example]
- [Anti-pattern 2 with concrete example]

## WEB RESEARCH REQUIRED (only when external libraries involved)
This task involves external libraries that may post-date your training cutoff: **[libraries]**
Before implementing, use WebSearch/WebFetch to verify current API surface.

## Context
- **JIRA**: [ticket if provided]
- **Stack**: [detected or provided from .ai-framework.config]
- **Files**: [relevant files if known]
- **Reference Patterns**: [similar working code to follow]

## Task Description
[User's description, structured and clear]

## Requirements
[Numbered list of specific requirements]

## Acceptance Criteria
[Checkboxes for what "done" looks like]

## PLAN MODE RECOMMENDED (only for 3+ file tasks)
This task likely spans 3+ files. Consider running `/create-plan` before implementing.

## Implementation Approach
1. **Understand**: Read existing code and reference implementations
2. **Plan**: Identify all files/methods needed
3. **Implement**: Follow project patterns
4. **Test**: Write comprehensive tests
5. **Verify**: Confirm requirements met

## BEFORE YOU RESPOND
- Read `{AGENTS_MD}` before starting
- Check KB for existing documentation on this topic
- Run pre-commit-check before committing
- Complex task? Consider `/create-plan` for formal planning

## Deliverables
[numbered deliverable list]

_Recommended model: **claude-opus** or **claude-sonnet** depending on task complexity_
```

### Step 5: Feature-Plan Type — Plans Integration

When generating a `feature-plan` prompt, ask:

> "Generate as formal development plan (.plans/ directory)?"

If yes, modify the Task section to instruct the agent to:
1. Run `/create-plan` to produce `{PLANS_DIR}/<slug>/overview.md` and task files
2. Provide a technical analysis and task breakdown as input to `/create-plan`

Update Deliverables to reference plan artifacts:
- `{PLANS_DIR}/<slug>/overview.md` — plan overview with phases and task summary
- `{PLANS_DIR}/<slug>/[phase-N/]task-XX-{slug}/task.md` — individual task files
- `{PLANS_DIR}/<slug>/[phase-N/]task-XX-{slug}/status.md` — per-task status tracking

### Step 6: Deliver the Prompt

Once the prompt is generated, ask the user:

1. **Use in current session** — Apply this prompt to the current conversation and start working
2. **Copy to clipboard** — Output the raw markdown for the user to paste elsewhere
3. **Launch new Claude session** — Save to a temp file and launch Claude from it

If the user chooses option 3:
```bash
tmpfile="$(mktemp /tmp/promptcraft-output.XXXXXX.md)"
# write prompt to "$tmpfile"
prompt="$(cat "$tmpfile")"
claude -p "$prompt"
rm -f "$tmpfile"
```

## Output

- Optimized, structured prompt in markdown format
- Prompt follows LLM optimization principles (primacy/recency, mandatory language, concrete examples)
- Delivery via one of: applied to current session, copied to clipboard, or launched in new Claude session
