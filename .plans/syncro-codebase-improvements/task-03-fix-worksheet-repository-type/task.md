# Task 03 — Fix WorksheetRepository Type Registration

**Plan**: syncro-codebase-improvements
**Priority**: 🟠 Major
**Branch**: `plan/syncro-codebase-improvements/task-03`

## Problem

In `app_repositories.dart`, `WorksheetRepository` is registered under the wrong type:

```dart
RepositoryProvider<WorksheetRepositoryImpl>(  // ← wrong: concrete type as key
  create: (_) => WorksheetRepositoryImpl(
    networkService: getIt<NetworkService>(),
  ),
)
```

Any code that calls `context.read<WorksheetRepository>()` (the interface) will throw a `ProviderNotFoundException` at runtime because `WorksheetRepository` (abstract) was never registered — only `WorksheetRepositoryImpl` (concrete) was.

## Objective

Fix the registration to use the interface type as the key AND delegate to GetIt (since it's already registered there as a singleton).

## Implementation Steps

1. **Fix registration in `app_repositories.dart`**:
   ```dart
   // Before:
   RepositoryProvider<WorksheetRepositoryImpl>(
     create: (_) => WorksheetRepositoryImpl(networkService: getIt<NetworkService>()),
   )
   
   // After:
   RepositoryProvider<WorksheetRepository>(
     create: (_) => getIt<WorksheetRepository>(),  // delegate to GetIt singleton
   )
   ```

2. **Search for `WorksheetRepositoryImpl` usages** in widget tree — any `context.read<WorksheetRepositoryImpl>()` should be changed to `context.read<WorksheetRepository>()`.

3. **Test**: Run `flutter test` to ensure no regressions.

## Files to Modify

- `lib/app/dependency/app_repositories.dart`
- Any file using `context.read<WorksheetRepositoryImpl>()` (search codebase)

## Acceptance Criteria

- [ ] `context.read<WorksheetRepository>()` succeeds without throwing
- [ ] The same `WorksheetRepository` instance is returned from both `context.read()` and `getIt()`
- [ ] No usages of `context.read<WorksheetRepositoryImpl>()`
- [ ] `flutter test` passes
