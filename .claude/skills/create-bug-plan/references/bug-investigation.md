# Investigation: [BUG TITLE]

**Date**: [DATE]
**Investigator**: [NAME or "AI-assisted"]
**Ticket**: [TICKET-KEY or "N/A"]
**Status**: confirmed | disproved | partial

## Severity Assessment

**Severity**: P0 | P1 | P2 | P3
**Blast Radius**: [N users / N tenants, or "unknown"]
**Data Corruption**: Yes / No — [If yes, what data is affected and how]
**Workaround Available**: Yes / No — [If yes, describe]
**Justification**: [1-2 sentences explaining the severity classification]

## Original Hypothesis

[What the ticket/reporter believed was the cause]

## Investigation Summary

[2-3 sentence summary of what was found. State whether the original hypothesis was confirmed, partially correct, or wrong.]

## Findings

### Finding 1: [Title]

**File**: `[path]` (lines [N-M])
**Severity**: Critical | High | Medium | Low

[Explanation of what this code does, what it should do, and why it's broken]

```[language]
// Relevant code snippet
```

### Finding 2: [Title]

**File**: `[path]` (lines [N-M])
**Severity**: Critical | High | Medium | Low

[Explanation]

## Root Cause

**Root Cause Confidence**: High | Medium | Low
<!-- High (>85%): Code path confirmed, evidence aligns
     Medium (50-85%): Likely cause, some uncertainty remains
     Low (<50%): Hypothesis disproved or competing causes — DO NOT create tasks -->

[Clear, definitive statement of the root cause. Reference the specific finding above.]

### What Would Raise Confidence
<!-- If Medium or Low, list specific information needed. Delete this section if High. -->
- [e.g., "Production logs during the incident window"]
- [e.g., "Access to {service} to verify consumer behavior"]

## Origin Analysis

<!-- MANDATORY: Trace how the bug got into the codebase. Use git blame / git log. -->

### How It Was Introduced

**Commit**: [hash] ([date])
**Related Ticket**: [TICKET-KEY or "N/A"]
**PR**: [PR link or "N/A"]
**Domain Expert**: [name — person with the most context on this area, useful to consult]
<!-- Name for consultation, not as the cause of the bug. -->

### Classification

<!-- Pick one. Use neutral, system-focused language. -->
- [ ] **Side effect of a prior change** — A previous change solved one problem and introduced this as a consequence
  - Related ticket: [TICKET-KEY]
  - What the change was addressing: [description]
  - How this bug resulted: [explanation]
- [ ] **Coverage gap** — A scenario not anticipated when the feature was built
  - Feature ticket: [TICKET-KEY]
  - Missing case or edge case: [description]
- [ ] **Evolved from a refactor** — Code behavior changed during a refactor or merge
  - Related commit: [hash]
  - What changed: [description]
- [ ] **Pre-existing gap** — This area predates the feature or structured tracking
  - Earliest relevant commit: [hash or "unknown"]
- [ ] **Configuration/data issue** — Code is correct; configuration or data is unexpected
  - What needs adjustment: [description]

### How This Happened

<!-- Detailed explanation. Focus on the system, not individuals.
     Good: "The fix for {ticket} correctly addressed {problem}, and as part of that
     change, {side effect} was introduced."
     Avoid: "The developer added an unnecessary bypass." -->

[Explanation]

## Feature History

<!-- MANDATORY (skip for P0): Trace the history of the feature area this bug affects. -->

### Original Requirement

**Ticket**: [TICKET-KEY or "unknown"]
**Date**: [date]
**Requirement**: [Quote or paraphrase the original spec]
**Still valid?**: Yes / No / Modified by [TICKET-KEY]

### Timeline

| Date | Ticket/Plan | What | Status |
|------|-------------|------|--------|
| [date] | [TICKET-KEY] | [What happened — requirement, implementation, fix, workaround] | [status] |

### Related Development Plans

<!-- Check {PLANS_DIR}/ for plans that implemented or modified this feature area. -->

| Plan Slug | Location | Status | Relevance |
|-----------|----------|--------|-----------|
| [slug] | {PLANS_DIR}/{slug}/ | active/completed | [What it reveals] |
<!-- Use `git log --all -- '.plans/'` to find deleted/archived plans -->
<!-- If no plans exist for this area, state: "No development plans found" -->

### Key Takeaways

- [Takeaway 1: what the timeline reveals that affects our fix approach]
- [Takeaway 2]

## Test Coverage Gap Analysis

<!-- MANDATORY: Explain why existing tests didn't catch this bug. -->

### Existing Tests Found

| Test File | What It Tests | Covers This Bug? |
|-----------|--------------|-----------------|
| [path] | [description] | Yes / No / Partial |
<!-- If no tests exist, state: "No existing tests found for [area]" -->

### Why Tests Didn't Catch It

- [ ] **No test coverage** — No tests exist for this code path
- [ ] **Tests use mocks that hide real behavior** — [which mocks, what they hide]
- [ ] **Test data doesn't trigger the edge case** — [what data would be needed]
- [ ] **Tests assert the wrong thing** — [what they assert vs. what they should]
- [ ] **Happy path only** — [what unhappy/edge path is missing]
- [ ] **Integration test gap** — Unit tests pass but the bug is in component interaction
- [ ] **Test was disabled/skipped** — [why]

### Recommended Regression Test

[Specific description: what scenario to set up, what action triggers the bug, what the expected result should be, why this test would have caught the bug.]

## Affected Systems

| System | How it's affected |
|--------|------------------|
| [System 1] | [Impact description] |

## Proposed Fix

[Brief description of the fix approach and why it's the right one]

## Risks

- [Risk 1: what could go wrong with the fix]
- [Risk 2: regression concern]
