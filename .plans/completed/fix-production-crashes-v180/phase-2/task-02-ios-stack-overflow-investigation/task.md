# Task: iOS — investigate + fix repetitive FlutterError Stack Overflow

**Plan**: Fix Production Crashes — v1.8.0 (fix-production-crashes-v180)
**Phase**: 2
**Task ID**: task-02
**Task Path**: `phase-2/task-02-ios-stack-overflow-investigation`
**Depends On**: None
**JIRA**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
**Crashlytics Issue**: `0bae269c5d2442a15a183ad4642da5c7` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/0bae269c5d2442a15a183ad4642da5c7))

## Objective

Find and fix the root cause of a `FlutterError` Stack Overflow (`"Error thrown Platform error"`). This is flagged `SIGNAL_REPETITIVE` — 4 crashes per user on average across 2 impacted users, meaning it's not a one-off: something in the user's session repeatedly hits the same recursive/looping path. firstSeen 1.4.0, lastSeen 1.7.0 (not yet confirmed in 1.7.1).

## Context

**Root cause is not known at planning time.** The Crashlytics issue title (". - ...") carries no useful symbol information — this is unlike the other 5 tasks in this plan, which all had a clear code-level lead from the stack trace alone. Do not skip straight to a fix; this task starts with a mandatory investigation step.

A Stack Overflow surfaced as a `FlutterError` (rather than a native `SIGSEGV`) in Flutter usually means a Dart-level infinite recursion or an infinite widget-rebuild loop (e.g. a `Cubit`/`BlocBuilder` triggering its own state change in `build()`, or a recursive `copyWith`/`toJson` call on a self-referential model) — but this is a hypothesis to verify, not a starting assumption to build a fix on.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/fix-production-crashes-v180/phase-2/task-02-ios-stack-overflow-investigation`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## Implementation Steps

### Step 1: Investigate (mandatory, before any code change)
1. Call `mcp__firebase__crashlytics_get_issue(appId: "1:920223298498:ios:a0d84c75923c83b35b5c5b", issueId: "0bae269c5d2442a15a183ad4642da5c7")` to re-confirm the current sample event.
2. Call `mcp__firebase__crashlytics_batch_get_events` on the sample event (`bb466325fdca4f2ba7588362242a62e4_2256495312801351786`, from the 2026-08-28 report run — re-verify it's current first) to pull the full stack trace, custom keys, breadcrumbs/logs, and device/session context.
3. Call `mcp__firebase__crashlytics_get_report(appId, report: "topVariants", filter: {issueId: "0bae269c5d2442a15a183ad4642da5c7"})` to see if there are multiple distinct variants (different exact traces) grouped under this one issue — that changes whether this is one bug or several with the same generic Stack Overflow symptom.
4. Write up findings as a short `investigation.md` in this task's directory before writing any fix code (mirrors the pattern used in `.plans/SE-12379-chat-list-ios/investigation.md`).

### Step 2: Fix
Determined by Step 1's findings — cannot be scoped further until the stack trace is known. Update this task's File Ownership table (below) once the implicated file(s) are identified, and note the addition in `status.md`'s Adaptations section.

## File Ownership

To be determined by Step 1's investigation — do not guess a file here. Once identified, this section and `status.md` must both be updated before Step 2 begins.

### Do NOT Modify

- Any file already owned by task-01, task-03, task-04, task-05, or task-06 in this plan, unless the investigation proves the same root cause spans multiple tasks — if so, stop and re-scope with the user rather than silently merging tasks.

## Testing

- [ ] Once the recursive/looping path is identified, add a unit or widget test that exercises it and asserts it terminates (does not stack-overflow) — exact test type depends on where the fix lands (cubit, widget, model method).
- [ ] `fvm flutter analyze` passes
- [ ] `fvm flutter test` passes (full suite)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Once root cause is confirmed, document it via `document-solution` if it reveals a non-obvious pattern (3+ files touched or a subtle recursion bug) — this task is explicitly the kind of KB gap `document-solution` exists for, since the issue title gave no upfront lead.

## Completion Criteria

- [ ] `investigation.md` written with confirmed root cause
- [ ] Fix applied and covered by a regression test
- [ ] No new occurrence of this issue in Crashlytics after the fix ships (follow-up check, not blocking this task's completion)
