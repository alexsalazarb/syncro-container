# Status: SDK Deprecation Sweep

**Current Status**: complete
**Last Updated**: 2026-06-08
**Agent**: implementer
**Branch**: plan/flutter-upgrade-3-44/phase-4/task-08-sdk-deprecation-sweep
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-06-08 | not-started | — | Task created |
| 2026-06-08 | complete | implementer | All deprecations fixed; flutter analyze on target files: no issues |

## Blockers

None

## Artifacts

- commit: df619aff — SE-12530: SDK deprecation sweep

## Adaptations

- SemanticsService.announce to sendAnnouncement requires a FlutterView as first argument; obtained via View.of(context) at each call site
- InputDecorationTheme renamed to InputDecorationThemeData in Flutter 3.38+; updated local variable, abstract getter, and both M2/M3 implementations in time_picker.dart
- Radio groupValue/onChanged deprecated in favour of RadioGroup ancestor; wrapped Radio in RadioGroup in ThemeOptionWidget
