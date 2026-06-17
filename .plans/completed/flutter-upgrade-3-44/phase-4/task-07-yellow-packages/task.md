# Task: Yellow Packages — Service Layer

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 4 — Routing + Yellow Packages
**Task ID**: task-07
**Task Path**: phase-4/task-07-yellow-packages
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Actualizar los packages yellow del service layer que requieren cambios en código fuente: `flutter_timezone`, `local_auth`, `flutter_secure_storage`, `timezone`, `get_it`, y `device_info_plus`.

## Context

**Cambios por package:**

| Package | Cambio clave |
|---------|-------------|
| flutter_timezone 4→5 | `getLocalTimezone()` ahora retorna `TimezoneInfo` no `String` → usar `.name` |
| local_auth any→3.0.1 | `stickyAuth` → `persistAcrossBackgrounding`; throws `LocalAuthException` no `PlatformException`; min Android API 24 |
| flutter_secure_storage 9→10 | Cipher change (usar `migrateOnAlgorithmChange: true`); min Android SDK 23; iOS min 12 |
| timezone 0.9→0.11 | `Location.offset` cambió de `int` a `Duration` — usar `.offset.inSeconds` o `.inHours` |
| get_it 8→9 | Remueve param `strictDisposalOrder` de `reset()`, `resetScope()`, etc. |
| device_info_plus 11→13 | Remueve `AndroidDeviceInfo.serialNumber` en v12 |

**Archivos afectados:**

```
lib/core/services/timezone_service.dart       (flutter_timezone)
lib/core/utils/app_datetime.dart              (timezone/TZDateTime — verificar offset usage)
lib/features/unlock_app/application/unlock_app_cubit.dart     (local_auth)
lib/features/unlock_app/infrastructure/unlock_service.dart    (local_auth)
lib/features/unlock_app/presentation/unlock_app_page.dart     (local_auth)
lib/features/settings/application_lock/application/application_lock_cubit.dart  (local_auth)
lib/features/settings/application_lock/presentation/application_lock_page.dart  (local_auth)
lib/core/services/storage.dart               (flutter_secure_storage)
lib/app/dependency/service_locator.dart      (get_it — verificar strictDisposalOrder)
```

`device_info_plus`: buscar `serialNumber` en el codebase — si no se usa, no hay cambios requeridos.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/timezone_service.dart` | modify | `.getLocalTimezone()` → `.name` |
| `syncro-flutter/lib/core/utils/app_datetime.dart` | review/modify | `Location.offset` → `.inSeconds`/`.inHours` |
| `syncro-flutter/lib/features/unlock_app/application/unlock_app_cubit.dart` | modify | local_auth API + exception type |
| `syncro-flutter/lib/features/unlock_app/infrastructure/unlock_service.dart` | modify | local_auth API |
| `syncro-flutter/lib/features/unlock_app/presentation/unlock_app_page.dart` | review | local_auth usage |
| `syncro-flutter/lib/features/settings/application_lock/application/application_lock_cubit.dart` | modify | local_auth API |
| `syncro-flutter/lib/features/settings/application_lock/presentation/application_lock_page.dart` | review | local_auth usage |
| `syncro-flutter/lib/core/services/storage.dart` | modify | flutter_secure_storage — agregar migrateOnAlgorithmChange |
| `syncro-flutter/lib/app/dependency/service_locator.dart` | review/modify | get_it — remover strictDisposalOrder si se usa |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- Archivos owned por otras tasks

## Implementation Steps

### Step 1: flutter_timezone → TimezoneInfo

En `timezone_service.dart`, el return type de `FlutterTimezone.getLocalTimezone()` cambió:

```dart
// ANTES (v4):
final String timezone = await FlutterTimezone.getLocalTimezone();

// DESPUÉS (v5):
final TimezoneInfo timezoneInfo = await FlutterTimezone.getLocalTimezone();
final String timezone = timezoneInfo.name;
```

Verificar si hay otras llamadas a `getLocalTimezone()` en el codebase:
```bash
rg "getLocalTimezone" lib/ --type dart
```

### Step 2: timezone — Location.offset

Buscar uso de `Location.offset` o `location.offset` en el codebase:
```bash
rg "\.offset\b" lib/ --type dart | rg -v "// "
```

Si se usa como `int`, actualizar a `.offset.inSeconds` o `.offset.inHours` según el contexto.

### Step 3: local_auth — API updates

En los archivos de unlock_app y application_lock:

**a) Renombrar parámetro:**
```dart
// ANTES:
await auth.authenticate(stickyAuth: true, ...);

// DESPUÉS:
await auth.authenticate(persistAcrossBackgrounding: true, ...);
```

**b) Actualizar exception handling:**
```dart
// ANTES:
} catch (e) {
  if (e is PlatformException) { ... }
}

// DESPUÉS:
} catch (e) {
  if (e is LocalAuthException) {
    // e.code es un LocalAuthExceptionCode enum
  }
}
```

**c) Verificar useErrorDialogs:**
```dart
// REMOVER si existe:
await auth.authenticate(useErrorDialogs: true, ...);
// useErrorDialogs fue eliminado
```

### Step 4: flutter_secure_storage — migrateOnAlgorithmChange

En `storage.dart`, agregar la opción de migración al instanciar:

```dart
// ANTES:
final storage = FlutterSecureStorage();

// DESPUÉS:
final storage = FlutterSecureStorage(
  aOptions: AndroidOptions(
    encryptedSharedPreferences: true,  // si se usaba
    migrateOnAlgorithmChange: true,    // CRÍTICO: evita pérdida de datos
  ),
);
```

**CRÍTICO**: Sin `migrateOnAlgorithmChange: true`, los datos cifrados con el algoritmo anterior no podrán leerse. Probar en device con sesión activa antes de shipear.

### Step 5: get_it — strictDisposalOrder

Buscar en `service_locator.dart`:
```bash
rg "strictDisposalOrder" lib/ --type dart
```

Si existe, remover el parámetro (LIFO ahora siempre se enforza):
```dart
// REMOVER:
GetIt.instance.reset(strictDisposalOrder: true);
```

### Step 6: device_info_plus — serialNumber

```bash
rg "serialNumber" lib/ --type dart
```

Si se usa `AndroidDeviceInfo.serialNumber`, remover o reemplazar con alternativa (el valor fue removido en v12 por restricciones de privacidad de Android).

### Step 7: Verificar compilación

```bash
cd syncro-flutter
fvm flutter analyze lib/core/services/timezone_service.dart lib/core/utils/app_datetime.dart lib/features/unlock_app/ lib/features/settings/application_lock/ lib/core/services/storage.dart lib/app/dependency/service_locator.dart
```

## Testing

- [ ] `fvm flutter analyze` sin errores en los 9 archivos
- [ ] `flutter_secure_storage`: probar en device con datos existentes — los datos se leen correctamente después del upgrade
- [ ] `local_auth`: probar biometric unlock en device físico (Face ID / fingerprint)
- [ ] Timezone: verificar que la zona horaria del device se obtiene correctamente

## Completion Criteria

- [ ] `getLocalTimezone()` actualizado a usar `.name`
- [ ] `local_auth` calls actualizadas (stickyAuth → persistAcrossBackgrounding, exception type)
- [ ] `flutter_secure_storage` tiene `migrateOnAlgorithmChange: true`
- [ ] `get_it` sin `strictDisposalOrder` si existía
- [ ] `fvm flutter analyze` sin errores
- [ ] Status actualizado en `status.md`
