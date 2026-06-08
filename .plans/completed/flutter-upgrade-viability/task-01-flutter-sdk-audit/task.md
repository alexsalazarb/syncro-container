# Task: Flutter SDK Upgrade Path Audit

**Plan**: Flutter Upgrade Viability Assessment
**Phase**: 1 — Assessment
**Task ID**: task-01
**Task Path**: task-01-flutter-sdk-audit
**Depends On**: None
**JIRA**: Pendiente

## Objective

Investigar el changelog de Flutter 3.32.4 → 3.38.7 y Dart 3.8.1 → 3.10.7, identificando todos los breaking changes que afectan a la codebase del proyecto (Dart 3.x patterns, Flutter framework APIs, rendering, platform channels).

## Context

El proyecto usa Flutter 3.32.4 / Dart 3.8.1 (fvm, canal stable). La versión 3.38.7 / Dart 3.10.7 ya está instalada localmente via fvm como `global` pero con status "Need setup".

El pubspec.yaml tiene constraint `sdk: ">=3.8.0 <4.0.0"` — Dart 3.10.7 lo cumple sin cambios.

Codebase highlights relevantes para breaking changes:
- `flutter_bloc` 9.x — verificar si hay cambios en BlocBase, Cubit, BlocObserver
- `go_router` 14.x — verificar cambios en GoRouter, ShellRoute, GoRouterState
- Material 3 / Widgets — verificar deprecations en MaterialApp, ThemeData, etc.
- `flutter_test` — verificar cambios en APIs de testing

## Before You Start

- [ ] Switch to base branch y pull latest: `git switch main && git pull --rebase origin main`
- [ ] Revisar Flutter release notes para 3.33.x, 3.34.x, 3.35.x, 3.36.x, 3.37.x, 3.38.x
- [ ] Revisar Dart changelog 3.9.x y 3.10.x
- [ ] Marcar esta task `in-progress` en `status.md` antes de proceder

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `task-01-flutter-sdk-audit/status.md` | modify | Actualizar estado |
| `task-01-flutter-sdk-audit/findings.md` | create | Documento de hallazgos (crear en la carpeta de esta task) |

### Do NOT Modify
- `task-02-package-compat-audit/` — owned by task-02
- `syncro-flutter/pubspec.yaml` — ninguna task de este plan modifica código

## Implementation Steps

### Step 1: Verificar Flutter release notes

Revisar las release notes oficiales de Flutter en `flutter.dev/release/release-notes` o el CHANGELOG en GitHub (`github.com/flutter/flutter`) para las versiones 3.33.0 hasta 3.38.7.

Anotar específicamente:
- Breaking changes en framework APIs usadas en el proyecto
- Deprecations con fecha de remoción próxima
- Cambios en `flutter_test`
- Cambios en Material/Cupertino widgets
- Cambios en platform channel APIs (Android/iOS)
- Cambios en `Isolate`, `async`, o concurrencia en Dart

### Step 2: Verificar Dart changelog 3.9.x y 3.10.x

Revisar `dart.dev/guides/whats-new` para 3.9.0 y 3.10.0.

Anotar:
- Nuevas keywords o cambios de sintaxis (macros, sealed classes extensions, etc.)
- Deprecations en `dart:io`, `dart:async`, `dart:core`
- Cambios en null-safety o type system
- APIs removidas

### Step 3: Verificar fvmrc y configuración

Localizar el archivo de versión de fvm en `syncro-flutter/` (`.fvmrc` o `.flutter-version`) y documentar los cambios necesarios para apuntar a 3.38.7.

Verificar si la versión 3.38.7 requiere algún setup adicional (el status "Need setup" en fvm).

### Step 4: Documentar hallazgos

Crear `task-01-flutter-sdk-audit/findings.md` con:
- Lista de breaking changes del SDK relevantes para este proyecto
- Cada item con: descripción, impacto (alto/medio/bajo), archivo(s) afectado(s) en la codebase si se pueden identificar
- Cambios necesarios en fvmrc/flutter-version
- Conclusión: ¿el SDK upgrade es mecánico o requiere work significativo?

## Testing

- [ ] No hay código a testear en esta task (es una tarea de investigación)
- [ ] Los hallazgos en `findings.md` deben referenciar fuentes oficiales (URLs)

## Documentation / KB Updates

- [ ] Los hallazgos se documentan en `task-01-flutter-sdk-audit/findings.md` dentro del plan
- [ ] No se requieren updates al KB del proyecto en esta etapa

## Completion Criteria

- [ ] `findings.md` creado con changelog relevante documentado
- [ ] Todos los breaking changes del SDK (3.32→3.38) identificados y clasificados por impacto
- [ ] Cambios necesarios en fvmrc documentados
- [ ] Status actualizado en `status.md`
