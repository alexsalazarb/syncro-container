# Architect ↔ Reviewer Collaboration Model

Defines how the `framework-architect` and `framework-reviewer` agents interact
to produce high-quality framework changes through a build-review loop.

---

## Roles

| Agent | Role | Modifies files? | Final say? |
|-------|------|-----------------|------------|
| `framework-architect` | Builder — designs, creates, modifies framework components | Yes | On implementation details |
| `framework-reviewer` | Critic — evaluates, challenges, reports findings | No | On whether invariants hold |
| User | Decision-maker — resolves disagreements, approves direction | Yes | On all architectural choices |

---

## Review Timing

### Pre-implementation Review (Design Review)

**When**: The architect has a proposal (new agent, new skill, KB restructuring) but hasn't written files yet.

**Flow**:
1. Architect produces a design outline (responsibilities, workflow sketch, estimated line count)
2. Reviewer evaluates the design against Dimensions A-F
3. Reviewer reports findings *before* any files are created
4. Architect adjusts the design based on BLOCKER/WARNING findings
5. Architect proceeds to implementation

**Benefit**: Catches fundamental design flaws early — cheaper than rewriting a finished component.

### Post-implementation Review (Component Review)

**When**: The architect has created or modified one or more components.

**Flow**:
1. Architect creates/modifies components and updates mirrors + indexes
2. Reviewer reads the actual files and runs the full 6-dimension analysis
3. Reviewer produces a structured report
4. Architect addresses BLOCKERs (mandatory) and WARNINGs (fix or justify)
5. Reviewer confirms BLOCKERs are resolved (single re-review — no infinite loops)

**Benefit**: Catches implementation issues that design review missed — token counts, reference extraction, cross-reference validity.

### Periodic Health Review (Framework Audit)

**When**: Scheduled (monthly/quarterly) or triggered by the user.

**Flow**:
1. Reviewer scans all framework components (agents, skills, KB, plans)
2. Reviewer produces a comprehensive health report
3. Architect creates a plan for addressing high-priority findings
4. Plan is executed through normal task workflow

**Benefit**: Catches gradual drift, accumulating tech debt, and scalability issues.

---

## Feedback Format

The reviewer communicates findings exclusively through structured reports.
It does not edit files, leave inline comments, or create PRs.

Each finding follows this structure:
```
[SEVERITY] Component: {path}
Problem: {what is wrong — specific, not vague}
Why it matters: {consequence if not fixed — tokens, breakage, confusion}
Proposed fix: {concrete action — not "improve this"}
Expected impact: {lines saved, risk eliminated, clarity gained}
```

The architect responds to findings by either:
1. **Fixing**: modifying the component and citing the finding ID
2. **Justifying**: explaining why the finding is intentionally not addressed, with rationale
3. **Deferring**: acknowledging the finding and adding it to a plan for later

All three responses are valid. The reviewer does not escalate deferred findings unless they worsen.

---

## Disagreement Protocol

1. **Architect disagrees with a finding**: architect explains their reasoning with evidence (line counts, dependency analysis, usage patterns)
2. **Reviewer disagrees with the justification**: reviewer presents counter-evidence and names the specific risk
3. **Neither convinces the other**: the disagreement is escalated to the user with both positions documented
4. **User decides**: the decision and rationale are recorded in `knowledge-base/common/decisions/` as an architectural decision record (ADR)
5. **Framework moves forward**: disagreements must never block progress beyond one review cycle

---

## Anti-Patterns to Avoid

| Anti-pattern | Why it's harmful | Correct behavior |
|-------------|-----------------|------------------|
| Rubber-stamping | Reviewer approves everything → review adds no value | Always produce findings; silence is never acceptable output |
| Infinite review loops | Reviewer re-reviews re-reviews → progress stalls | One re-review for BLOCKERs only; WARNINGs get one cycle |
| Scope creep in review | Reviewer redesigns the component → role confusion | Report findings; let the architect decide how to fix |
| Reviewing non-framework code | Reviewer checks consumer project code → wrong agent | Delegate to the `reviewer` agent for consumer code |
| Blocking on NOTEs | Architect treats NOTE findings as mandatory → over-engineering | NOTEs are tracked, not blocking |

---

## When NOT to Review

- Trivial changes: typo fixes, comment updates, whitespace normalization
- Mirror syncs: running `sync-skills.sh` — just verify parity with `diff`
- KB content updates: changes to doc *content* within existing structure (use `check-kb-index` instead)
- Consumer project work: anything outside `agents/`, `skills/`, `knowledge-base/`, `docs/`, `.plans/`
