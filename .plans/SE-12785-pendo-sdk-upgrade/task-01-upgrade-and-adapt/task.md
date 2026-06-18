# task-01 — Upgrade pendo_sdk and Adapt API

## Objective

Bump `pendo_sdk` to `^3.13.1`, run the pub upgrade, and fix any compile or API breakages.

## Jira

[SE-12785](https://repairtechsolutions.atlassian.net/browse/SE-12785)

## Branch

`feature/SE-12785`

## File Ownership

| Action | File |
|---|---|
| Modify | `syncro-flutter/pubspec.yaml` |
| Modify (if needed) | `syncro-flutter/lib/core/services/pendo_service.dart` |
| Modify (if needed) | `syncro-flutter/lib/app/view/app.dart` |
| Read-only | `syncro-flutter/lib/main.dart` |
| Read-only | `syncro-flutter/lib/app/dependency/service_locator.dart` |

## Do NOT Modify

- Any file outside the `syncro-flutter/` folder
- Test files (owned by task-02)

## Steps

### 1. Switch to working branch

```bash
cd syncro-flutter
git checkout feature/SE-12785
```

### 2. Update pubspec.yaml

Change:
```yaml
pendo_sdk: ^3.7.1
```
To:
```yaml
pendo_sdk: ^3.13.1
```

### 3. Resolve dependencies

```bash
fvm flutter pub upgrade pendo_sdk
fvm flutter pub get
```

If `pub upgrade` raises version conflicts with other packages, attempt:
```bash
fvm flutter pub upgrade --major-versions pendo_sdk
```
Document any conflict in `status.md`.

### 4. Check for breaking changes

```bash
fvm flutter analyze
```

Look for errors in these files (the full Pendo surface used in this project):
- `pendo_service.dart`: `PendoSDK.setup()`, `PendoSDK.startSession()`, `PendoSDK.endSession()`, `PendoSDK.track()`, `PendoSDK.setVisitorData()`, `PendoSDK.setAccountData()`
- `app.dart`: `PendoActionListener`

> Check the [pendo_sdk CHANGELOG on pub.dev](https://pub.dev/packages/pendo_sdk/changelog) to understand what changed between 3.7.1 and 3.13.1 before making changes.

### 5. Fix any API changes

If method signatures or class names changed, update `pendo_service.dart` and/or `app.dart` accordingly. Follow the existing patterns — do not refactor unrelated code.

### 6. Final analyze pass

```bash
fvm flutter analyze
```

Must be clean (0 errors, 0 warnings introduced by this task).

## Acceptance Criteria

- `pubspec.yaml` has `pendo_sdk: ^3.13.1`
- `pubspec.lock` updated with new resolved version
- `fvm flutter analyze` passes with no new errors
- All Pendo API call sites compile successfully

## Testing

Run after completing steps:
```bash
fvm flutter test lib/core/services/
```

Pass any failures to task-02 — do not fix test files in this task.

## Notes

- `pendo_sdk` is analytics-only and initialized fire-and-forget (non-blocking). Errors in Pendo initialization do not crash the app.
- If the new SDK requires changes to Android native config (e.g. `AndroidManifest.xml`, `build.gradle`), apply them and document in `status.md`.
