
# Framework Review

## Context Required
HIGH-CONTEXT: Framework architecture docs, target component(s), dependency graph

## Triggers

### Automatic
- After `framework-builder` creates or modifies a component (recommended, not mandatory)

### Manual
- `/framework-review`
- `/framework-review [component-path]`
- `/framework-review --scope agents|skills|kb|plans|all`
- "review this agent definition"
- "review the framework"
- "audit framework health"
- "is this skill well-designed?"

## Instructions

### Step 0: Load Framework Context

Read `.ai-framework.config` from the project root. Since this skill operates on the framework itself, resolve:

- `FRAMEWORK_ROOT` — `.ai-framework/` (or repo root if inside the framework repo)
- Agent definitions: `{FRAMEWORK_ROOT}/agents/`
- Skill definitions: `{FRAMEWORK_ROOT}/skills/`
- KB root: `{FRAMEWORK_ROOT}/knowledge-base/`
- Architecture docs: `{FRAMEWORK_ROOT}/docs/`
- Plans: `{PLANS_DIR}/` from config (default `.plans`)

Load authoritative references:
1. `docs/AGENT-RUNTIME-FLOW.md` — initialization chain, per-agent behavior table
2. `skills/common/resolve-project-context.md` — resolver contract and variable names
3. `skills/TRIGGER-CHECKLIST.md` — trigger conventions
4. `agents/framework-architect.md` § Invariants — the 7 invariants that must hold

---

### Step 1: Determine Review Scope

| Scope | What to read | When to use |
|-------|-------------|-------------|
| **Single component** | The target agent `.md` or skill `SKILL.md` + its references | After creating or modifying one component |
| **Component type** | All agents, all skills, all KB stacks, or all plans | Periodic type-level audit |
| **Full framework** | Everything — agents, skills, KB, plans, architecture docs | Quarterly health check or after major changes |
| **Diff-based** | Only files changed since a git ref (branch, tag, commit) | PR review or post-implementation review |

If scope is ambiguous, ask: "What should I review? (specific file path / agents / skills / kb / plans / all)"

For diff-based scope, identify changed files:
```
git diff --name-only {base-ref}...HEAD
```
Then filter to framework-relevant files (agents/, skills/, knowledge-base/, docs/).

---

### Step 2: Build Dependency Map

For each component under review, trace its connections:

**For an agent:**
- Which skills does its Workflow reference?
- Which other agents does its Delegation table target?
- Which KB docs would its trigger keywords load?
- Which architecture docs does it reference?

**For a skill:**
- Which agents invoke it (search agent definitions for the skill name)?
- Which `references/` files does it use?
- Which shared logic in `skills/common/` does it reference?
- Which other skills does it call or chain to?

**For KB docs:**
- Which `_index.md` or `README.md` entries point to it?
- Which agent/skill triggers would load it?
- Does a `.compact.md` variant exist?

Record the dependency map — it reveals blast radius and orphan risk.

---

### Step 3: Run the 6-Dimension Analysis

Apply each dimension from [`references/review-rubric.md`](references/review-rubric.md). For each dimension, evaluate every component in scope.

**Dimension A — Agent Design**: Responsibilities, model tier, access level, delegation, workflow inflation
**Dimension B — Skill Design**: Single-purpose, trigger precision, Step 0 compliance, actionable instructions, reference extraction
**Dimension C — Knowledge Base**: Layer correctness, trigger precision, compact parity, duplication, index sync
**Dimension D — Plan System**: Task boundaries, dependencies, kill criteria, phasing, validation steps
**Dimension E — Token & Context Efficiency**: Line budgets, over-fetching, conditional gating, shared logic duplication
**Dimension F — Maintainability & Scalability**: Naming, initialization chain, mirror parity, comprehensibility, growth resilience

Skip dimensions that don't apply to the review scope (e.g., skip Dimension D for a single agent review).

---

### Step 4: Measure Token Impact

For each component under review, collect concrete metrics:

| Metric | How to measure | Target |
|--------|---------------|--------|
| Definition length | `wc -l {file}` | Agent ≤ 80, Skill ≤ 150 |
| Reference files length | `wc -l references/*.md` | Each ≤ 100 |
| KB docs without compact | Count `*.md` without matching `*.compact.md` | 0 for docs > 100 lines |
| Trigger keyword overlap | Scan `_index.md` for keywords appearing in 3+ doc rows | 0 overlaps |
| Mirror parity | `diff` canonical vs mirror files | 0 differences |
| Dead references | Grep for referenced paths that don't exist | 0 dead refs |

Report actual values alongside targets. Show the delta.

---

### Step 5: External Benchmarking (optional)

Skip this step unless the analysis in Steps 3-4 reveals a questionable architectural choice or the user explicitly requests benchmarking.

When activated:
1. Identify the specific concern (e.g., "agent delegation pattern", "KB tiered loading", "plan-driven execution")
2. Search for comparable open-source frameworks or published research
3. Compare the framework's approach against 2-3 alternatives
4. Assess migration cost and token impact of adopting the alternative

Integration rules:
- External findings supplement internal analysis — they don't override it
- Every external suggestion must include: source, migration cost estimate, and token impact
- Never adopt external patterns wholesale; propose targeted adaptations

---

### Step 6: Classify Findings

Assign severity to every finding:

| Severity | Criteria | Required response |
|----------|----------|-------------------|
| `BLOCKER` | Breaks an invariant, causes incorrect agent behavior, or wastes >50 lines of unnecessary context per invocation | Must fix before merge |
| `WARNING` | Suboptimal design that will degrade at scale or confuse future contributors | Fix or explicitly justify |
| `NOTE` | Improvement opportunity with clear benefit but no urgency | Track for next refactoring |
| `QUESTION` | Ambiguity the reviewer cannot resolve — needs architect or user input | Respond with clarification |

For each finding, fill this structure:
```
[SEVERITY] Component: {path}
Problem: {what is wrong}
Why it matters: {concrete consequence — token waste, breakage risk, confusion}
Proposed fix: {specific action, not vague guidance}
Expected impact: {lines saved, risk eliminated, clarity gained}
Priority: {high / medium / low}
```

---

### Step 7: Produce Review Report

Generate the report following [`references/review-report-template.md`](references/review-report-template.md).

Sections:
1. **Executive Summary** — 2-3 sentences: scope reviewed, overall health, critical finding count
2. **What Works Well** — brief acknowledgment of strong design choices (max 5 bullets — don't pad)
3. **Findings** — all findings grouped by dimension, each with the severity structure from Step 6
4. **Token & Context Optimization** — dedicated section for efficiency findings with before/after metrics
5. **Architecture Recommendations** — cross-cutting improvements that span multiple dimensions
6. **Priority Actions** — top 3-5 findings ranked by impact, with suggested execution order

The report must be self-contained — a reader should understand every finding without loading additional files.

---

### Step 8: Suggest Follow-ups

Based on the review:
- If `BLOCKER`s exist: recommend running `framework-builder` in Improve mode on the affected components
- If token waste is systemic: recommend a `create-plan` for framework-wide optimization
- If KB issues found: recommend `check-kb-index` and optionally `cleanup-kb`
- If mirror drift detected: recommend `sync-skills.sh`
- If the review itself surfaced a KB gap: recommend `document-solution` to capture the insight

## Output

- Structured review report (per § Output Format in the agent definition)
- Severity-classified findings with concrete fix recommendations
- Token impact metrics with before/after comparisons
- Priority action list for the framework-architect
- Suggested follow-up skills or plans
