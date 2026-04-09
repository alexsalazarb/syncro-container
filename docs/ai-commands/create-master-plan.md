
# Create Master Plan

## Context Required
HIGH-CONTEXT: Feature description, target repos, hub access

## Triggers

### Automatic
- None (manual only)

### Manual
- `/create-master-plan`
- `/create-master-plan [feature description]`
- "create a master plan for..."
- "new master plan"
- "plan across repos"

## Instructions

### Step 0: Load Config & Resolve Hub

Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md). Extract: `LAYOUT`, `PLANS_DIR`, `WORK_PLANS_PATH`, `WORK_PLANS_BRANCH`.

Resolve hub directory per [`common/hub-resolution.md`](../common/hub-resolution.md). Store as `HUB_ROOT` (`external` or `self-hosted`). On tier 4: abort with "No work-plans hub available. Set `WORK_PLANS_PATH` in `.ai-framework.config` or switch to a container layout."

Verify `HUB_ROOT` exists and contains a `plans/` directory. If `plans/` is missing, create the scaffold:
```
plans/
  INDEX.md          ← From templates/plans-INDEX.md
  backlog/
  planned/
  active/
  completed/
  deprioritized/
  cancelled/
```

**If external hub not found at resolved path**, guide setup:
1. Inform the user the hub path does not exist
2. Ask for the correct path or offer to create the directory
3. If the directory does not exist and the user provides a git remote, clone it
4. Set the git global config so it persists:
   ```bash
   git config --global ai-framework.workplanspath <path>
   ```
5. Continue — do not abort after successful setup

---

### Step 1: Gather Requirements

If not provided as arguments, ask the user for:
- **Title**: Clear, outcome-oriented name
- **Type**: Product | Technical | Bug/Support
- **Before/After**: What changes for the user/customer
- **Problem statement**: Business context (NOT technical details)
- **Target repos**: Which repos are involved, plus the canonical repo key for each (e.g. `api`, `ios`, `web`)
- **Owner**: Who owns this plan
- **Target demo date**: When this should be demonstrable
- **JIRA Epic**: Link or N/A
- **Initial state**: `backlog`, `planned` (default), or `active`
- **Kill criteria**: Specific, measurable conditions that would stop work
- **Milestones**: Key deliverables with target dates
- **Cross-repo contracts**: Shared interfaces, payloads, naming conventions, producers, and consumers
- **Quality / validation expectations**: Testing, QA, rollout, or acceptance evidence repo-level plans must include
- **Knowledge base / documentation expectations**: KB/docs to update, or areas likely to require `document-solution`

---

### Step 2: Generate Slug

Create a kebab-case slug (2-5 words) from the plan title.
Examples: `user-auth-v2`, `inspection-workflow`, `api-rate-limiting`

**Validate uniqueness** — scan ALL state directories in the hub:
```
plans/backlog/
plans/planned/
plans/active/
plans/completed/
plans/deprioritized/
plans/cancelled/
```

Also confirm the slug does not already appear in `plans/INDEX.md`.

- If an active/planned plan uses the slug → stop, ask user to choose a different title or confirm reuse (if reuse, skip Steps 3-5)
- If a completed/cancelled plan uses the slug → stop, ask user to choose a different slug or confirm they reference the existing historical plan (if reuse, skip Steps 3-5)

---

### Step 3: Create Plan Directory

Before creating files in `HUB_ROOT`:

**If `HUB_MODE=external`:**
```bash
cd "$HUB_ROOT"
git status --porcelain
```
If dirty, stop and ask the user to clean or stash first.
```bash
git fetch origin "${WORK_PLANS_BRANCH}"
git switch "${WORK_PLANS_BRANCH}" 2>/dev/null || \
  git switch --track -c "${WORK_PLANS_BRANCH}" "origin/${WORK_PLANS_BRANCH}"
git rebase "origin/${WORK_PLANS_BRANCH}"
```

