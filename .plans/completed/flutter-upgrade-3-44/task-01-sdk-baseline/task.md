# Task: SDK Baseline — fvm bump + pubspec update + analyze

**Plan**: Flutter Upgrade 3.32.4 → 3.44.1
**Phase**: 1 — Foundation
**Task ID**: task-01
**Task Path**: task-01-sdk-baseline
**Depends On**: None
**JIRA**: SE-12530

## Objective

Actualizar el Flutter SDK a stable (3.44.1 / Dart 3.12.1) via fvm, actualizar TODOS los constraints de packages en pubspec.yaml, correr `flutter pub get`, y documentar el baseline de errores de `flutter analyze` para guiar las tasks siguientes.

Esta task NO corrige errores — solo establece el baseline. Toca pubspec.yaml y .fvmrc; ninguna otra task debe modificar esos archivos.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/.fvmrc` | modify | Cambiar `"flutter": "3.32.4"` → `"flutter": "stable"` (3.44.1 / Dart 3.12.1) |
| `syncro-flutter/pubspec.yaml` | modify | Actualizar todos los constraints (ver pasos) |
| `syncro-flutter/pubspec.lock` | modify | Regenerado por `flutter pub get` |
| `task-01-sdk-baseline/baseline-analysis.md` | create | Resultado completo de `flutter analyze` |

### Do NOT Modify
- Ningún archivo en `lib/`, `ios/`, `android/` — solo configuración

## Implementation Steps

### Step 1: Actualizar fvm

```bash
cd syncro-flutter
# Cambiar .fvmrc
# De: { "flutter": "3.32.4" }
# A:  { "flutter": "stable" }
# NOTA: fvm use stable ya fue ejecutado — verificar que .fvmrc esté correcto
fvm use stable
```

Verificar con `fvm flutter --version` que responde `stable (3.44.1 / Dart 3.12.1)`.

### Step 2: Actualizar pubspec.yaml

Actualizar los siguientes constraints (mantener los demás sin cambios):

**Runtime dependencies — major bumps:**
```yaml
connectivity_plus: ^7.1.1
device_info_plus: ^13.1.0
file_picker: ^11.0.2
firebase_analytics: ^12.4.1
firebase_core: ^4.9.0
firebase_crashlytics: ^5.2.2
firebase_messaging: ^16.2.2
firebase_remote_config: ^6.5.1
flutter_dotenv: ^6.0.1
flutter_foreground_task: ^9.2.2
flutter_local_notifications: ^21.0.0
flutter_secure_storage: ^10.3.1
flutter_timezone: ^5.1.0
get_it: ^9.2.1
go_router: ^17.2.3
infinite_scroll_pagination: ^5.1.1
intl: ^0.20.2
local_auth: ^3.0.1      # era "any"
package_info_plus: ^9.0.1
permission_handler: ^12.0.2
app_links: ^7.1.1
timezone: ^0.11.0
flutter_foreground_task: ^9.2.2
```

**Webview — levantar pin:**
```yaml
webview_flutter_wkwebview: ^3.26.0  # era ^3.16.0, pin levantado (bug login fixeado en 3.18.3)
```

**Nota**: `flutter_mentions` NO se cambia — sigue siendo git dep al fork `syncro_update` (ya actualizado a Dart 3).

### Step 3: flutter pub get

```bash
cd syncro-flutter
fvm flutter pub get
```

Si hay errores de resolución, documentarlos en `baseline-analysis.md` y parar.

### Step 4: flutter analyze + capturar baseline

```bash
cd syncro-flutter
fvm flutter analyze 2>&1 | tee /tmp/flutter_analyze_baseline.txt
wc -l /tmp/flutter_analyze_baseline.txt
```

### Step 5: Documentar baseline

Crear `task-01-sdk-baseline/baseline-analysis.md` con:
- Versiones confirmadas (Flutter, Dart, packages clave)
- Output completo de `flutter analyze` (o resumen si es muy largo)
- Conteo de errores por categoría/package
- Issues agrupados por tarea responsable (task-02 through task-09)

## Testing

- [ ] `fvm flutter --version` muestra stable (3.44.1 / Dart 3.12.1)
- [ ] `fvm flutter pub get` termina sin errores
- [ ] `baseline-analysis.md` creado con el output completo

## Completion Criteria

- [ ] `.fvmrc` apunta a stable (3.44.1 / Dart 3.12.1)
- [ ] `pubspec.yaml` actualizado con todos los nuevos constraints
- [ ] `fvm flutter pub get` exitoso
- [ ] `baseline-analysis.md` creado con errors/warnings documentados
- [ ] Status actualizado en `status.md`
