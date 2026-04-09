# Continuous Improvement & Research Loop

Reference document for the `framework-architect` agent.
Defines how the agent proactively discovers improvements and integrates external research.

---

## The Research Loop

The framework architect operates on a continuous improvement cycle with four phases:

```
  ┌─────────────┐
  │   OBSERVE    │ ← Detect signals from framework usage
  │              │
  └──────┬───────┘
         │
         ▼
  ┌─────────────┐
  │   RESEARCH   │ ← Explore external sources for solutions
  │              │
  └──────┬───────┘
         │
         ▼
  ┌─────────────┐
  │   PROPOSE    │ ← Draft structured improvement proposals
  │              │
  └──────┬───────┘
         │
         ▼
  ┌─────────────┐
  │   INTEGRATE  │ ← Apply approved changes, measure impact
  │              │
  └──────┬───────┘
         │
         └──────────► back to OBSERVE
```

---

## Phase 1: Observe

The architect detects improvement signals from multiple sources:

| Signal source | What to look for | Priority |
|---------------|-----------------|----------|
| `mistake-log.md` | Recurring agent errors (3+ occurrences of same pattern) | High |
| Session handoffs | "Context was too large", "Agent loaded wrong docs", "Had to repeat instructions" | High |
| Plan status files | Tasks marked `blocked` or `adapted` — indicates plan assumptions were wrong | Medium |
| KB gap backfills | Topics that required `add-kb-doc` after KB miss — systemic gaps | Medium |
| Agent definitions | Skills referenced in agents that don't exist, or dead skill references | Low |
| Framework issues/PRs | User-reported problems or feature requests | High |

**Trigger**: Run observation when:
- `/framework-builder audit` is invoked
- A session handoff mentions framework friction
- A mistake is logged 3+ times for the same pattern
- After any framework upgrade (`upgrade-framework`)

---

## Phase 2: Research

When an observation reveals a gap, research solutions before proposing changes.

### Internal research (always do first)
1. Check if another framework component already solves this (deduplication check)
2. Read `knowledge-base/common/` for prior architectural decisions
3. Read completed plans in `.plans/completed/` for historical context on past attempts

### External research (when internal is insufficient)
Use WebSearch/WebFetch to explore:

| Research topic | Search queries |
|----------------|---------------|
| Agent architecture patterns | "AI agent framework architecture {year}", "multi-agent orchestration patterns" |
| Prompt optimization | "LLM prompt engineering efficiency {year}", "reducing token usage in agent systems" |
| KB/RAG strategies | "knowledge base retrieval strategies for AI agents", "tiered document loading patterns" |
| Developer workflow | "AI-assisted development workflow {year}", "developer experience with AI coding agents" |
| Plan/task systems | "task decomposition patterns for AI agents", "dependency-aware task scheduling" |

**Research rules**:
- Prefer official documentation and academic sources over blog posts
- Look for concrete implementations, not theoretical frameworks
- Note specific techniques with measurable improvements (e.g. "chunked retrieval reduced context by 40%")
- Always check publication date — prefer sources from the last 12 months

---

## Phase 3: Propose

Structure every improvement proposal as a decision document:

```markdown
## Proposal: {Title}

**Signal**: {What observation triggered this}
**Current state**: {How the framework handles this today}
**Problem**: {Why the current approach is suboptimal — with evidence}

### Option A: {Name}
- Approach: {description}
- Token impact: {+/- estimate}
- Migration effort: {low/medium/high}
- Risk: {what could go wrong}

### Option B: {Name}
- Approach: {description}
- Token impact: {+/- estimate}
- Migration effort: {low/medium/high}
- Risk: {what could go wrong}

### Recommendation
{Which option and why. Include tradeoff rationale.}

### Implementation plan
1. {Step 1}
2. {Step 2}
...
```

**Proposal routing**:
- Small change (1-2 files, no architectural impact) → implement directly after user approval
- Medium change (3-10 files, localized impact) → create a plan via `/create-plan`
- Large change (architectural, cross-component) → create a plan, discuss with user, execute incrementally

---

## Phase 4: Integrate

After approval, apply the change using the appropriate framework-builder mode:

1. Execute the implementation (Mode A/B/C/D from the skill)
2. Update all mirrors
3. Run `check-agent-drift` to verify no unintended side effects
4. Measure the actual token impact (before/after line counts)
5. Document the decision in `knowledge-base/common/` if it represents an architectural choice
6. Update `AGENT-RUNTIME-FLOW.md` if agent behavior changed

---

## Proactive Improvement Areas

The architect should periodically review these areas even when no explicit signal triggers it:

### Architecture
- Are agent boundaries still clean? (no role overlap, no orphaned responsibilities)
- Is the 3-layer KB model still sufficient? (or does a new layer/dimension make sense)
- Are plan templates capturing all necessary context? (or are agents frequently adapting)

### Context Handling
- Are compact variants up to date with their full-doc counterparts?
- Are `_index.md` triggers precise enough? (low false-positive rate)
- Could any agent definition be shortened without losing important instructions?

### Token Efficiency
- Which skills are the longest? Could they be split or have logic extracted?
- Are there skills that are rarely used? (candidates for deprecation)
- Are reference templates being used effectively? (or are skills still inlining large blocks)

### Developer Workflows
- Are trigger phrases aligned with how developers actually speak? (check session logs)
- Are automatic triggers firing correctly? (check mistake logs for "forgot to run X")
- Is the plan → task → branch → commit → PR flow smooth? (check plan status files for bottlenecks)

---

## Research Cadence

| Activity | Frequency | Trigger |
|----------|-----------|---------|
| Mistake log review | Every session (automatic) | Session start |
| KB gap analysis | Weekly or on `/framework-builder audit` | Manual or scheduled |
| External research | When internal research is insufficient | Observation phase |
| Full framework audit | Monthly or after major changes | Manual or post-upgrade |
| Architecture review | Quarterly or when adding new agent/skill types | Structural change detected |
