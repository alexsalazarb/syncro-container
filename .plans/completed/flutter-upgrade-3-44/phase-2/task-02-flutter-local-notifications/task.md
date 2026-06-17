# Task: flutter_local_notifications 18→21 Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 2 — API Rewrites
**Task ID**: task-02
**Task Path**: phase-2/task-02-flutter-local-notifications
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Migrar `flutter_local_notifications` de la versión 18 a la 21. Los cambios en v19, v20 y v21 acumulan breaking changes que requieren actualizar todas las call sites en `notifications_manager.dart`.

## Context

**Breaking changes acumulados:**
- **v19**: Remueve `uiLocalNotificationDateInterpretation` de `zonedSchedule()`
- **v20**: TODOS los parámetros posicionales pasan a nombrados en `initialize()`, `show()`, `periodicallyShow()`, `cancel()`, `zonedSchedule()`, `startForegroundService()`, y más
- **v21**: Min Flutter 3.38.1 / Dart 3.10.0 (ya cumplido por task-01)

**1 archivo afectado**:
- `lib/core/services/push_notification/notifications_manager.dart`

**Nota**: Firebase Messaging también usa este archivo — task-05 depende de esta task porque comparten el mismo archivo.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | modify | Actualizar todas las call sites de flutter_local_notifications |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- `task-03-infinite-scroll-pagination/` — owned by task-03
- Ningún otro archivo de features

## Implementation Steps

### Step 1: Leer el archivo actual

Leer `notifications_manager.dart` completo para entender qué APIs se usan.

### Step 2: Actualizar initialize()

Cambiar parámetros posicionales a nombrados. Ejemplo:
```dart
// ANTES (v18):
await flutterLocalNotificationsPlugin.initialize(
  initializationSettings,
  onDidReceiveNotificationResponse,
  onDidReceiveBackgroundNotificationResponse,
);

// DESPUÉS (v21):
await flutterLocalNotificationsPlugin.initialize(
  initializationSettings,
  onDidReceiveNotificationResponse: onDidReceiveNotificationResponse,
  onDidReceiveBackgroundNotificationResponse: onDidReceiveBackgroundNotificationResponse,
);
```

### Step 3: Actualizar show()

```dart
// ANTES:
flutterLocalNotificationsPlugin.show(id, title, body, notificationDetails);

// DESPUÉS:
flutterLocalNotificationsPlugin.show(id, title, body, notificationDetails, payload: payload);
// (payload ya era nombrado, pero verificar orden de los demás)
```

### Step 4: Actualizar zonedSchedule() — remover uiLocalNotificationDateInterpretation

```dart
// ANTES (v18):
flutterLocalNotificationsPlugin.zonedSchedule(
  id, title, body, scheduledDate, notificationDetails,
  androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
  uiLocalNotificationDateInterpretation: UILocalNotificationDateInterpretation.absoluteTime,
  payload: payload,
);

// DESPUÉS (v21):
flutterLocalNotificationsPlugin.zonedSchedule(
  id, title, body, scheduledDate, notificationDetails,
  androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
  payload: payload,
  // uiLocalNotificationDateInterpretation REMOVIDO
);
```

### Step 5: Actualizar cancel() y cualquier otro método usado

```dart
// ANTES: cancel(id)
// DESPUÉS: cancel(id) — verificar si el segundo param (tag) se pasaba posicionalmente
```

### Step 6: Actualizar startForegroundService() si se usa

Si `startForegroundService` está en el archivo, actualizar parámetros a nombrados según la firma de v21.

### Step 7: Verificar compilación

```bash
cd syncro-flutter
fvm flutter analyze lib/core/services/push_notification/notifications_manager.dart
```

Zero errores en ese archivo.

## Testing

- [ ] `fvm flutter analyze lib/core/services/push_notification/notifications_manager.dart` sin errores
- [ ] No hay llamadas con parámetros posicionales a métodos de flutter_local_notifications

## Completion Criteria

- [ ] `notifications_manager.dart` actualizado con parámetros nombrados en todas las calls
- [ ] `uiLocalNotificationDateInterpretation` removido de `zonedSchedule()` si existía
- [ ] `fvm flutter analyze` en el archivo sin errores
- [ ] Status actualizado en `status.md`
