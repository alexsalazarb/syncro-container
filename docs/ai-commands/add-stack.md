
# Add Stack

## Context Required
META: Framework repo structure only — no consumer project context needed.

## Triggers

### Automatic
None (manual only).

### Manual
- `/add-stack`
- `/add-stack [stack-name]`
- "add support for [technology]"
- "add [technology] stack to the framework"
- "we need [technology] support"

## Instructions

### Step 0: Confirm framework-repo mode

This skill MUST run inside the framework repo itself (not a consumer project).

Check: `knowledge-base/` exists at repo root AND `.ai-framework.config` does NOT exist.
If not in framework-repo mode, abort with:
> "This skill runs inside the ai-framework repo. cd into it first."

### Step 1: Gather stack requirements

Collect from the user (or infer from the technology name):

| Field | Required | Example |
|-------|----------|---------|
| **Stack slug** | Yes | `angular`, `vue`, `go`, `kotlin-multiplatform` |
| **Display name** | Yes | `Angular`, `Vue.js`, `Go`, `Kotlin Multiplatform` |
| **Primary language** | Yes | `TypeScript`, `JavaScript`, `Go`, `Kotlin` |
| **Detection file** | Yes | `angular.json`, `vue.config.js`, `go.mod` |
| **CLI tool** (if any) | No | `ng`, `vue-cli`, `go` |
| **Test framework** | No | `Jasmine/Karma`, `Jest/Vitest`, `go test` |
| **Common libraries** | No | List of 3-5 key ecosystem libraries |

If the user just provides a name (e.g. "add Vue support"), infer reasonable defaults
and confirm before proceeding.

### Step 2: Validate uniqueness

1. Check `knowledge-base/` subdirectories — stack slug must not already exist.
2. Check `templates/AGENTS.*.md` — template must not already exist.
3. Check `scripts/init-repo.sh` `detect_stack()` — must not already detect this stack.

If the stack already exists, offer to update it instead of creating from scratch.

### Step 3: Research the stack

Use **WebSearch** to gather current best practices for the technology:

→ See [research-areas.md](references/research-areas.md) for required research topics
  and search query patterns.

Combine web research with knowledge of existing framework templates (read 1-2 existing
`templates/AGENTS.*.md` files as structural references).

### Step 4: Create AGENTS.md template

Write `templates/AGENTS.{slug}.md` following the framework's template structure.

→ See [agents-template-guide.md](references/agents-template-guide.md) for section-by-section
  guidance on what to customize per stack.

Use an existing template as the structural base (copy section order, triggers, skills table)
and replace all stack-specific content: Project Context table, Critical Constraints,
Patterns section, Naming Conventions, Things to Avoid, Quick Reference commands.

### Step 5: Create knowledge-base structure

Create the KB directory and seed files:

```
knowledge-base/{slug}/
├── _index.md
├── README.md
├── best-practices/
│   └── {slug}-standards.md
├── architecture/
├── ui-patterns/
├── integrations/
└── ai-patterns/
```

**`_index.md`**: Follow the format in
[`add-kb-doc/references/new-stack-scaffold.md`](../add-kb-doc/references/new-stack-scaffold.md).

**`README.md`**: Follow the format of an existing stack README (e.g. `knowledge-base/flutter/README.md`).

**`{slug}-standards.md`**: Write universal quality standards for the stack.
→ See [standards-template-guide.md](references/standards-template-guide.md) for the
  required sections and how to adapt per language/framework.

### Step 6: Update init-repo.sh

Edit `scripts/init-repo.sh` in two places:

**6a. Help text** (line ~9): Add the slug to the `--stack` option list.

**6b. `detect_stack()` function**: Add an `elif` clause that detects the stack's
signature file. Place it in the correct priority order:
- Stack-specific config files (e.g. `angular.json`, `go.mod`) go BEFORE the
  generic `package.json` check.
- Language-specific files (e.g. `Podfile`, `build.gradle`) keep their existing position.

Example:
```bash
elif [ -f "$root/{detection-file}" ]; then
    echo "{slug}"
```

### Step 7: Offer KB population

Ask the user:
> "Stack scaffolded. Want to research and populate KB docs for {stack}'s common
> libraries? This will use `/research-kb-topic --scan` logic to add integration
> docs for libraries like {examples}."

If yes, delegate to `/research-kb-topic` in single mode for each confirmed library.
If no, skip — the KB will grow organically as projects use `add-kb-doc`.

### Step 8: Report

```text
Stack Added: {slug}
=====================================

Created:
  - templates/AGENTS.{slug}.md
  - knowledge-base/{slug}/_index.md
  - knowledge-base/{slug}/README.md
  - knowledge-base/{slug}/best-practices/{slug}-standards.md

Updated:
  - scripts/init-repo.sh (detection + help text)

Detection: projects with `{detection-file}` will auto-detect as `{slug}`
Usage:     .ai-framework/scripts/init-repo.sh --stack {slug}

Next steps:
  1. Review the generated template and standards for accuracy
  2. Run /research-kb-topic to populate integration docs (optional)
  3. Commit: git add -A && git commit -m "feat({slug}): add {display-name} stack support"
```

## Output

- `templates/AGENTS.{slug}.md` — Stack-specific AGENTS template
- `knowledge-base/{slug}/` — KB directory with _index.md, README.md, standards doc
- `scripts/init-repo.sh` — Updated detection logic and help text
- Report with usage instructions and next steps
