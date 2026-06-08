# Status: Exclude RUNNING_TIMER_ENTRY from Android Auto-Backup

**Current Status**: complete
**Last Updated**: 2026-05-26
**Agent**: claude-sonnet-4-6
**Branch**: feature/SE-12509
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-05-26 | not-started | — | Task created |
| 2026-05-26 | complete | claude-sonnet-4-6 | Implemented on feature/SE-12509 |

## Blockers

None

## Artifacts

- `syncro-flutter/android/app/src/main/res/xml/backup_rules.xml` — New: excludes `RUNNING_TIMER_ENTRY` from Auto-Backup (API 23–30)
- `syncro-flutter/android/app/src/main/res/xml/data_extraction_rules.xml` — New: excludes `RUNNING_TIMER_ENTRY` from cloud backup and device transfer (API 31+)
- `syncro-flutter/android/app/src/main/AndroidManifest.xml` — Changed `android:fullBackupContent="true"` → `android:fullBackupContent="@xml/backup_rules"` and added `android:dataExtractionRules="@xml/data_extraction_rules"`

## Adaptations

None — implemented exactly as specified in task.md.
