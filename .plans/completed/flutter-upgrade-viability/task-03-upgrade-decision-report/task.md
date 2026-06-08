# Task: Upgrade Decision Report

**Plan**: Flutter Upgrade Viability Assessment
**Phase**: 2 — Decision
**Task ID**: task-03
**Task Path**: task-03-upgrade-decision-report
**Depends On**: task-01-flutter-sdk-audit, task-02-package-compat-audit
**JIRA**: Pendiente

## Objective

Sintetizar los hallazgos de task-01 y task-02 en un reporte ejecutivo de viabilidad que incluya: go/no-go recommendation, estimación de esfuerzo total, orden de ejecución del upgrade si es greenlight, y plan de mitigación de riesgos.

## Context

Este task es 100% de síntesis. Depende de los `findings.md` generados por task-01 y task-02.

Inputs requeridos:
- `task-01-flutter-sdk-audit/findings.md` — breaking changes del SDK
- `task-02-package-compat-audit/findings.md` — categorización de packages (green/yellow/red)

Output:
- `task-03-upgrade-decision-report/report.md` — reporte ejecutivo final

## Before You Start

- [ ] Verificar que task-01 y task-02 están en status `complete`
- [ ] Leer `task-01-flutter-sdk-audit/findings.md` completamente
- [ ] Leer `task-02-package-compat-audit/findings.md` completamente
- [ ] Marcar esta task `in-progress` en `status.md` antes de proceder

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `task-03-upgrade-decision-report/status.md` | modify | Actualizar estado |
| `task-03-upgrade-decision-report/report.md` | create | Reporte ejecutivo final |

### Do NOT Modify
- `task-01-flutter-sdk-audit/` — owned by task-01
- `task-02-package-compat-audit/` — owned by task-02
- `syncro-flutter/pubspec.yaml` — ninguna task de este plan modifica código

## Implementation Steps

### Step 1: Sintetizar breaking changes del SDK

De task-01/findings.md, resumir:
- Breaking changes del SDK que requieren cambios en código del proyecto
- Esfuerzo estimado para cada fix
- Si hay alguno que sea bloqueador (requiere cambio arquitectural profundo)

### Step 2: Sintetizar package audit

De task-02/findings.md:
- Contar packages 🟢 / 🟡 / 🔴
- Listar packages 🔴 con su complejidad estimada
- Identificar si `flutter_mentions` es un bloqueador real
- Estado del bug de `webview_flutter_wkwebview`

### Step 3: Calcular esfuerzo total

Suma de:
- Esfuerzo SDK (task-01)
- Esfuerzo packages amarillos (task-02)
- Esfuerzo packages rojos (task-02)
- Buffer por integraciones y testing (15-20%)

Expresar en: story points, días de desarrollo, o rangos (ej. "3-5 días de engineering")

### Step 4: Definir orden de ejecución (si go)

Si la recomendación es proceder, proponer el orden lógico de upgrade:
1. fvm version bump (tarea mecánica, baja duración)
2. Firebase packages (todos juntos, migration guide de FlutterFire)
3. go_router (migration guide independiente)
4. get_it (si hay breaking changes que impacten service_locator.dart)
5. Resto de packages yellow
6. flutter_mentions (requiere work manual en el fork)
7. Test suite completa

### Step 5: Redactar el reporte

Crear `task-03-upgrade-decision-report/report.md` con las siguientes secciones:

```markdown
# Upgrade Viability Report — Flutter 3.32.4 → 3.38.7

## Executive Summary
[GO / NO-GO / CONDITIONAL GO] — 2-3 oraciones

## Upgrade Scope
- Flutter SDK: 3.32.4 → 3.38.7 (Dart 3.8.1 → 3.10.7)
- Breaking changes de SDK: N items
- Packages Green (no requieren cambios): N
- Packages Yellow (upgrade minor): N  
- Packages Red (breaking changes significativos): N
- **Bloqueadores identificados**: sí/no

## Bloqueadores
[Lista de items que bloquearían el upgrade, si los hay]

## Esfuerzo Estimado
- SDK fixes: X días
- Firebase migration: X días
- go_router migration: X días
- Otros packages: X días
- Testing y validación: X días
- **Total**: X-Y días de engineering

## Riesgos Clave
1. flutter_mentions custom fork — [status y acción recomendada]
2. webview_flutter_wkwebview login bug — [status]
3. Packages transitive discontinuados — [acción]

## Orden de Ejecución Recomendado (si go)
[Lista ordenada de pasos]

## Recomendación Final
[GO / NO-GO / CONDITIONAL GO con justificación]
```

## Testing

- [ ] No hay código a testear (es una tarea de síntesis/investigación)
- [ ] El reporte debe citar findings de task-01 y task-02 como fuentes

## Documentation / KB Updates

- [ ] El reporte se guarda en `task-03-upgrade-decision-report/report.md`
- [ ] Si el go/no-go es GO, crear una nota en `docs/kb-projects/syncro-flutter/` referenciando el plan de upgrade a crear

## Completion Criteria

- [ ] `report.md` creado con todas las secciones requeridas
- [ ] Go/no-go recommendation clara con justificación
- [ ] Estimación de esfuerzo calculada
- [ ] Orden de ejecución definido (si go)
- [ ] Bloqueadores documentados
- [ ] Status actualizado en `status.md`
