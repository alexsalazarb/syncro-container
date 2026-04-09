# Task 02 — Fix DI Double-Registration

**Plan**: syncro-codebase-improvements
**Priority**: 🟠 Major
**Branch**: `plan/syncro-codebase-improvements/task-02`

## Problem

Several repositories are registered in both `service_locator.dart` (GetIt) and `app_repositories.dart` (widget tree), creating two separate instances:

| Repository | GetIt | Widget Tree |
|-----------|-------|-------------|
| `AssetsRepository` | ✅ `AssetsRepositoryImpl` | ✅ `AssetsRepositoryImpl` (new instance) |
| `TimeClockRepository` | ✅ | ✅ registered twice in `AppProviders` |

This means `getIt<AssetsRepository>()` and `context.read<AssetsRepository>()` return different instances. Mutations from cubit A won't be visible to cubit B.

## Objective

Remove duplicate registrations. Each repository should have exactly one canonical source.

## Decision: Which tier wins?

**Rule**: GetIt wins for repositories that are referenced by GetIt-registered cubits (e.g., `AssetsCubit` in GetIt uses `AssetsRepository` from GetIt). Widget-tree `RepositoryProvider` should just expose the GetIt instance:

```dart
// In app_repositories.dart — delegate to GetIt instead of creating new instance
RepositoryProvider<AssetsRepository>(
  create: (_) => getIt<AssetsRepository>(),  // ← expose the singleton
)
```

## Implementation Steps

1. **Audit double-registrations**: Confirm all repositories that are in both tiers.
   - `AssetsRepository` — in GetIt AND widget tree (creates new instance)
   - `TimeClockRepository` — in GetIt AND widget tree (two `AppProviders` entries)

2. **Fix `AssetsRepository` in `app_repositories.dart`**:
   - Change `create: (_) => AssetsRepositoryImpl(...)` to `create: (_) => getIt<AssetsRepository>()`

3. **Fix `TimeClockRepository` in `app_providers.dart`**:
   - There are two entries for `TimeClockRepository` — remove the duplicate
   - Confirm which cubit needs it and keep one `RepositoryProvider` that delegates to GetIt

4. **Audit `app_providers.dart` for duplicate BlocProviders** too (e.g., `AssetFilterCubit` is in both GetIt and widget tree)

5. **Test**: Run `flutter test` to ensure no regressions

## Files to Modify

- `lib/app/dependency/app_repositories.dart`
- `lib/app/dependency/app_providers.dart`

## Acceptance Criteria

- [ ] `getIt<AssetsRepository>()` and `context.read<AssetsRepository>()` return the same instance
- [ ] `TimeClockRepository` is registered exactly once in the widget tree
- [ ] No duplicate `BlocProvider` registrations
- [ ] `flutter test` passes
