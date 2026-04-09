# Project Overview — syncro-flutter

**Last Updated**: April 2026

## Context

Syncro is an MSP (Managed Service Provider) field-service mobile app for technicians. The Flutter codebase targets iOS and Android. It is the sole project in this container repo.

## Stack

| Aspect | Value |
|--------|-------|
| Language | Dart 3.x (`sdk: ">=3.8.0 <4.0.0"`) |
| Framework | Flutter (iOS + Android) |
| Version | 1.4.0+412 |
| State Management | `flutter_bloc` 9.x — Cubit-first, BLoC where stream transformations are needed |
| Navigation | `go_router` 14.x — `StatefulShellRoute.indexedStack` for bottom-tab nav |
| DI / Service Locator | `get_it` 8.x for singletons; `BlocProvider`/`RepositoryProvider` for scoped widget-tree DI |
| HTTP | `dio` 5.x with custom interceptors (auth headers, token refresh, connectivity check) |
| Local DB | `hive` 2.x + `hive_flutter` (Hive boxes managed by `LocalDBService`) |
| Preferences | `shared_preferences` |
| Secure Storage | `flutter_secure_storage` |
| Real-time | `phoenix_socket` — Elixir/Phoenix WebSocket for chat |
| Auth | OAuth2 via `oauth_webauth` (browser-based) + token stored in secure storage |
| Error Handling | `dartz` — `Either<Failure, T>` throughout all layers |
| Testing | `flutter_test` + `mockito` for mocks, `bloc_test` for cubits |
| Analytics | Firebase Analytics + `pendo_sdk` |
| Error Reporting | Firebase Crashlytics |
| Feature Flags | Firebase Remote Config via `FeatureFlagManager` |
| Push Notifications | Firebase Cloud Messaging (FCM) |

## Entry Point

`lib/main.dart` — `main()` runs inside `runZonedGuarded` with a structured init pipeline:

1. `Firebase.initializeApp()` (with retry)
2. Parallel: `initializeDateFormatting`, `Hive.init`, `SharedPreferences.init`, `dotenv.load`, `PreloadImages.init`
3. Parallel: `FirebaseRemoteConfigManager.initCacheFirst()`, `OAuthWebAuth.init`
4. Synchronous: `serviceLocatorInit(env)` — registers GetIt singletons
5. Sequential: `NotificationManager.init()` → `NotificationManager.register()`
6. Fire-and-forget: `PendoService.init()`
7. `bootstrap()` → mounts `AppRepositories(child: AppProviders(child: App()))`

## Top-Level lib Structure

```
lib/
  main.dart                   # Entry point — app init pipeline
  app/
    dependency/
      service_locator.dart    # GetIt singleton registrations
      app_repositories.dart   # RepositoryProvider widget tree registrations
      app_providers.dart      # BlocProvider widget tree registrations
    view/
      app.dart                # MaterialApp.router using RouteCubit's GoRouter
      boostrap.dart           # Wraps runApp with Firebase error zones
  core/
    configs/                  # Environment, env keys
    design_system/            # Themes, ThemeCubit
    domain/                   # Shared mixins: NamedEntityMixin, SelectableEntity
    enums/                    # App-wide enums
    global_widgets/           # Reusable atoms, molecules, custom pickers
    networking/               # NetworkService abstraction + Dio impl
    routing/                  # RouteCubit + GoRouter (app_router.dart)
    services/                 # Analytics, Notifications, RemoteConfig, Hive, Storage, Timezone
    usecases/                 # Abstract UseCase base classes
    utils/                    # logger, SharedPreferences, extensions
  features/                   # Feature-first modules (17 features)
```

## Key Files

| File | Purpose |
|------|---------|
| `lib/main.dart` | App initialization pipeline |
| `lib/app/dependency/service_locator.dart` | GetIt singleton registrations |
| `lib/app/dependency/app_repositories.dart` | Widget-tree repository registrations |
| `lib/app/dependency/app_providers.dart` | Widget-tree BLoC/Cubit registrations |
| `lib/core/routing/route_cubit.dart` | Navigation controller (GoRouter wrapper) |
| `lib/core/routing/app_router.dart` | GoRouter config, shell branches |
| `lib/core/routing/routes_names.dart` | `AppRoute` enum with paths and names |
| `lib/core/networking/services/rest_api/rest_network_service.dart` | Dio-based HTTP client with token refresh |
| `lib/core/services/chat_websocket_service.dart` | Phoenix WebSocket singleton for chat |
| `lib/core/configs/environment.dart` | Multi-env config loaded from `.env` files |

## Features List

| Feature | Description |
|---------|-------------|
| `authentication` | OAuth + credential login, token lifecycle |
| `splash` | Startup routing screen |
| `home` | Bottom tab shell + `HomePage` |
| `dashboard` | Summary of tickets, appointments, alerts |
| `ticket` | Full ticket lifecycle (list, create, details, notes, charges, worksheets, timer entries) |
| `appointments` | Appointment list, create, edit |
| `assets` | RMM asset list, filter, barcode scan |
| `asset_detail` | Asset details, remote scripts, add-to-queue |
| `alerts` | RMM alerts list |
| `chat` | Chat interaction list |
| `chat_detail` | Real-time chat with Phoenix WebSocket |
| `customers` | Customer search and details |
| `end_user` | Contact/end-user detail and edit |
| `technicians` | Technician list |
| `settings` | Appearance, locale, about, app lock |
| `time_clock` | Technician clock-in/out |
| `unlock_app` | Biometric / PIN app lock |
