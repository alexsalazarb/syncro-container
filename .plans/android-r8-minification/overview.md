# Android R8 Minification + Deobfuscation File

## Summary

| Field | Value |
|-------|-------|
| Type | Type 2 — Technical |
| Project | syncro-flutter |
| Status | backlog |
| Target Demo | TBD |
| Dev | Unassigned |
| QA | Unassigned |
| Master Plan | None |

## Objective

Resolver el warning de Play Console: _"There is no deobfuscation file associated with this App Bundle."_

Actualmente `minifyEnabled false` en `build.gradle` porque R8/ProGuard causaba crashes inmediatos en Play Store internal testing (ver historial en `proguard-rules.pro`). El objetivo es re-habilitar R8 de forma segura identificando las reglas faltantes, validar extensamente antes de producción, y configurar el upload del `mapping.xml`.

## Context

- `syncro-flutter/android/app/build.gradle` — `minifyEnabled false`, `shrinkResources false`
- `syncro-flutter/android/app/proguard-rules.pro` — reglas extensas ya escritas (Flutter, Pendo, Firebase, Fragments, threading, etc.)
- Los crashes anteriores ocurrieron en **Play Store internal testing**, no en debug ni release local
- El warning es opcional (Play Console lo muestra pero el AAB se sube igual)

## Kill Criteria

1. Crashes en Play Store internal testing que no tienen fix conocido en ProGuard → revertir a `minifyEnabled false`
2. El tamaño del AAB con R8 supera el del actual en más de 5MB (indicaría que las reglas están keeping demasiado)
3. Pendo SDK deja de funcionar en producción con R8 activo

## Phases

### Phase 1 — Investigation
- `task-01-local-r8-investigation` — Habilitar R8 localmente, instalar en device real, identificar crashes

### Phase 2 — Fix
- `task-02-proguard-rules-fix` — Agregar reglas faltantes basadas en task-01

### Phase 3 — Validation
- `task-03-play-store-validation` — Subir a internal testing, verificar que no hay crashes

### Phase 4 — Release Integration
- `task-04-mapping-upload` — Configurar upload automático de `mapping.xml` en el release process

## Dependency Graph

```
task-01 → task-02 → task-03 → task-04
```

Todas secuenciales — cada fase depende de la anterior.

## Branch Convention

`plan/android-r8-minification/{task-id}`

## Files Affected

- `syncro-flutter/android/app/build.gradle`
- `syncro-flutter/android/app/proguard-rules.pro`
- `.claude/skills/syncro-create-release/SKILL.md` (agregar paso de mapping upload)
