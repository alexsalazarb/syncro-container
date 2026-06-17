# Status: SDK Baseline — fvm bump + pubspec + analyze

**Current Status**: complete
**Last Updated**: 2026-06-08
**Agent**: claude-sonnet
**Branch**: plan/flutter-upgrade-3-44/task-01-sdk-baseline
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-06-08 | not-started | — | Task created |
| 2026-06-08 | in-progress | claude-sonnet | fvm use 3.44.1, pubspec updated, pub get resolved |
| 2026-06-08 | complete | claude-sonnet | baseline-analysis.md created, 80+ errors in lib/ documented |

## Blockers

None

## Artifacts

- `baseline-analysis.md` — full flutter analyze output grouped by responsible task

## Adaptations

- Target bumped from 3.38.7 → 3.44.1 (user confirmed latest stable)
- `win32: ^6.0.1` added to dependency_overrides (file_picker/device_info_plus conflict, mobile-only app)
- `firebase_core_platform_interface` bumped to ^7.0.1 (required by firebase_remote_config ^6.5.1)
- `hive_generator` removed from dev_dependencies (analyzer conflict with Dart 3.12 toolchain; .g.dart files committed)
- .fvmrc uses "3.44.1" (specific version, not channel name)