**If `HUB_MODE=self-hosted`:** no git operations needed (same repo).

Create the directory structure:
```
plans/{state}/{slug}/
  plan.md         ← From references/master-plan-overview.md template
  STATUS.md       ← Aggregated status (initialized)
  repos/          ← Per-repo status files
    {repo-key-1}.md   ← From references/master-plan-repo-status.md template
    {repo-key-2}.md   ← One per target repo
```

Where `{state}` is the initial state from Step 1 (default: `planned`).

**plan.md**: Populate from the `references/master-plan-overview.md` template with gathered requirements. Include Quality/Validation Expectations, KB/Documentation Expectations, Cross-Repo Contracts, and Deployment Environments sections when applicable. Keep language product-focused — NO implementation details, NO task breakdowns, NO file ownership.

If creating in `planned` state, add `**Priority**: normal` after the Status field.

**STATUS.md**: Initialize with:
```markdown
# Status: {Plan Title}

**Overall**: planning
**Last Updated**: {ISO date}
**Updated By**: create-master-plan skill
```

If creating in `active` state, add `**Health**: on-track` after the Overall line.

```markdown
## Repo Progress

| Repo | Phase | Done/Total | Current Work | Blockers |
|------|-------|------------|--------------|----------|
| {repo-1} | — | 0/0 | Not started | None |
| {repo-2} | — | 0/0 | Not started | None |

## Timeline

| Date | Event |
|------|-------|
| {today} | Master plan created |

## Active Blockers
- None
```

**repos/{repo-key}.md**: Initialize from `references/master-plan-repo-status.md` with plan slug and canonical repo key, zero progress.

---

### Step 4: Update Plans Index

Add a row to the correct section of `plans/INDEX.md` based on the chosen initial state:

- `backlog` → **Backlog** section:
  ```
  | {slug} | {title} | {type} | {owner} | {today} |
  ```
- `planned` → **Planned** section:
  ```
  | {slug} | {title} | {type} | normal | {owner} | {target-demo} | {repo-1}, {repo-2} |
  ```
- `active` → **Active Plans** section:
  ```
  | {slug} | {title} | {type} | planning | {owner} | {target-demo} | {repo-1}, {repo-2} |
  ```

If `plans/INDEX.md` does not exist, create it from the `templates/plans-INDEX.md` template first.

---

### Step 5: Commit and Push (External Hub Only)

**If `HUB_MODE=external`:**
```bash
cd "$HUB_ROOT"
git add plans/{state}/{slug}/ plans/INDEX.md
git commit -m "feat: create master plan {slug}"
git push -u origin "${WORK_PLANS_BRANCH}"
```

Step 3 already rebased before file creation. Do NOT rebase again with uncommitted files.

If the push fails because `plans/INDEX.md` changed upstream: return checkout to clean state on `${WORK_PLANS_BRANCH}`, replay the generated files on top of the latest branch tip, then commit and push.

**If `HUB_MODE=self-hosted`:** commit locally in the current repo:
```bash
git add {PLANS_DIR}/plans/{state}/{slug}/ {PLANS_DIR}/plans/INDEX.md
git commit -m "feat: create master plan {slug}"
```

---

### Step 6: Report Summary

Present to the user:
- Plan title, slug, and type
- Hub location and mode (external / self-hosted)
- Repos involved
- Path to plan.md
- Reminder that each linked repo plan must carry forward the master plan's test/validation and KB/documentation expectations
- **Next steps**: For each target repo, run `/create-plan --master-plan {slug}` to create the repo-level technical plan linked to this master plan

## Output

- Master plan directory: `plans/{state}/{slug}/` containing `plan.md`, `STATUS.md`, and `repos/{repo-key}.md` per target repo
- Updated hub `STATUS.md` with initial repo progress table and timeline entry
- Updated `plans/INDEX.md` with new plan row in the correct state section
- Summary presented to user: plan title, slug, type, hub location, repos involved, path, and next steps
