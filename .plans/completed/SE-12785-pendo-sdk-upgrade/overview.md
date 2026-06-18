# SE-12785 — Pendo SDK Upgrade

## Summary

Upgrade `pendo_sdk` from `^3.7.1` to `^3.13.1` to comply with Google's Certificate Transparency (CT) enforcement. Android apps using non-compliant SDKs will be rejected by Play Store after **July 1, 2026**.

| Field | Value |
|---|---|
| **Status** | complete |
| **Last Updated** | 2026-06-18 |
| **Plan type** | Type 2 — Technical (dependency upgrade) |
| **Jira** | [SE-12785](https://repairtechsolutions.atlassian.net/browse/SE-12785) |
| **Project** | syncro-flutter |
| **Branch** | `feature/SE-12785` |
| **Target** | Before July 1, 2026 |
| **Developer** | Unassigned |
| **QA** | Unassigned |
| **Master Plan** | None |

## Context

- Reference: [Pendo Mobile SDK Issue #327](https://github.com/pendo-io/pendo-mobile-sdk/issues/327)
- Current: `pendo_sdk: ^3.7.1` in `syncro-flutter/pubspec.yaml`
- Required minimum: `3.13.1`
- Affects Android only; iOS is unaffected

## Affected Files

| File | Role |
|---|---|
| `syncro-flutter/pubspec.yaml` | Dependency version |
| `syncro-flutter/lib/core/services/pendo_service.dart` | Main Pendo integration |
| `syncro-flutter/lib/app/view/app.dart` | `PendoActionListener` widget usage |
| `syncro-flutter/lib/main.dart` | PendoService initialization |
| `syncro-flutter/lib/app/dependency/service_locator.dart` | Singleton registration |

## Phases & Tasks

### Phase 1 — Upgrade

| ID | Task | Status |
|---|---|---|
| task-01 | Upgrade dependency and adapt API | complete |

### Phase 2 — Verification

| ID | Task | Status | Depends on |
|---|---|---|---|
| task-02 | Verify build, tests, and analytics | complete | task-01 |

## Dependency Graph

```
task-01 (upgrade) → task-02 (verify)
```

## Kill Criteria

1. `pendo_sdk 3.13.x` introduces API breaking changes that cannot be resolved without Pendo support
2. Android build fails after upgrade and no fix is available without SDK downgrade
3. Task-01 complete but `flutter test` shows regressions unrelated to Pendo (indicates deeper conflicts)

## Branch Convention

```
feature/SE-12785           ← working branch (all tasks)
```

> Tasks are small and sequential — no parallel execution needed. Work directly on `feature/SE-12785`.
