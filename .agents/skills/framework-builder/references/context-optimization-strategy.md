# Context & Token Optimization Strategy

Reference document for the `framework-builder` skill and `framework-architect` agent.
Defines concrete strategies for minimizing context consumption while maximizing output quality.

---

## Core Principle

Every token loaded into an agent's context window has a cost: latency, money, and displacement of potentially more relevant information. The framework architect treats context as a scarce resource and optimizes for **precision of retrieval** over **breadth of loading**.

---

## Strategy 1: Structured Retrieval Over Raw Context

**Problem**: Loading entire files or directories into context wastes tokens on irrelevant content.

**Solution**: Use trigger-gated, index-mediated retrieval at every layer.

| Layer | Index | Retrieval method |
|-------|-------|-----------------|
| Layer 1 (Framework KB) | `_index.md` | Match task keywords → trigger list → load `.compact.md` → escalate to full only if needed |
| Layer 2 (Container KB) | `README.md` | Scan topic headings → load only matching docs |
| Layer 3 (Project KB) | `README.md` | Same scan-and-match pattern |
| Plans | `README.md` | Read active plan count and status table — load full overview only when executing within a plan |
| Codebase | `AGENTS.md` key files | Read key files listed in AGENTS.md first — avoid broad exploration |

**Example**: An agent working on a SwiftUI task should load `swiftui-patterns.compact.md` (50 lines) — not all 19 Swift KB docs (2000+ lines).

---

## Strategy 2: Compact Variants as Default

**Problem**: Full KB docs often contain implementation detail irrelevant to the current task.

**Solution**: Every KB doc over 100 lines must have a `.compact.md` companion. Agents load compact first.

**Compact doc rules**:
- Maximum 50 lines
- Contains: overview, key patterns (as code snippets), gotchas, and links to full doc
- Omits: exhaustive examples, historical context, alternative approaches
- Trigger keywords identical to full doc (so `_index.md` routing works for both)

**When to escalate to full**: The compact doc explicitly states escalation conditions (e.g. "Read full doc when implementing custom middleware" or "Read full doc for migration procedures").

---

## Strategy 3: Conditional Sections in Skills

**Problem**: Skill instructions contain branches for container/single layout, PR integration, master plans, etc. Agents read all branches even when most don't apply.

**Solution**: Gate every conditional section with a clear skip directive.

```markdown
### Step 5a: Resolve Master Plan (If Linked)

Skip this step if `--master-plan` was not specified and the user did not indicate a master plan link.
```

**Rules for conditional sections**:
- Start with a skip condition on the first line
- Use `(If X)` or `(Conditional)` in the heading
- Keep the skip condition evaluable from information already loaded (no additional file reads to decide whether to skip)

---

## Strategy 4: Reference Extraction

**Problem**: Inline templates and large code blocks in skill instructions consume tokens even when the agent doesn't need them yet.

**Solution**: Extract to `references/` files. The skill loads them only when the step that needs them is reached.

**Extraction thresholds**:
- Template > 20 lines → extract to `references/{template-name}.md`
- Code block > 15 lines → extract to `references/{example-name}.md`
- Decision table > 10 rows → extract to `references/{table-name}.md`

**Naming convention**: `references/plan-overview.md`, `references/plan-task.md`, `references/audit-checklist.md`

---

## Strategy 5: Shared Logic in `skills/common/`

**Problem**: Multiple skills duplicate resolver logic, KB loading patterns, or git operations.

**Solution**: Extract shared logic to `skills/common/{name}.md` and reference it.

**Currently extracted**: `resolve-project-context.md` (used by 15+ skills).

**Candidates for extraction**:
- Git branch management (create, switch, push pattern used by execute-task, execute-plan)
- KB loading sequence (the 3-layer pattern repeated in every agent workflow)
- Status file updates (the status.md update pattern used by execute-task, add-defect)

**How to reference**: `> See [resolve-project-context](../common/resolve-project-context.md)` — agents follow the link and load the shared doc once.

---

## Strategy 6: Plans as Persistent Context

**Problem**: Multi-step tasks require agents to re-derive context (file ownership, dependencies, architecture decisions) for every task.

**Solution**: Plans store derived context as persistent state that agents load instead of recomputing.

- `overview.md` — shared context: objective, phases, kill criteria, cross-task decisions
- `task.md` — per-task context: file ownership, implementation steps, dependency links
- `status.md` — execution state: adaptations, blockers, completion timestamps
- `contracts/` — interface agreements between parallel tasks

An agent executing task-05 loads `overview.md` + `task-05/task.md` + dependency `status.md` files — it does not re-explore the entire codebase.

---

## Strategy 7: Decide What Is Necessary vs. Irrelevant

Before loading any document, apply this relevance filter:

1. **Does the task keyword appear in the doc's trigger list?** No → skip.
2. **Is the doc in the same layer as the task's scope?** Layer 1 for generic tech, Layer 2 for shared domain, Layer 3 for project-specific. If the doc is in the wrong layer → skip.
3. **Has the agent already loaded a doc covering this topic?** Yes → skip (no duplicate coverage).
4. **Is this a compact variant and will the task need implementation detail?** If only planning or reviewing → compact is sufficient, skip the full doc.

**Anti-patterns to avoid**:
- Loading all KB docs "just in case"
- Reading an entire directory before knowing which files are relevant
- Re-reading AGENTS.md in every skill step (read once in Step 0/1, reference in memory)
- Loading external web pages when KB already covers the topic

---

## Measuring Token Efficiency

Track these metrics when auditing or improving the framework:

| Metric | Target | How to measure |
|--------|--------|----------------|
| Agent definition length | ≤ 80 lines | `wc -l agents/*.md` |
| Skill instruction length | ≤ 150 lines | `wc -l skills/*/SKILL.md` |
| KB docs with compact variants | ≥ 80% of docs > 100 lines | Compare `*.md` vs `*.compact.md` counts |
| Average KB docs loaded per task | 2-4 | Trace a typical task through trigger matching |
| Trigger keyword overlap | ≤ 2 docs per keyword | Scan `_index.md` for duplicate triggers |
| Shared logic extraction rate | Common patterns in ≤ 1 location | Grep for duplicated instruction blocks across skills |
