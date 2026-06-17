# Task: go_router 14→17 Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 4 — Routing + Yellow Packages
**Task ID**: task-06
**Task Path**: phase-4/task-06-go-router
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Migrar `go_router` de la versión 14.8.0 a la 17.2.3. Tres major bumps acumulan cambios en URL case-sensitivity, `GoRouteData`, y `ShellRoute` observer notifications.

## Context

**Breaking changes acumulados:**
- **v15**: URLs ahora son **case-sensitive** por defecto. Si hay deep links o rutas con mayúsculas/minúsculas mixtas, agregar `caseSensitive: false` al `GoRouter()`.
- **v16**: `GoRouteData` define `.go()`, `.push()`, `.location` en la clase misma (no en el mixin). Requiere `go_router_builder >= 3.0.0` si se usa type-safe routing. Fix de URL case para branch routes.
- **v17**: `ShellRoute` cambió cómo notifica a los observers. Nuevo parámetro `notifyRootObserver`. Si la app usa observers en ShellRoute, verificar comportamiento.

**7 archivos afectados:**
```
lib/core/delegates/barcode_scanner_search_trailing.dart
lib/core/routing/app_router.dart
lib/core/routing/route_cubit.dart
lib/features/home/presentation/home_page.dart
lib/features/dashboard/presentation/dashboard_page.dart
lib/features/ticket/shared/ticket_cubit_registry.dart
lib/features/home/navbar/presentation/scaffold_with_nested_navigation.dart
```

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/routing/app_router.dart` | modify | GoRouter config — case-sensitivity, ShellRoute |
| `syncro-flutter/lib/core/routing/route_cubit.dart` | review/modify | GoRouter navigation calls |
| `syncro-flutter/lib/features/home/navbar/presentation/scaffold_with_nested_navigation.dart` | review/modify | ShellRoute integration |
| `syncro-flutter/lib/features/home/presentation/home_page.dart` | review | go_router usage |
| `syncro-flutter/lib/features/dashboard/presentation/dashboard_page.dart` | review | go_router usage |
| `syncro-flutter/lib/features/ticket/shared/ticket_cubit_registry.dart` | review | go_router usage |
| `syncro-flutter/lib/core/delegates/barcode_scanner_search_trailing.dart` | review | go_router usage |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- Archivos owned por otras tasks

## Implementation Steps

### Step 1: Leer app_router.dart

Leer `app_router.dart` completo — es el archivo principal de configuración del router.

### Step 2: Agregar caseSensitive: false en GoRouter

A menos que el análisis de rutas confirme que todas las rutas son minúsculas y no hay deep links con mayúsculas, agregar el parámetro por seguridad:

```dart
GoRouter(
  caseSensitive: false,  // AGREGAR — restaura comportamiento v14
  routes: [...],
  ...
)
```

Alternativamente, si se confirma que todas las rutas son lowercase, dejarlo como está (case-sensitive es el nuevo default correcto).

### Step 3: Verificar ShellRoute observers

En `app_router.dart` y `scaffold_with_nested_navigation.dart`:
```bash
rg "observers|NavigatorObserver|ShellRoute" lib/core/routing/ lib/features/home/navbar/ --type dart
```

Si hay observers en `ShellRoute`, revisar el changelog de v17 para el parámetro `notifyRootObserver` y ajustar si necesario.

### Step 4: Verificar GoRouteData (si se usa)

```bash
rg "GoRouteData|TypedGoRoute|TypedShellRoute" lib/ --type dart
```

Si se usa type-safe routing con `go_router_builder`, verificar que los archivos generados son compatibles con v16+. Si hay generación de código, regenerar:
```bash
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Step 5: Verificar todos los archivos afectados

```bash
cd syncro-flutter
fvm flutter analyze lib/core/routing/ lib/features/home/ lib/features/dashboard/ lib/features/ticket/shared/ lib/core/delegates/barcode_scanner_search_trailing.dart
```

### Step 6: Probar navegación en app

Correr en device y verificar:
- Navegación entre tabs funciona
- Deep links funcionan (si los hay)
- Back navigation funciona correctamente
- Shell navigation (nested navigators) funciona

## Testing

- [ ] `fvm flutter analyze` sin errores en los 7 archivos
- [ ] Navegación básica probada en device/emulator
- [ ] No hay rutas rotas por case-sensitivity

## Completion Criteria

- [ ] `caseSensitive: false` agregado si necesario (o confirmado que todas las rutas son lowercase)
- [ ] ShellRoute observers verificados y ajustados si necesario
- [ ] `fvm flutter analyze` sin errores en archivos de routing
- [ ] Status actualizado en `status.md`
