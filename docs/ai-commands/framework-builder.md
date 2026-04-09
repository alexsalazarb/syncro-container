
# Framework Builder

## Context Required
HIGH-CONTEXT: Framework architecture docs, existing component exemplars, AGENTS.md

## Triggers

### Automatic
- None (manual only)

### Manual
- `/framework-builder`
- `/framework-builder [component-type] [description]`
- "build a new agent for..."
- "create a new skill for..."
- "improve this agent definition"
- "optimize the framework"
- "audit framework components"
- "refactor this skill"

## Instructions

### Step 0: Load Framework Context

Read `.ai-framework.config` from the project root. Since this skill operates on the framework itself, the relevant paths are:

- `FRAMEWORK_ROOT` — `.ai-framework/` (or repo root if working inside the framework repo)
- Agent definitions: `{FRAMEWORK_ROOT}/agents/` and mirrors
- Skill definitions: `{FRAMEWORK_ROOT}/skills/` and mirrors
- KB root: `{FRAMEWORK_ROOT}/knowledge-base/`
- Architecture docs: `{FRAMEWORK_ROOT}/docs/`
- Plans: `{PLANS_DIR}/` from config (default `.plans`)

Load these authoritative references (compact-first where available):
1. `docs/AGENT-RUNTIME-FLOW.md` — agent initialization chain, per-agent behavior table
2. `skills/common/resolve-project-context.md` — resolver algorithm and variable names
3. `skills/TRIGGER-CHECKLIST.md` — trigger conventions
4. `docs/KNOWLEDGE-LAYERING.md` — 3-layer KB model (scan for structure, don't deep-read unless KB work)
5. `docs/KB_TOPIC_STANDARDS.md` — topic standard routing (scan only)

---

### Step 1: Classify the Request

Determine the operation mode from the user's input:

| Mode | Trigger | Output |
|------|---------|--------|
| **Generate Agent** | "new agent", "build agent", "agent for X" | New agent `.md` in `agents/` + mirrors |
| **Generate Skill** | "new skill", "build skill", "skill for X" | New `SKILL.md` in `skills/{slug}/` + mirrors |
| **Generate Plan Template** | "plan template", "reference template" | New template in `skills/{skill}/references/` |
| **Improve Component** | "improve", "optimize", "refactor" + component path | Edited component across all mirrors |
| **Audit Framework** | "audit", "review framework", "check health" | Diagnostic report with prioritized recommendations |
| **Generate KB Structure** | "KB for X stack", "knowledge base structure" | Stack folder with `_index.md` and stub docs |

If unclear, ask: "What type of component? (agent / skill / plan template / KB structure / audit)"

---

### Step 2: Gather Requirements

Collect the following based on mode. Use defaults where the user doesn't specify.

#### For Agent Generation
- **Role name** — kebab-case slug (e.g. `security-auditor`)
- **Role description** — what the agent does in one paragraph
- **Model tier** — `opus` (high-reasoning), `sonnet` (standard), `haiku` (lightweight). Default: `sonnet`
- **Access level** — `full`, `read-only`, `read-only + web`, `coordination-only`
- **Tools** (if restricted) — explicit allowlist, or omit for full access
- **Delegation targets** — which existing agents this one delegates to
- **KB interaction pattern** — indexes only (like orchestrator), tiered loading (like implementer), or routing (like documenter)

#### For Skill Generation
- **Skill name** — kebab-case slug (e.g. `security-scan`)
- **Skill description** — what the skill does in one paragraph
- **Context level** — `META` (no project context), `LOW-CONTEXT`, `HIGH-CONTEXT`, `FULL-CONTEXT`
- **Trigger type** — automatic (with conditions) or manual only
- **Inputs** — what the skill needs from the user or calling agent
- **Outputs** — what artifacts the skill produces
- **References needed** — whether the skill needs `references/` templates

#### For Audit
- **Scope** — `all` (full framework), `agents`, `skills`, `kb`, `plans`, or a specific component path
- **Focus** — `token-optimization`, `consistency`, `completeness`, `drift`, or `all`

---

### Step 3: Read Exemplars

Before generating, read 2-3 existing components of the same type to extract current patterns:

**For agents** — always read:
- `agents/orchestrator.md` — most complex agent, shows coordination patterns
- `agents/implementer.md` — standard agent with KB escalation protocol
- One agent closest to the new agent's role

**For skills** — always read:
- `skills/create-plan/SKILL.md` — complex skill with references and phased output
- `skills/document-solution/SKILL.md` — skill with routing logic and KB interaction
- One skill closest to the new skill's purpose

**For audit** — scan directory listings:
- `ls agents/*.md` — count and list all agent definitions
- `ls skills/*/SKILL.md` — count and list all skill definitions
- `ls knowledge-base/*/` — list all stack KB folders

Extract from exemplars:
- Section ordering and naming conventions
- How Step 0 (resolver) is referenced
- How KB loading is structured (3-layer pattern)
- How plans awareness is included
- How guidelines are phrased

---

### Step 4: Execute by Mode

#### Mode A: Generate Agent

**4a-1. Draft the agent definition** using the Agent Definition Template from [`references/component-templates.md`](references/component-templates.md).

**4a-2. Validate against invariants:**
- Steps 1-4 of Workflow match the initialization chain from `AGENT-RUNTIME-FLOW.md`
- Variable names match `resolve-project-context.md` exactly
- Model tier is justified by the agent's cognitive requirements
- Tools list (if present) matches the access level description

**4a-3. Write to all mirror locations:**
- `agents/{slug}.md`
- `.claude/agents/{slug}.md`

**4a-4. Update the per-agent behavior table** in `docs/AGENT-RUNTIME-FLOW.md` § 5.

#### Mode B: Generate Skill

**4b-1. Draft the skill definition** using the Skill Definition Template from [`references/component-templates.md`](references/component-templates.md).

**4b-2. Apply token optimization to the draft:**

| Check | Action |
|-------|--------|
| Step 0 duplicates resolver logic | Replace with reference to `../common/resolve-project-context.md` |
| Any step exceeds 30 lines | Extract substeps or move details to `references/` |
| Inline templates exceed 20 lines | Move to `references/{template-name}.md` |
| Conditional logic with 3+ branches | Use a decision table instead of prose |
| Same instruction appears in another skill | Extract to `skills/common/` |

**4b-3. Create reference templates** (if needed) in `skills/{slug}/references/`.

**4b-4. Write to all mirror locations:**
- `skills/{slug}/SKILL.md`
- `.claude/skills/{slug}/SKILL.md`
- `.agents/skills/{slug}/SKILL.md`
- (and any `references/` files to all three trees)

**4b-5. Add to `skills/TRIGGER-CHECKLIST.md`** under the appropriate section (Automatic or Manually Invoked).

#### Mode C: Audit Framework

**4c-1. Scan all components:**

```
agents/*.md          → agent definitions
skills/*/SKILL.md    → skill definitions
skills/common/       → shared logic
knowledge-base/*/    → KB stacks and indexes
docs/                → architecture docs
.plans/              → plan templates and active plans
```

**4c-2. Run checks based on focus:**

> See [`references/audit-checks.md`](references/audit-checks.md) for the full check matrix (token optimization, consistency, completeness, drift).

**4c-3. Produce the audit report** using the Audit Report Template from [`references/component-templates.md`](references/component-templates.md).

#### Mode D: Generate KB Structure

**4d-1.** Create the stack folder under `knowledge-base/{stack}/`
**4d-2.** Generate `_index.md` with trigger-based routing table
**4d-3.** Create stub docs for the stack's core topics (architecture, patterns, integrations)
**4d-4.** Generate `.compact.md` variants for any doc over 100 lines
**4d-5.** Create `README.md` catalog
**4d-6.** Run `check-kb-index` to verify index integrity

---

### Step 5: Post-Generation Validation

After writing any component, verify:

1. **Structural compliance** — required sections present, frontmatter valid
2. **Cross-references valid** — all referenced files, skills, agents exist
3. **Mirror parity** — all copies are identical
4. **No orphaned triggers** — new automatic triggers are in `TRIGGER-CHECKLIST.md`
5. **Token budget** — count lines; compare against component-type budgets:
   - Agent definitions: target ≤ 80 lines (hard limit: 120)
   - Skill instructions: target ≤ 150 lines (hard limit: 250, use references for overflow)
   - KB docs: target ≤ 100 lines (generate compact if over)

---

### Step 6: Report

Present results to the user:

```text
Framework Builder — {mode} complete

Created/Modified:
- {path1} ({lines} lines)
- {path2} ({lines} lines)

Mirrors updated:
- {mirror paths}

Indexes updated:
- {index paths}

Token impact: {+/-N lines} across {M} files
Next steps: {suggestions}
```

## Output

- New or modified framework components at canonical and mirror locations
- Updated indexes (`TRIGGER-CHECKLIST.md`, `AGENT-RUNTIME-FLOW.md`, KB indexes)
- Audit report (for audit mode)
- Token impact summary
- Suggested follow-up actions
