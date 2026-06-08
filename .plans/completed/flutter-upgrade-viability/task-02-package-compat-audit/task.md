# Task: Package Compatibility Audit

**Plan**: Flutter Upgrade Viability Assessment
**Phase**: 1 — Assessment
**Task ID**: task-02
**Task Path**: task-02-package-compat-audit
**Depends On**: None
**JIRA**: Pendiente

## Objective

Auditar todos los packages en `syncro-flutter/pubspec.yaml` para determinar su compatibilidad con Flutter 3.38.7 / Dart 3.10.7, categorizar el esfuerzo de upgrade para cada uno, e identificar bloqueadores.

## Context

Estado actual de packages (resultado de `flutter pub outdated` al 2026-05-29):

**Packages con MAJOR version bump (breaking changes probables):**
| Package | Actual | Resolvable | Latest |
|---------|--------|------------|--------|
| firebase_core | 3.11.0 | 4.9.0 | 4.9.0 |
| firebase_analytics | 11.4.2 | 12.4.1 | 12.4.1 |
| firebase_crashlytics | 4.3.2 | 5.2.2 | 5.2.2 |
| firebase_messaging | 15.2.2 | 16.2.2 | 16.2.2 |
| firebase_remote_config | 5.4.0 | 6.5.1 | 6.5.1 |
| go_router | 14.8.0 | 17.2.3 | 17.2.3 |
| get_it | 8.0.3 | 9.2.1 | 9.2.1 |
| flutter_local_notifications | 18.0.1 | 21.0.0 | 21.0.0 |
| file_picker | 8.3.7 | 11.0.2 | 11.0.2 |
| device_info_plus | 11.3.0 | 13.1.0 | 13.1.0 |
| flutter_secure_storage | 9.2.4 | 10.3.1 | 10.3.1 |
| flutter_foreground_task | 8.17.0 | 9.2.2 | 9.2.2 |
| flutter_timezone | 4.1.0 | 5.1.0 | 5.1.0 |
| connectivity_plus | 6.1.4 | 7.1.1 | 7.1.1 |
| local_auth | 2.3.0 | 2.3.0 | 3.0.1 |
| package_info_plus | 8.2.1 | 9.0.1 | 10.1.0 |
| permission_handler | 11.3.1 | 12.0.2 | 12.0.2 |
| app_links | 6.3.3 | 6.4.1 | 7.1.1 |
| infinite_scroll_pagination | 4.1.0 | 5.1.1 | 5.1.1 |
| flutter_dotenv | 5.2.1 | 6.0.1 | 6.0.1 |
| timezone | 0.9.4 | 0.10.1 | 0.11.0 |
| intl | 0.19.0 | 0.20.2 | 0.20.2 |

**Packages especiales / riesgo alto:**
- `flutter_mentions`: git fork propio (`github.com/alexsalazarb/flutter_mentions`, ref: `syncro_update`) — RIESGO MÁXIMO, sin versión en pub.dev
- `webview_flutter_wkwebview`: pinneado a 3.16.0 por bug conocido de login (comentario en pubspec.yaml: `#3.18.1 make login fails`)
- `local_auth`: constraint `any` (en lugar de versión semver)
- `scribble`: 0.10.0+1 — verificar si sigue mantenido

**Packages transitive discontinuados:**
- `js` — discontinuado (transitive dep)
- `build_resolvers` — discontinuado (transitive dep)
- `build_runner_core` — discontinuado (transitive dev dep)

## Before You Start

- [ ] Switch a base branch y pull latest: `git switch main && git pull --rebase origin main`
- [ ] Leer `overview.md` del plan para contexto completo
- [ ] Marcar esta task `in-progress` en `status.md` antes de proceder

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `task-02-package-compat-audit/status.md` | modify | Actualizar estado |
| `task-02-package-compat-audit/findings.md` | create | Documento de hallazgos detallado |

### Do NOT Modify
- `task-01-flutter-sdk-audit/` — owned by task-01
- `syncro-flutter/pubspec.yaml` — ninguna task de este plan modifica código

## Implementation Steps

### Step 1: Verificar compatibilidad de packages críticos con Flutter 3.38.7

Para cada package con major version bump, consultar pub.dev y verificar:
1. ¿Soporta Dart 3.10.x?
2. ¿La versión "Resolvable" es compatible con Flutter 3.38.7?
3. ¿Cuáles son los breaking changes entre la versión actual y la resolvable?

Prioridad de investigación:
1. **firebase_*** (5 packages, todos con major bump) — revisar FlutterFire migration guide
2. **go_router** (14→17, 3 major versions) — revisar migration guide oficial
3. **get_it** (8→9) — revisar changelog
4. **flutter_local_notifications** (18→21) — revisar migration guide
5. Resto de packages con major bump

### Step 2: Investigar flutter_mentions custom fork

Verificar:
1. Navegar a `github.com/alexsalazarb/flutter_mentions`, branch `syncro_update`
2. ¿Cuál es el último commit? ¿Tiene actividad reciente?
3. ¿El código usa Dart APIs que serán deprecadas o removidas en 3.10.x?
4. ¿Existe el package original `flutter_mentions` en pub.dev con soporte para Dart 3.10.x?
5. Opciones: actualizar el fork, migrar al package original, reemplazar por alternativa

### Step 3: Verificar webview_flutter_wkwebview

El comentario `#3.18.1 make login fails` indica que hubo un bug al actualizar.
Verificar si el bug está resuelto en versiones más recientes (actual latest: 3.25.1).
Revisar el changelog y los issues de `webview_flutter_wkwebview` en GitHub.

### Step 4: Verificar packages transitive discontinuados

Verificar los replacements recomendados para:
- `js` → `package:web` o `dart:js_interop`
- `build_resolvers` → `package:build` o alternativa
- `build_runner_core` → `package:build_runner`

Determinar si los packages que los usan como dependencias ya los reemplazaron en sus versiones más nuevas.

### Step 5: Categorizar todos los packages

Para cada package del pubspec.yaml, asignar una categoría:

- 🟢 **Green**: Compatible sin cambios o upgrade de patch/minor sin breaking changes
- 🟡 **Yellow**: Major bump disponible, breaking changes manejables (estimado < 1 día de work)
- 🔴 **Red**: Major bump con breaking changes significativos (> 1 día de work) o bloqueador

### Step 6: Documentar hallazgos

Crear `task-02-package-compat-audit/findings.md` con:
- Tabla completa de todos los packages con: versión actual, versión target, categoría, notas de breaking changes
- Sección especial para `flutter_mentions` con recomendación de acción
- Sección especial para `webview_flutter_wkwebview` con estado del bug
- Lista de packages discontinuados y acciones requeridas
- Estimación de esfuerzo total (en días) para actualizar todos los packages amarillos y rojos

## Testing

- [ ] No hay código a testear (es una tarea de investigación)
- [ ] Los hallazgos deben referenciar pub.dev, changelogs o GitHub issues como fuentes

## Documentation / KB Updates

- [ ] Hallazgos se documentan en `task-02-package-compat-audit/findings.md`
- [ ] No se requieren updates al KB del proyecto en esta etapa

## Completion Criteria

- [ ] `findings.md` creado con TODOS los packages auditados
- [ ] `flutter_mentions` custom fork evaluado con recomendación de acción
- [ ] `webview_flutter_wkwebview` bug status verificado
- [ ] Tabla de categorización (green/yellow/red) completa
- [ ] Estimación de esfuerzo de upgrade documentada
- [ ] Status actualizado en `status.md`
