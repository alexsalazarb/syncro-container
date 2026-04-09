## FIRST: Resolve Project Context

### Container repos — do this before anything else

> **Skip** if `.ai-framework.config` is absent or `LAYOUT` is not `container`.

If this repo uses a container layout:

1. Read `.ai-framework.config` → get `LAYOUT`, `PROJECTS`, `AI_CONTEXT_ROOT`
2. Infer the **active project** from context (open file, user mention, working directory)
3. Resolve paths via `skills/common/resolve-project-context.md`:
   - `AGENTS_MD` = `{PROJECT_CONTEXT_ROOT}AGENTS.md`
   - `KB_DIR` = `{folder}/docs/kb-project/` (embedded) or `{PROJECT_CONTEXT_ROOT}/` (isolated)
4. Read `{AGENTS_MD}` — stack conventions, build commands, constraints
5. Read `{KB_DIR}/README.md` — project KB index (Layer 3)

Without this step you work with generic guidance only and miss all project-specific conventions.

### Single-layout repos

Read root `AGENTS.md` and `docs/kb-container/README.md` (or `{KB_DIR}/README.md` as configured).

---

## DURING SESSION

- Load KB docs when task keywords match the README index
- On EVERY user message, check triggers: commit? correction? session end? → run matching skill
- When context appears lost (unable to recall project conventions), re-read `{AGENTS_MD}`

---

## Quick Reference

- This shim is intentionally generic.
- Read `{AGENTS_MD}` for the repo's actual stack, branch strategy, test commands, and constraints.
- Do not replace project-specific instructions with placeholders here.

## Critical Constraints

- Project-specific constraints live in `{AGENTS_MD}`.
- **No magic strings**: Use constants, enums, or config values

## Mandatory Auto-Triggers (Do NOT Skip)

These behaviors are MANDATORY. Do not wait for user to ask:

| Trigger | Action |
|---------|--------|
| User says "commit", "stage", "git add" | Run `pre-commit-check` BEFORE the operation |
| KB lookup failed → solved via code search | Run `document-solution` to update KB |
| User corrects you ("that's wrong", "actually...", etc.) | Run `log-mistake` immediately |
| Modified any KB file | Run `check-kb-index` |
| User says "done", "thanks", "bye", ending session | Run `session-end-checklist` |

Skill procedures available in `docs/ai-commands/` for detailed instructions.

## Key Patterns

- Re-read `{AGENTS_MD}` and relevant KB docs before making assumptions about stack-specific patterns.
- Treat this shim as a pointer to canonical project instructions, not as a second source of truth.
