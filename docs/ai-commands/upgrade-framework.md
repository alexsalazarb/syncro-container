
# Upgrade Framework

## Context Required
HIGH-CONTEXT: AGENTS.md, current skill state, framework changelog

## Triggers

### Automatic
- Session start detects `.ai-framework` submodule is behind remote
- User says "upgrade the framework", "update ai-framework", "pull latest framework"

### Manual
- `/upgrade-framework`
- "upgrade framework"
- "sync framework"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve per [`common/resolve-project-context.md`](../common/resolve-project-context.md):
- **`AGENTS_MD`** — per-project `AGENTS.md` (`{PROJECT_CONTEXT_ROOT}AGENTS.md`; in single layout this is the repo-root `AGENTS.md`)
- **`CONTAINER_AGENTS_MD`** — root-level `AGENTS.md` (always `AGENTS.md` at repo root; same as `AGENTS_MD` in single layout)
- **`STACK`** — tech stack for the consumer repo

If config is absent, use defaults: `AGENTS_MD=AGENTS.md`, `CONTAINER_AGENTS_MD=AGENTS.md`.

---

### Step 1: Check Current State

1. Read `{AGENTS_MD}` to understand repo-specific context (project type, test commands, conventions)
2. Check the current framework submodule commit:
   ```bash
   cd .ai-framework && git log -1 --format="%h %s" && cd ..
   ```
3. Check if there's a newer version available:
   ```bash
   cd .ai-framework && git fetch origin main && git log HEAD..origin/main --oneline && cd ..
   ```
4. If already up to date, inform the user and stop.

### Step 2: Review What Changed

Before pulling, understand what's coming:

```bash
cd .ai-framework && git log HEAD..origin/main --format="%h %s" && cd ..
```

Read the commit messages. Categorize changes:
- **New skills** — skills that don't exist in the repo yet
- **Updated skills** — skills that changed in the framework
- **Updated templates** — AGENTS.md template changes (new skill table entries, new sections)
- **Updated scripts** — changes to init/upgrade/sync scripts
- **Breaking changes** — anything that changes skill interfaces or removes functionality

### Step 3: Pull the Update

```bash
cd .ai-framework && git checkout main && git pull origin main && cd ..
```

### Step 4: Run the Sync Script

```bash
.ai-framework/scripts/sync-skills.sh
```

This copies all framework skills to `.agents/skills/` and `.claude/skills/`, regenerates legacy commands, and reports any repo-specific skills.

### Step 5: Intelligent Review

This is the core value of the AI-assisted upgrade. Review each category:

#### 5a: New Skills

For each new skill that was just installed:
1. Read its `SKILL.md`
2. Check if `AGENTS.md` needs a new entry in the Skills table
3. If yes, add it (matching the existing table format)
4. Check if the skill has trigger conditions that should be in the Quick Triggers section

#### 5b: Updated Skills — Repo-Specific Carryover

This is the critical step. Some repos have context in `{AGENTS_MD}` that affects how skills behave:

1. Read `{AGENTS_MD}` for any repo-specific notes about skills (e.g., "our test command is `bundle exec rspec`", "our main branch is `develop`", "we use Shoryuken not Sidekiq")
2. For each updated skill, check if the skill references things that the repo overrides:
   - Branch names (some repos use `develop` instead of `main`)
   - Test commands
   - Deployment patterns
   - Multi-tenant scoping requirements
   - Jira project keys
3. Skills themselves are framework-owned and NOT modified — but if a skill's behavior depends on `{AGENTS_MD}` context (which it should via the `Context Required` header), verify that `{AGENTS_MD}` still provides what the skill needs
4. If a skill now requires context that `{AGENTS_MD}` doesn't provide, flag it: "Skill `{name}` now references `{field}` but `{AGENTS_MD}` doesn't include it. Add to `{AGENTS_MD}`?"

#### 5c: Repo-Specific Skills

For any skills reported as "repo-specific" by the sync script:
1. Read each one briefly
2. Check if it overlaps with or is superseded by a new framework skill
3. If overlap: inform the user — "Repo skill `{name}` overlaps with new framework skill `{framework-skill}`. Consider migrating."
4. If no overlap: leave it alone, it's intentional

#### 5d: `{AGENTS_MD}` Template Diff

Compare `{AGENTS_MD}` against the latest framework template for its stack:
1. Detect the stack (same logic as init-repo.sh)
2. Diff the template's structure against `{AGENTS_MD}`:
   - Missing sections that the template now has
   - New skill table entries
   - Updated trigger checklist items
3. Do NOT overwrite `{AGENTS_MD}` — suggest the specific additions needed
4. If changes are needed, make them (the skill table, trigger section, etc. are framework-owned sections within the repo-owned `{AGENTS_MD}`)

### Step 6: Sync CLAUDE.md

After any `{AGENTS_MD}` changes:
```bash
cp AGENTS.md CLAUDE.md
```

### Step 7: Validate

```bash
.ai-framework/scripts/validate-setup.sh
```

Fix any errors found. Warnings are informational.

### Step 8: Commit

Stage all changed files individually:
```bash
git add .ai-framework .agents/skills/ .claude/skills/ .claude/agents/ docs/ai-commands/ AGENTS.md CLAUDE.md
git commit -m "chore: upgrade AI agent framework to $(cd .ai-framework && git log -1 --format='%h')"
```

### Step 9: Report

Present to the user:
- Framework version: old → new (commit hashes)
- New skills installed (list with one-line descriptions)
- Updated skills (list what changed)
- `{AGENTS_MD}` changes made (new skill entries, new sections)
- Repo-specific skills found (and any overlap notes)
- Validation result
- Any manual action needed

## Output

- Updated `.ai-framework/` submodule
- Updated skills in `.agents/skills/` and `.claude/skills/`
- Updated agent definitions in `.claude/agents/`
- Updated `{AGENTS_MD}` and `CLAUDE.md` with new skill entries
- Regenerated legacy command files
- Commit with all changes

## Examples

### Routine upgrade

```
User: /upgrade-framework

Agent:
1. Checks .ai-framework — 3 commits behind origin/main
2. Reviews: 1 new skill (add-defect), 2 updated skills (create-bug-plan, execute-task)
3. Pulls latest, runs sync
4. Reads `{AGENTS_MD}` — Rails repo, main branch is "develop"
5. Checks updated skills — create-bug-plan references "main" as default base branch,
   but `{AGENTS_MD}` says main branch is "develop". Notes: "Skills use `{AGENTS_MD}` context
   for branch names — your 'develop' config is still correct."
6. Adds add-defect to `{AGENTS_MD}` skill table
7. Syncs CLAUDE.md, runs validate, commits
8. Reports: "Upgraded framework abc1234 → def5678. 1 new skill: add-defect.
   2 skills updated. `{AGENTS_MD}` skill table updated. All checks pass."
```
