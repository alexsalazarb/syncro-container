# task-02 — Verify Build, Tests, and Analytics

## Objective

Confirm the upgraded SDK compiles on Android, all tests pass, and Pendo analytics still work end-to-end.

## Jira

[SE-12785](https://repairtechsolutions.atlassian.net/browse/SE-12785)

## Branch

`feature/SE-12785`

## Depends on

`task-01` must be complete and `fvm flutter analyze` must be clean.

## File Ownership

| Action | File |
|---|---|
| Modify (if needed) | `syncro-flutter/test/features/general/repositories_impl_test.mocks.dart` |
| Modify (if needed) | `syncro-flutter/test/features/ticket/core/infrastructure/ticket_repository_impl_test.mocks.dart` |

> Only regenerate mocks if task-01 changed the Pendo service interface.

## Do NOT Modify

- Any source file in `lib/` (owned by task-01)
- `pubspec.yaml` (owned by task-01)

## Steps

### 1. Run full test suite

```bash
cd syncro-flutter
fvm flutter test
```

All tests must pass. If failures are unrelated to Pendo (pre-existing), document in `status.md` — do not fix them here.

### 2. Regenerate mocks (only if needed)

If task-01 changed any public interface of `PendoService`, regenerate mocks:

```bash
fvm flutter pub run build_runner build --delete-conflicting-outputs
fvm flutter test
```

### 3. Android debug build

```bash
fvm flutter build apk --debug --flavor qa
```

Build must succeed with no Pendo-related errors.

### 4. Manual smoke test (on Android device or emulator)

- Launch the app (QA flavor)
- Log in with a valid account
- Open Logcat / device logs and verify Pendo log lines:
  - `✅ PendoService initialized successfully`
  - `🚀 Pendo authenticated session started for user: ...`
- Navigate to at least 2 screens and verify no Pendo-related crashes in logs

### 5. Lint final check

```bash
fvm flutter analyze
```

## Acceptance Criteria

- `fvm flutter test` passes (0 failures introduced by this upgrade)
- Android debug build succeeds
- Pendo session logs appear correctly on device
- No Pendo-related crashes in logcat

## Notes

- Pendo is fire-and-forget: if `PendoService.init()` throws, it is caught and logged — the app continues. A caught exception in logs is NOT a blocker as long as it's handled.
- iOS does not require verification — this issue is Android-only.
