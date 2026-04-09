# Firebase Integration — syncro-flutter

**Last Updated**: April 2026

## Context

Firebase is the primary backend-as-a-service layer for analytics, crash reporting, push notifications, and feature flags.

## Services Used

| Package | Version | Purpose |
|---------|---------|---------|
| `firebase_core` | ^3.11.0 | Required foundation for all Firebase services |
| `firebase_crashlytics` | ^4.3.2 | Crash and error reporting |
| `firebase_analytics` | ^11.4.2 | Event tracking (wrapped by `AnalyticsManager`) |
| `firebase_messaging` | ^15.2.2 | FCM push notifications |
| `firebase_remote_config` | ^5.4.0 | Feature flags and remote settings |

## Initialization Order

Firebase must be the **first** service initialized in `main.dart`:

```dart
await Firebase.initializeApp();  // Step 1 — always first
await FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(true);
// Then all other services...
```

## Crashlytics

All errors should be reported through Crashlytics in non-debug contexts:

```dart
// Flutter UI errors
FlutterError.onError = (details) {
  FirebaseCrashlytics.instance.recordFlutterError(details);
};

// Platform (Dart) errors
PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};

// Manual error reporting in services
FirebaseCrashlytics.instance.recordError(
  e, stackTrace,
  fatal: false,
  reason: 'Descriptive context message',
);
```

**Convention**: Always include a `reason:` string to help triage. Use `fatal: true` only for errors that prevent app function.

## Remote Config — Feature Flags

Managed via `FirebaseRemoteConfigManager` (singleton) and accessed through `FeatureFlagManager`:

```dart
// Always go through FeatureFlagManager — never access RemoteConfig directly
final shouldShow = FeatureFlagManagerImpl().showLastAssetFeatures();
```

**Initialization**: Uses "cache-first" strategy — `activate()` is called at startup to immediately apply previously fetched values, then fresh values are fetched in the background for the next launch.

Current feature flags:
- `showSwitchEnvironment` — enables the in-app environment switcher (dev only)
- `showLastAssetFeatures` — hardcoded to `true` in code (not actually reading remote config)

## Analytics — AnalyticsManager

All analytics events go through `AnalyticsManager` (GetIt singleton). Never call `FirebaseAnalytics` directly:

```dart
getIt<AnalyticsManager>().trackEvent('ticket_created', {'ticket_id': id});
```

## Push Notifications — NotificationManager

`NotificationManager` (singleton) wraps FCM. It must be initialized **after** `serviceLocatorInit()` because it depends on GetIt-registered services:

```dart
await NotificationManager.instance.init();
await NotificationManager.instance.register();
```

On notification tap, `NotificationManager` calls `RouteCubit.redirectOnNotification(PushNotification)` which handles deep-link navigation. Supported push sources:
- `appointment` → opens appointment detail
- `ticket` → opens ticket detail
- `comment` → opens ticket detail
- `rmmAlert` → opens alerts tab
- `openStruct` → opens chat detail (async — waits for WebSocket to initialize)
