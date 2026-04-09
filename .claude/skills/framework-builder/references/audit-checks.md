# Audit Checks

Check matrix for Mode C (Audit Framework) of the `framework-builder` skill.
Run checks based on the `--focus` parameter, or run all by default.

---

## Token Optimization Checks

- Agent definitions over 80 lines → flag for extraction
- Skill instructions over 150 lines → flag for reference extraction
- KB docs over 100 lines without `.compact.md` → flag for compact generation
- `_index.md` trigger keywords that match 3+ docs → flag overlapping triggers
- Skills that inline resolver logic instead of referencing `resolve-project-context.md`

## Consistency Checks

- Agent Workflow steps 1-4 match the `AGENT-RUNTIME-FLOW.md` initialization chain
- All agents use identical variable names from `resolve-project-context.md`
- All skills have the required sections: frontmatter, Context Required, Triggers, Instructions, Output
- Mirror parity: `agents/` = `.claude/agents/`; `skills/` = `.claude/skills/` = `.agents/skills/`

## Completeness Checks

- Every agent referenced in delegation tables actually exists
- Every skill referenced in agent workflows or trigger checklists actually exists
- Every KB stack folder has an `_index.md`
- `TRIGGER-CHECKLIST.md` lists all automatic triggers from all skills

## Drift Checks

- Cross-reference `AGENT-RUNTIME-FLOW.md` per-agent table against actual agent files
- Cross-reference `TRIGGER-CHECKLIST.md` against actual trigger sections in skills
- Check that `knowledge-base/README.md` doc counts match actual files
