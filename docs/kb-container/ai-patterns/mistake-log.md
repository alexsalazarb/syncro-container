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

## 2026-06-04 - convention

**Mistake:** Committed a bugfix (SE-12659) on an active plan branch (`plan/android-r8-minification/task-04`) instead of creating a dedicated feature branch from `develop`.

**Correction:** Independent bugfixes must always branch from `develop`, not from unrelated plan branches. The correct flow: `git switch develop && git switch -c feature/SE-XXXXX`.

**Prevention:** Before committing any fix, verify the current branch with `git branch --show-current`. If the branch name belongs to a different plan or task, stop and create a new branch from `develop` first.

**Files involved:** `syncro-flutter` — any file modified as part of a bugfix

---

## 2026-06-24 - convention

**Mistake:** Created `feature/SE-12850` branching from `main` in `syncro-flutter`. `main` only has 3 initial commits — it's not the development base.

**Correction:** `syncro-flutter` uses `develop` as the base branch for all feature work. The correct flow: `git switch develop && git pull && git switch -c feature/SE-XXXXX`.

**Prevention:** Before creating any feature branch in `syncro-flutter`, verify the current branch state with `git log --oneline -3 main` and `git log --oneline -3 develop`. If `main` has no Flutter source code, it is NOT the base branch — use `develop`.

**Files involved:** `syncro-flutter` — any new feature branch creation

---

## 2026-08-25 - testing

**Mistake:** Before running Patrol E2E tests, checked `which patrol` / `dart pub global run patrol_cli --version` and reported "patrol not found" as a blocker, without first checking the project KB for how tests are actually run.

**Correction:** This project's Patrol suite uses `patrol_finders` only (widget-tree automation, no native automation bridge) — `patrol_cli` is never invoked and never needs to be installed. Tests run via plain `fvm flutter test integration_test/patrol/features/<file>.dart --flavor qa -d <device> --dart-define=PATROL=true`, exactly as documented in `docs/kb-projects/syncro-flutter/technical/testing/patrol-integration-tests.md`.

**Prevention:** Before probing the shell for a CLI tool needed to run a stack's test suite, apply the KB-First Rule — check `docs/kb-projects/{project}/README.md` for a "how to run tests" doc before assuming tooling is missing.

**Files involved:** `docs/kb-projects/syncro-flutter/technical/testing/patrol-integration-tests.md`

---

**Last Updated**: 2026-08-25
