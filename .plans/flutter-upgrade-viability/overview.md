# Plan: Flutter Upgrade Viability Assessment

**Status**: not-started
**Created**: 2026-05-29
**Last Updated**: 2026-05-29
**Estimated Demo Date**: TBD
**Assigned Dev**: Alex Salazar
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: main

## Objective

Evaluar la viabilidad de actualizar Flutter de la versión actual (3.32.4 / Dart 3.8.1) a la versión más reciente disponible (3.38.7 / Dart 3.10.7), auditando los breaking changes del SDK y la compatibilidad de todos los packages del proyecto.

## Context

- **Flutter actual (fvm)**: 3.32.4, canal stable, Dart 3.8.1 (Jun 13 2025)
- **Flutter disponible más reciente (fvm)**: 3.38.7, Dart 3.10.7 (global, needs setup)
- **Dart SDK constraint**: `">=3.8.0 <4.0.0"` — Dart 3.10.7 lo satisface
- **Packages totales con major bump disponible**: 15+ (firebase_*, go_router, get_it, flutter_local_notifications, etc.)
- **Packages custom/riesgo especial**: `flutter_mentions` (git fork propio), `local_auth` (any), `webview_flutter_wkwebview` (pinned por bug conocido)
- **Packages transitive discontinuados**: `js`, `build_resolvers`, `build_runner_core`

## Scope

### In Scope
- Análisis del changelog de Flutter 3.32.4 → 3.38.7 (breaking changes de framework y Dart)
- Compatibilidad de TODOS los packages en `pubspec.yaml` con Flutter 3.38.7 / Dart 3.10.7
- Evaluación del riesgo de cada package con major version bump
- Evaluación especial de `flutter_mentions` (git fork — `github.com/alexsalazarb/flutter_mentions`, ref: `syncro_update`)
- Packages transitive discontinuados y sus replacements
- Estimación de esfuerzo y go/no-go recommendation

### Out of Scope
- La ejecución real del upgrade — eso es un plan separado si el go/no-go es positivo
- Upgrades de packages que no impacten compatibilidad con la nueva Flutter version
- Cambios en CI/CD o build pipelines

## Kill Criteria

- Si `flutter_mentions` (custom fork) es incompatible con Dart 3.10.x y el fork no tiene mantenimiento activo → bloquea el upgrade
- Si más de 3 packages críticos (firebase_*, go_router, get_it) tienen breaking changes que requieren más de 2 semanas de work → recomendar postergación
- Si la nueva versión de Flutter introduce incompatibilidades con iOS 15 o Android API 23 (min targets del proyecto)

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Assessment | task-01, task-02 | None | SDK audit + package audit (parallelizables) |
| 2 | Decision | task-03 | task-01, task-02 | Reporte de viabilidad y recomendación |

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-flutter-sdk-audit | Flutter SDK Upgrade Path Audit | not-started | — |
| task-02-package-compat-audit | Package Compatibility Audit | not-started | — |
| task-03-upgrade-decision-report | Upgrade Decision Report | not-started | task-01, task-02 |

## Branch Convention

Pattern: `plan/flutter-upgrade-viability/{task-path}`

Examples:
- `plan/flutter-upgrade-viability/task-01-flutter-sdk-audit`
- `plan/flutter-upgrade-viability/task-02-package-compat-audit`
- `plan/flutter-upgrade-viability/task-03-upgrade-decision-report`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/pubspec.yaml` | Package constraints actuales |
| `syncro-flutter/.fvmrc` o `.flutter-version` | Flutter version config |
| `syncro-flutter/pubspec.lock` | Versiones resueltas actualmente |

## Risks

- `flutter_mentions` custom git fork podría no ser compatible con Dart 3.10.x — Mitigación: verificar en task-02, contactar mantenedor si necesario
- `go_router` 14→17 tiene breaking changes conocidos en la API de routes — Mitigación: documentar scope en task-02
- `webview_flutter_wkwebview` está pinneado a 3.16.0 por bug conocido de login — Mitigación: verificar si la versión nueva resuelve o mantiene el bug
- Packages transitive discontinuados (`js`, `build_resolvers`) pueden bloquear `pub get` con la nueva SDK

## Success Criteria

- [ ] Changelog de Flutter 3.32.4 → 3.38.7 documentado con breaking changes relevantes
- [ ] Todos los packages en pubspec.yaml auditados y categorizados (green/yellow/red)
- [ ] Reporte de go/no-go producido con estimación de esfuerzo
- [ ] Bloqueadores identificados y documentados

## References

- **JIRA**: [SE-12530](https://syncrotech.atlassian.net/browse/SE-12530)
- **Related Plans**: None
