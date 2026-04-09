# Review Rubric

Reference checklist for the `framework-review` skill.
Each dimension contains specific questions. A "Yes" to any red-flag column means a finding should be logged.

---

## Dimension A: Agent Design

| # | Question | Red flag (log finding if true) |
|---|----------|-------------------------------|
| A1 | Are responsibilities clearly bounded to one domain? | Agent description requires "and" connecting unrelated domains |
| A2 | Could this agent be merged into an existing one? | 70%+ responsibility overlap with another agent |
| A3 | Is the model tier justified? | `opus` for template work; `haiku` for multi-step reasoning |
| A4 | Does the tools list match declared access? | `full` access but tools list present; `read-only` but tools list missing |
| A5 | Does every delegation target exist? | Delegation table references a non-existent agent |
| A6 | Does the workflow restate skill logic? | Workflow step duplicates instructions already in a skill's SKILL.md |
| A7 | Are Steps 1-4 the standard initialization chain? | Steps deviate from `AGENT-RUNTIME-FLOW.md` §§2-4 |
| A8 | Is the definition within budget? | >80 lines (warning) or >120 lines (blocker) |
| A9 | Does it have a KB Escalation Protocol? | Missing protocol for handling KB gaps |
| A10 | Are guidelines actionable? | Vague guidelines like "be careful" instead of specific checks |

---

## Dimension B: Skill Design

| # | Question | Red flag |
|---|----------|----------|
| B1 | Does the skill do exactly one thing? | Skill has 2+ unrelated output types |
| B2 | Are automatic triggers precise? | Trigger phrase matches common conversation ("thanks", "done") unintentionally |
| B3 | Do manual triggers overlap with another skill? | Same slash command or phrase triggers two skills |
| B4 | Does Step 0 reference `resolve-project-context.md`? | Inline resolver logic instead of reference |
| B5 | Are instructions specific and testable? | Steps use "analyze appropriately" or "review as needed" |
| B6 | Are templates/code blocks extracted to references? | Inline template >20 lines or code block >15 lines |
| B7 | Is the output section well-defined? | No artifact list; unclear deliverables |
| B8 | Is the skill within budget? | >150 lines (warning) or >250 lines (blocker) |
| B9 | Does it update indexes when creating artifacts? | Creates files but doesn't update README or TRIGGER-CHECKLIST |
| B10 | Are conditional sections gated with skip directives? | Reader must parse 10+ lines to determine applicability |

---

## Dimension C: Knowledge Base

| # | Question | Red flag |
|---|----------|----------|
| C1 | Is content at the correct layer? | Project-specific pattern in Layer 1; generic pattern in Layer 3 |
| C2 | Do trigger keywords cause false-positive loading? | Keyword appears in 3+ docs in the same `_index.md` |
| C3 | Does every doc >100 lines have a `.compact.md`? | Missing compact variant |
| C4 | Is the compact variant up-to-date with the full doc? | Full doc modified more recently than compact |
| C5 | Is the same concept explained in multiple layers? | Duplication across Layer 1 and Layer 2/3 |
| C6 | Does `_index.md` / `README.md` match actual files? | Index entry with no file; file with no index entry |
| C7 | Are trigger keywords specific enough? | Single-word triggers like "architecture" or "data" |
| C8 | Is the doc structure consistent? | Missing "Last Updated", "Context", or section headings |

---

## Dimension D: Plan System

| # | Question | Red flag |
|---|----------|----------|
| D1 | Do parallel tasks have non-overlapping file ownership? | Same file in two tasks' File Ownership lists |
| D2 | Are dependencies explicitly declared? | Task reads output of prior task but "Depends On" is empty |
| D3 | Are kill criteria defined? | `overview.md` has no kill criteria section |
| D4 | Is phasing logically ordered? | Schema changes after business logic; UI before data model |
| D5 | Do tasks have completion criteria? | `task.md` has no testable completion criteria |
| D6 | Are branch names consistent? | Tasks use branches that don't follow `plan/{slug}/{task-path}` |
| D7 | Is the plan appropriately sized? | >15 tasks or >4 phases without justification |

---

## Dimension E: Token & Context Efficiency

| # | Question | Red flag |
|---|----------|----------|
| E1 | Is the agent/skill within its line budget? | See A8, B8 |
| E2 | Does the agent load all 3 KB layers for a narrow task? | Loads Layer 2 container docs for a project-local change |
| E3 | Are conditional sections gated? | No skip directive on sections that apply <50% of the time |
| E4 | Is shared logic duplicated? | Same instruction block in 2+ skills instead of `skills/common/` |
| E5 | Are references used for large blocks? | See B6 |
| E6 | Could the component be split to reduce per-invocation cost? | Single skill that's only partially relevant to most invocations |
| E7 | Are KB triggers causing over-fetching? | Agent typically loads 5+ docs per task |

---

## Dimension F: Maintainability & Scalability

| # | Question | Red flag |
|---|----------|----------|
| F1 | Are naming conventions consistent? | Mixed kebab-case/camelCase; slug doesn't match frontmatter name |
| F2 | Do all agents follow the same initialization chain? | Agent skips or reorders Steps 1-4 |
| F3 | Are all mirrors in parity? | `diff` between canonical and mirror files shows differences |
| F4 | Can a new contributor understand the component in isolation? | Component requires reading 5+ other files first |
| F5 | Are there hardcoded lists that should be dynamic? | Agent lists all skills by name instead of referencing TRIGGER-CHECKLIST |
| F6 | Will this break if a component is renamed? | Path references use hardcoded strings instead of variables |
| F7 | Is the component documented in framework indexes? | New agent not in AGENT-RUNTIME-FLOW table; new skill not in TRIGGER-CHECKLIST |
