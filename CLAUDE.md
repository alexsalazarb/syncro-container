# AGENTS.md - AI Agent Instructions

Cursor, Codex, Windsurf, and Copilot read this file directly. Claude Code uses `CLAUDE.md`, Antigravity uses `.agent/rules/agent.md` as shims.

---

## Container repos: resolve active project first

> **Skip this section** if `.ai-framework.config` is absent or `LAYOUT=single` (or `LAYOUT` is not set).

If this file is the root `AGENTS.md` of a **container repository** (multiple projects in one repo), do the following **before any other work**:

1. Read `.ai-framework.config` to get `LAYOUT`, `PROJECTS`, and `AI_CONTEXT_ROOT`
2. Infer the **active project** from context: which folder does the open file belong to? Which project did the user mention?
3. Resolve paths using `skills/common/resolve-project-context.md`:
   - `PROJECT_CONTEXT_ROOT` = `{folder}/` (embedded) or `{AI_CONTEXT_ROOT}/{folder}/` (isolated)
   - `AGENTS_MD` = `{PROJECT_CONTEXT_ROOT}AGENTS.md`
   - `KB_DIR` = `{folder}/docs/kb-project/` (embedded) or `{PROJECT_CONTEXT_ROOT}/` (isolated)
4. Read `{AGENTS_MD}` — this contains stack-specific conventions, build commands, and constraints for the active project
5. Read `{KB_DIR}/README.md` (Layer 3 project KB index)

Without this step, you will work with generic container-level guidance only and miss all stack-specific conventions.

---

## QUICK TRIGGERS (Memorize These)

**Session Start**: Check `docs/kb-container/ai-patterns/mistake-log.md` for patterns to avoid. If `docs/kb-container/conventions.md` (L2) or `{KB_DIR}/conventions.md` (L3) exists, read it for project-specific rules.

**During Session**:
- User says "commit/stage" → run `pre-commit-check`
- You get corrected → run `log-mistake`
- KB lookup fails → after solving, run `document-solution`
- You modify KB files → run `check-kb-index`

**Session End** (user says "done/thanks/bye"): Run `session-end-checklist`

---

## Project Context

**Syncro MSP**: Mobile field-service platform for IT technicians — tickets, assets, real-time chat, appointments, and time tracking.

| Aspect | Value |
|--------|-------|
| Product | Syncro MSP mobile app |
| Projects | `syncro-flutter` (iOS + Android) |
| Language/Framework | Flutter / Dart 3.x |
| Main branch | `main` |
| Deployment | App Store + Google Play |
| Testing | `flutter_test` + `mockito` + `bloc_test` |
| AI Context Root | `docs/kb-projects/` |
| Container KB | `docs/kb-container/` |

**Key commands**:
```bash
# syncro-flutter (uses fvm — always prefix with fvm)
cd syncro-flutter
fvm flutter run                # run on connected device
fvm flutter test               # all tests
fvm flutter analyze            # lint
fvm dart format .              # format
fvm flutter pub run build_runner build --delete-conflicting-outputs  # regenerate mocks
```

---

## Knowledge Base

### 3-Layer Loading (follow this order)

1. **Layer 1 — Framework KB**: Read `.ai-framework.config` to get `STACK`. Then read `.ai-framework/knowledge-base/{STACK}/_index.md`. Match task keywords against Triggers. Load `.compact.md` first; escalate to full doc only if compact is insufficient.
2. **Layer 2 — Container KB**: Read `docs/kb-container/README.md` at repo root for cross-project domain docs (in single layout, same as Layer 3).
3. **Layer 3 — Project KB**: Read `docs/kb-container/README.md` for project-specific docs.

Load ONLY relevant docs. Do not load entire KB.

### KB-First Rule

Before exploring code in any area, check KB across all three layers:

1. Check Layer 1 `_index.md` triggers for the topic
2. Check Layer 2/3 `README.md` indexes
3. If a matching KB doc exists → read it before any grep/read operations on that code
4. Only explore code directly if no KB doc covers the area
5. If you explore code directly and discover important patterns → create KB documentation using `document-solution`
6. If you modify functionality that has existing KB docs → update those docs to reflect the changes

This applies mid-task too. If you start investigating a different subsystem, re-check the KB indexes before exploring that code.

**Why**: KB docs contain curated knowledge that prevents unnecessary code exploration.

### KB Loading

When loading Layer 2 (`CONTAINER_KB_DIR`) or Layer 3 (`KB_DIR`):
- If `kb-index.yaml` exists at the root: use trigger-based loading — match keywords, load compact variant first, escalate to full doc if needed, co-load `load_with` siblings.
- If absent: scan `README.md` for relevant docs (current behavior).

See `docs/KNOWLEDGE-LAYERING.md` for the full schema and `docs/AGENT-RUNTIME-FLOW.md` for the decision algorithm.

### Optional: technology choices per topic

Projects do **not** need this on day one. When you standardize (e.g. async/await vs sync execution, REST vs GraphQL, relational vs document storage), add Layer 3 docs and list them here so agents load the right framework guides and skip conflicting ones.

See `.ai-framework/docs/KB_TOPIC_STANDARDS.md` for the supported shapes (per-topic files, one routing table, or AGENTS-only).

