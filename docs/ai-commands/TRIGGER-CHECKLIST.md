# Trigger Checklist

Quick reference for automatic skill triggers. These fire without being asked.

| Trigger Words | Skill | Notes |
|---------------|-------|-------|
| "commit", "stage", "git add", "create a PR" | `pre-commit-check` | BEFORE git operations |
| "that's wrong", "actually...", "you forgot", "you missed" | `log-mistake` | IMMEDIATELY on detection |
| "done", "thanks", "bye", "that's all" | `session-end-checklist` | BEFORE ending |
| Any file in `{CONTAINER_KB_DIR}/` (Layer 2) or `{KB_DIR}/` (Layer 3) created/modified | `check-kb-index` | AFTER the change |
| KB lookup missed → solved via code search; bug fix 3+ files; non-obvious pattern; 5+ exchanges | `document-solution` | AFTER solving the problem |
| Tiered KB loading found no match → solved; after `contribute-pattern` promotes a project doc | `add-kb-doc` | Fill framework KB gap |
| `execute-task` completes a task linked to a master plan | `sync-master-plan` | AFTER task completion |
| `audit-and-plan` completes with framework KB gaps; `cleanup-kb` finds used libs without KB docs | `research-kb-topic` | Suggest (don't auto-run) |
| Session start detects `.ai-framework` submodule is behind remote | `upgrade-framework` | AT session start |

## Manually Invoked Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| `pre-commit-check` | `/pre-commit-check` | Validate changes before commit |
| `session-end-checklist` | `/session-end-checklist` | Wrap-up checklist |
| `log-mistake` | `/log-mistake` | Log a correction |
| `document-solution` | `/document-solution` | Add to knowledge base |
| `check-kb-index` | `/check-kb-index` | Sync KB index |
| `save-session` | `/save-session` | Create handoff document |
| `create-plan` | `/create-plan [description]` | Plan a new feature |
| `create-bug-plan` | `/create-bug-plan [description]` | Create a bug investigation & fix plan |
| `execute-task` | `/execute-task {plan}/{task}` | Work on a plan task |
| `execute-plan` | `/execute-plan {plan}` | Run all remaining tasks |
| `archive-plan` | `/archive-plan {plan}` | Archive completed plan |
| `add-defect` | `/add-defect {plan-slug}` | Add a defect task to an existing plan |
| `manage-contracts` | `/manage-contracts {plan-slug}` | Create/freeze/update/deprecate interface contracts |
| `audit-and-plan` | `/audit-and-plan` | Audit codebase, generate KB docs and improvement plan |
| `check-agent-drift` | `/check-agent-drift` | Verify AGENTS.md reflects current codebase |
| `check-test-coverage` | `/check-test-coverage` | Map source files to test files, identify gaps |
| `promptcraft` | `/promptcraft [type] [description]` | Generate optimized prompts for AI sessions |
| `cleanup-kb` | `/cleanup-kb` | Remove unused/stale KB docs |
| `cleanup-sessions` | `/cleanup-sessions` | Delete old session handoff documents |
| `contribute-pattern` | `/contribute-pattern` | Contribute a reusable pattern back to the framework KB |
| `add-kb-doc` | `/add-kb-doc` | Add a knowledge base document to the framework KB |
| `plans-status` | `/plans-status` | Show aggregated plan status dashboard (local + hub) |
| `create-master-plan` | `/create-master-plan` | Create a master plan in the work-plans hub |
| `transition-plan` | `/transition-plan {slug} --to {state}` | Move a master plan between lifecycle states |
| `sync-master-plan` | `/sync-master-plan` | Sync local plan progress to the hub |
| `merge-framework` | `/merge-framework` | Merge AI framework conventions into existing AGENTS.md |
| `pr-cleanup` | `/pr-cleanup` | Triage PR review threads, fix CI, loop until clean |
| `upgrade-framework` | `/upgrade-framework` | AI-assisted framework upgrade for a consumer repo |
| `list-skills` | `/list-skills` | Show all available skills |
| `research-kb-topic` | `/research-kb-topic [--scan\|technology]` | Detect KB gaps and research missing technology docs |
| `framework-builder` | `/framework-builder [component-type] [description]` | Generate or improve framework components (agents, skills, KB, plans) |
| `framework-review` | `/framework-review [component-path\|--scope type]` | Critical review of framework components with structured report |
