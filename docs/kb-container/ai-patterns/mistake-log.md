# Mistake Log

This document records patterns of errors that agents have made, to prevent repetition.

## How to Use

1. **Session start**: Scan this document for patterns to avoid
2. **When corrected**: Add new entry using the `log-mistake` command
3. **Pattern threshold**: If an error appears 3+ times, add it to AGENTS.md "Things to Avoid"

---

## Log Entries

<!-- Entries are added automatically by the log-mistake command -->
<!-- Format:
### [Category] - Brief Description
**Date**: YYYY-MM-DD
**Error**: What happened
**Correct approach**: What should have been done
**Prevention**: How to avoid in future
-->

---

## 2026-04-13 - config

**Mistake:** Used `flutter` CLI directly instead of `fvm flutter` for all Flutter commands (test, analyze, build, run, pub get).

**Correction:** Always prefix Flutter commands with `fvm` when the project has an `.fvmrc` file (e.g. `fvm flutter test`, `fvm flutter analyze`, `fvm flutter pub get`). The `.fvmrc` at `syncro-flutter/.fvmrc` pins Flutter `3.32.4`.

**Prevention:** At session start, check for `syncro-flutter/.fvmrc` or `syncro-flutter/.fvm/fvm_config.json`. If present, use `fvm flutter` for all commands in that subproject.

**Files involved:** All shell commands running Flutter in `syncro-flutter/`

---

**Last Updated**: 2026-04-13
