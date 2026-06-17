# Plan: Flutter Upgrade 3.32.4 → 3.44.1

**Status**: complete
**Created**: 2026-06-08
**Last Updated**: 2026-06-17
**Completed**: 2026-06-17
**Estimated Demo Date**: TBD
**Assigned Dev**: Alex Salazar
**Assigned QA**: Unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: main

## Objective

Ejecutar el upgrade de Flutter 3.32.4 / Dart 3.8.1 a Flutter 3.44.1 / Dart 3.12.1 en el proyecto syncro-flutter. Incluye actualización del SDK (fvm), migración de packages con breaking changes, y sweep de deprecaciones del SDK.

## Type

Type 2 — Technical (version upgrade)

## JIRA

[SE-12530](https://syncrotech.atlassian.net/browse/SE-12530)

## Context

- **Viabilidad**: Evaluada en plan `flutter-upgrade-viability` (CONDITIONAL GO)
- **Bloqueador resuelto**: `flutter_mentions` fork actualizado a Dart 3 (sdk: `>=3.0.0 <4.0.0`, commit 5a1bbc9 en branch `syncro_update`)
- **flutter_mentions repo**: `/Users/alex/Development/flutter_mentions` (sibling del container)
- **56 packages auditados**: 29 🟢 green, 24 🟡 yellow, 3 🔴 red
- **Esfuerzo estimado**: 8–14 días de engineering
- **Flutter actual**: 3.32.4 / Dart 3.8.1 → target: 3.44.1 / Dart 3.12.1

## Kill Criteria

1. `flutter_secure_storage 10.x` causa pérdida de datos en producción y `migrateOnAlgorithmChange: true` no es suficiente — revertir upgrade de ese package.
2. UISceneDelegate rompe Pendo deep links o el screen-blur pattern de seguridad y no hay forma de repararlo sin API de Pendo — escalar al equipo de Pendo.
3. `scribble 0.10.0+1` no resuelve bajo Dart 3.10 y el feature de firma/dibujo no tiene reemplazo viable en el sprint — extraer scribble a una branch separada y continuar el upgrade sin él.

## Phases

| Phase | Name | Tasks | Description |
|-------|------|-------|-------------|
| 1 | Foundation | task-01 | SDK bump + pubspec update + analyze baseline |
| 2 | API Rewrites | task-02, task-03 | Migrate packages con breaking API changes (parallel) |
| 3 | Platform + Services | task-04, task-05 | UISceneDelegate iOS + Firebase lock-step |
| 4 | Routing + Yellow | task-06, task-07, task-08 | go_router + yellow packages + deprecation sweep (parallel) |
| 5 | Verification | task-09 | Compat check + test suite completa |

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-sdk-baseline | SDK Baseline: fvm bump + pubspec + analyze | complete | — |
| phase-2/task-02-flutter-local-notifications | flutter_local_notifications 18→21 | complete | task-01 |
| phase-2/task-03-infinite-scroll-pagination | infinite_scroll_pagination 4→5 | complete | task-01 |
| phase-3/task-04-uiscenedelegate-ios | UISceneDelegate iOS Migration | complete | task-01 |
| phase-3/task-05-firebase-migration | Firebase 5-package Lock-step | complete | task-01, task-02 |
| phase-4/task-06-go-router | go_router 14→17 | complete | task-01 |
| phase-4/task-07-yellow-packages | Yellow Packages: service layer | complete | task-01 |
| phase-4/task-08-sdk-deprecation-sweep | SDK Deprecation Sweep | complete | task-01 |
| phase-4/task-10-icondata-kotlin | IconData final + Kotlin AGP Migration | complete | task-01 |
| phase-5/task-09-verification | Verification + Test Suite | adapted | all |

## Dependency DAG

```
task-01
  ├── task-02 ──────────────────► task-05
  ├── task-03
  ├── task-04
  ├── task-06
  ├── task-07
  ├── task-08
  └── task-10
                                      ↓
                          task-09 (depends on all)
```

## Branch Convention

Pattern: `plan/flutter-upgrade-3-44/{task-path}`

Examples:
- `plan/flutter-upgrade-3-44/task-01-sdk-baseline`
- `plan/flutter-upgrade-3-44/phase-2/task-02-flutter-local-notifications`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/pubspec.yaml` | Todos los version constraints — sólo task-01 lo modifica |
| `syncro-flutter/.fvmrc` | Flutter version — sólo task-01 lo modifica |
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | flutter_local_notifications + Firebase messaging |
| `syncro-flutter/ios/Runner/AppDelegate.swift` | UISceneDelegate migration |
| `syncro-flutter/lib/core/routing/app_router.dart` | go_router routes |
| `syncro-flutter/lib/app/dependency/service_locator.dart` | get_it registrations |

## Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| UISceneDelegate rompe Pendo deep links | HIGH | Probar deep links en device físico antes de cerrar task-04 |
| flutter_secure_storage 10.x data loss | HIGH | Usar `migrateOnAlgorithmChange: true`, probar en device con data existente |
| IconData `final` — 5 files need manual inspection | HIGH | Check if any file extends/implements IconData |
| scribble incompatible con Dart 3.10 | MEDIUM | Smoke test en task-09; fallback: extraer a branch separada |
| go_router URLs case-sensitive por defecto | MEDIUM | Agregar `caseSensitive: false` en GoRouter() si hay deep links con mayúsculas |
| hive resolución incierta bajo Dart 3.10 | MEDIUM | Verificar con `flutter pub get` en task-09 |
| Kotlin AGP 9 migration | MEDIUM | Remove id "kotlin-android", replace kotlinOptions |
| Material color token algorithm change | MEDIUM | ColorScheme.fromSeed() produces different colors for 4 tokens — QA visual regression |
| FontWeight variable font axis (Lexend) | MEDIUM | Audit TextStyle with fontWeight + FontVariation in Lexend files |

## Success Criteria

- [ ] `fvm flutter analyze` sin errores (warnings aceptables)
- [ ] `fvm flutter test` sin regresiones
- [ ] App corre en iOS físico y Android: login, navegación, push notifications, timer
- [ ] `flutter_mentions` (@ mentions en chat) funciona en device
- [ ] Pendo deep links funcionan post-UISceneDelegate
- [ ] flutter_secure_storage no pierde datos existentes en device con sesión activa

## References

- **JIRA**: [SE-12530](https://syncrotech.atlassian.net/browse/SE-12530)
- **Viability Report**: `.plans/completed/flutter-upgrade-viability/task-03-upgrade-decision-report/report.md`
- **flutter_mentions fork**: `github.com/alexsalazarb/flutter_mentions` branch `syncro_update`