<!-- CUSTOMIZE (optional): When ready, uncomment and link your Layer 3 docs -->
<!-- - Concurrency: {KB_DIR}/technical/architecture/concurrency-standard.md -->
<!-- - Persistence: {KB_DIR}/technical/architecture/persistence-standard.md -->
<!-- - Or single table: {KB_DIR}/technical/architecture/technology-choices.md -->

---

## Critical Constraints

<!-- CUSTOMIZE: Add your project's critical constraints -->
1. **[Constraint 1]** - Description
2. **[Constraint 2]** - Description
3. **No magic strings** - Use constants, enums, or config values
4. **Tests required** - Include tests for all changes

---

## Things to Avoid

1. Over-engineering - only make directly requested changes
2. Breaking existing tests - always run tests before committing
3. Creating documentation unless explicitly requested
4. Magic strings - use constants/enums/config
5. Ignoring existing patterns in the codebase

<!-- CUSTOMIZE: Add project-specific anti-patterns -->

---

## Mandatory Auto-Triggers

These MUST fire automatically. Do not wait for user to ask:

| Trigger | Action | Why |
|---------|--------|-----|
| User says "commit", "stage", "git add" | `pre-commit-check` | Catch violations |
| KB lookup fails → solved via code | `document-solution` | Update KB |
| User corrects you | `log-mistake` | Build patterns |
| Modified KB files | `check-kb-index` | Keep index current |
| User ending session | `session-end-checklist` | Catch missed automation |

### Trigger Detection Examples

```
USER: "let's commit these changes"
→ Detected "commit" → run pre-commit-check BEFORE git operations

USER: "no that's wrong, it should be..."
→ Detected correction phrase → run log-mistake

USER: "thanks, that's all for today"
→ Detected session end → run session-end-checklist
```

**Correction phrases** (run `log-mistake` immediately):
"that's wrong", "actually...", "no, it should be...", "you forgot to...", "you missed..."

Do NOT wait for explicit requests - detect and trigger proactively.

---

## Skills & Agents

Skills: `.agents/skills/` | `.claude/skills/`
Agents: `.claude/agents/` (orchestrator, planner, implementer, reviewer, researcher, documenter, tester, test-writer)

| Skill | When |
|-------|------|
| `pre-commit-check` | Before commit/staging |
| `session-end-checklist` | Session ending |
| `log-mistake` | User corrects you |
| `document-solution` | Complex problem solved or KB miss |
| `check-kb-index` | After KB file changes |
| `save-session` | Long session (20+ turns) |
| `check-test-coverage` | After implementing feature/fix |
| `check-agent-drift` | Periodic / on request |
| `cleanup-sessions` | Manual / maintenance |
| `create-new-build` | Explicitly requested build bump (develop only) |
| `list-skills` | Discover available automation |
| `create-plan` | Starting a new feature/project |
| `create-bug-plan` | Investigating + planning a production bug fix |
| `add-defect` | QA/reviewer adds a bug to an existing plan |
| `execute-task` | Working on a specific plan task |
| `execute-plan` | Running remaining plan tasks (parallel agents) |
| `manage-contracts` | Managing cross-repo interface contracts |
| `archive-plan` | Archiving completed plans and cleaning up linked repo plans |
| `transition-plan` | Moving plan between lifecycle states (promote, deprioritize, cancel) |
| `create-master-plan` | Creating cross-repo master plans in the work-plans repo |
| `sync-master-plan` | Syncing repo plan status to linked master plan |
| `plans-status` | Aggregated plan status dashboard |
| `pr-cleanup` | Automated CodeRabbit triage and CI polling for PRs |
| `upgrade-framework` | AI-assisted framework upgrade with intelligent skill review |
| `capture-convention` | Capture a project-specific rule to conventions.md (L2 or L3) |
| `generate-kb-index` | Generate kb-index.yaml stub for README-based KB directories |
| `research-implementation` | Proactive KB gap research before planning or building |
*Invoke skills with `/skill-name` or natural language. Legacy commands: `docs/ai-commands/`*

### Hub Commands (cross-repo master plans)
- `/create-master-plan` — Create a product-focused master plan in the work-plans hub
- `/transition-plan {slug} --to {state}` — Move master plan between lifecycle states
- `/sync-master-plan` — Sync local plan progress to hub
- `/plans-status` — Aggregated plan status dashboard

---

## Self-Documentation Rules

### Update KB when:
- KB lookup failed but you solved via code search → run `document-solution`
- Complex solution (3+ files, 5+ exchanges) → run `document-solution`
- After any KB change → run `check-kb-index`

### Format:
- Kebab-case filenames
- Include "Last Updated" date
- Include "Context" section

---

## Quick Reference

<!-- CUSTOMIZE: Update with your project commands -->
| Task | Command |
|------|---------|
| Run tests | `[test command]` |
| Lint | `[lint command]` |
| Build | `[build command]` |

---

## Memory

You have access to Engram persistent memory via MCP tools (mem_save, mem_search, mem_session_summary, etc.).
- Save proactively after significant work — don't wait to be asked.
- After any compaction or context reset, call `mem_context` to recover session state before continuing.

---

## REMEMBER (End of File)

Before responding, check:
1. **Am I being corrected?** → Run `log-mistake`
2. **Is user committing/staging?** → Run `pre-commit-check`
3. **Is user ending session?** → Run `session-end-checklist`
4. **Did I solve something complex without KB?** → Run `document-solution`

Detect and execute automatically.
