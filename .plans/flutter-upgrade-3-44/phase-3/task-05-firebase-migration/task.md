# Task: Firebase 5-Package Lock-step Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 3 — Platform + Services
**Task ID**: task-05
**Task Path**: phase-3/task-05-firebase-migration
**Depends On**: task-01, task-02
**JIRA**: SE-12530

## Objective

Actualizar los 5 packages de Firebase en lock-step (deben ir juntos porque comparten el mismo native SDK bump: iOS SDK 12 / Android SDK 34). Los cambios a nivel Flutter son menores — principalmente se remueven métodos ya deprecados.

## Context

**5 packages a actualizar juntos:**
| Package | Actual | Target |
|---------|--------|--------|
| firebase_core | 3.11.0 | 4.9.0 |
| firebase_analytics | 11.4.2 | 12.4.1 |
| firebase_crashlytics | 4.3.2 | 5.2.2 |
| firebase_messaging | 15.2.2 | 16.2.2 |
| firebase_remote_config | 5.4.0 | 6.5.1 |

**Breaking changes Flutter-level:**
- `firebase_messaging 16.0.0`: Remueve `iOSNotificationSettings`, `requestNotificationPermissions()`, `deleteInstanceID()` (ya deprecados desde hace mucho)
- `firebase_analytics 12.0.0`: Remueve métodos deprecados de analytics
- Los demás: solo native SDK bump, sin cambios Flutter-level

**Depende de task-02** porque `notifications_manager.dart` es shared y task-02 lo toca primero.

**13 archivos afectados:**
```
lib/firebase_options.dart
lib/main.dart
lib/core/services/push_notification/push_notification.dart
lib/core/services/push_notification/notifications_manager.dart  (post task-02)
lib/core/services/remote_config/feature_flag_manager.dart
lib/core/services/remote_config/remote_config_manager.dart
lib/core/services/analytics/analytics_manager.dart
lib/core/services/chat_websocket_service.dart
lib/core/networking/services/rest_api/rest_network_service.dart
lib/features/assets/infrastructure/asset_filter_deserializer.dart
lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart
lib/features/ticket/ticket_home/domain/tickets_settings.dart
```

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/firebase_options.dart` | review/modify | Puede requerir regeneración con FlutterFire CLI |
| `syncro-flutter/lib/main.dart` | review/modify | Firebase.initializeApp() — verificar |
| `syncro-flutter/lib/core/services/push_notification/push_notification.dart` | modify | Remover deprecated calls de firebase_messaging |
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | review | Post task-02, verificar firebase_messaging calls |
| `syncro-flutter/lib/core/services/remote_config/feature_flag_manager.dart` | review | firebase_remote_config API |
| `syncro-flutter/lib/core/services/remote_config/remote_config_manager.dart` | review | firebase_remote_config API |
| `syncro-flutter/lib/core/services/analytics/analytics_manager.dart` | review/modify | firebase_analytics — remover deprecated |
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | review | Firebase usage |
| `syncro-flutter/lib/core/networking/services/rest_api/rest_network_service.dart` | review | Firebase usage |
| `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` | review | Firebase usage |
| `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` | review | Firebase usage |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/tickets_settings.dart` | review | Firebase usage |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- `task-02` files — task-02 ya manejó `notifications_manager.dart`

## Implementation Steps

### Step 1: Verificar firebase_options.dart

`firebase_options.dart` puede necesitar regeneración si el FlutterFire CLI fue usado para generarlo. Verificar si la nueva versión de `firebase_core` requiere un formato diferente.

Si es necesario regenerar:
```bash
cd syncro-flutter
flutterfire configure
```

### Step 2: Limpiar firebase_messaging deprecated calls

En `push_notification.dart` y `notifications_manager.dart`, buscar y remover:
- `requestNotificationPermissions()` → reemplazar con `requestPermission()`
- `iOSNotificationSettings` → remover
- `deleteInstanceID()` → remover o reemplazar con `deleteToken()`

```bash
rg "requestNotificationPermissions|iOSNotificationSettings|deleteInstanceID" lib/ --type dart
```

### Step 3: Limpiar firebase_analytics deprecated calls

En `analytics_manager.dart`:
```bash
rg "setCurrentScreen|setUserProperty\b" lib/core/services/analytics/ --type dart
```
Actualizar cualquier método removido según el changelog de firebase_analytics 12.x.

### Step 4: Verificar todos los demás archivos

Para los archivos "review" — leer y verificar que no usan APIs removidas en firebase 4.x/5.x/6.x.

```bash
cd syncro-flutter
fvm flutter analyze lib/firebase_options.dart lib/main.dart lib/core/services/push_notification/ lib/core/services/remote_config/ lib/core/services/analytics/ lib/core/services/chat_websocket_service.dart lib/core/networking/services/rest_api/ lib/features/assets/infrastructure/ lib/features/ticket/ticket_home/infrastructure/ lib/features/ticket/ticket_home/domain/tickets_settings.dart
```

### Step 5: Verificar compilación Android + iOS

```bash
cd syncro-flutter
fvm flutter build apk --debug
fvm flutter build ios --debug --no-codesign
```

## Testing

- [ ] `fvm flutter analyze` sin errores en los 12 archivos Firebase
- [ ] Build Android compila sin errores
- [ ] Build iOS compila sin errores
- [ ] En device: push notifications recibidas (si posible probar manualmente)
- [ ] Remote config se inicializa correctamente (verificar logs)
- [ ] Crashlytics no reporta errores de inicialización

## Completion Criteria

- [ ] Los 5 packages de Firebase funcionando en lock-step
- [ ] No hay llamadas a métodos removidos (requestNotificationPermissions, iOSNotificationSettings, etc.)
- [ ] `fvm flutter analyze` sin errores en archivos Firebase
- [ ] Build Android + iOS compila
- [ ] Status actualizado en `status.md`
