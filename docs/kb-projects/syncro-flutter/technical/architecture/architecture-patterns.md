# Architecture Patterns — syncro-flutter

**Last Updated**: April 2026

## Context

Syncro Flutter follows a modified **Clean Architecture** pattern with **feature-first** directory organization and **BLoC/Cubit** state management. The codebase is Class B (partially structured) — the core patterns are solid but there are DI inconsistencies and layer violations in some features.

---

## Layer Structure (per feature)

Each feature is organized into 4 layers:

```
features/[feature]/
  domain/           # Entities, params, response types, repository interfaces
  infrastructure/   # Repository implementations, use cases, deserializers
  application/      # Cubits / BLoCs (state management)
  presentation/     # Pages, widgets
```

> **Known deviation**: Use cases are placed in `infrastructure/` rather than `domain/` in most features. This is the established pattern — follow it unless refactoring.

---

## State Management — Cubit / BLoC

`flutter_bloc` is the exclusive state management library. Cubits are preferred; BLoC is used only when stream transformations are needed.

```dart
class TicketCubit extends Cubit<TicketState> {
  TicketCubit({
    required this.getTicketUseCase,
    required this.ticketFilterCubit,
  }) : super(TicketInitial());
}
```

**Cubit extension (`safeEmit`)**: Always use `safeEmit()` from `core/utils/cubit_extension.dart` instead of `emit()` to guard against emitting after close.

---

## Error Handling — Either<Failure, T>

All repository methods and use cases return `Either<Failure, T>` from the `dartz` package. Never throw from repositories.

```dart
// Repository method signature
Future<Either<Failure, GetCustomersResponse>> getCustomers(GetCustomersParams params);

// Usage in cubit
final result = await useCase.call(params);
result.fold(
  (failure) => safeEmit(ErrorState(failure.message)),
  (data) => safeEmit(SuccessState(data)),
);
```

---

## Dependency Injection — Two-tier

There are two DI mechanisms. Understanding where each is registered is critical:

### Tier 1 — GetIt singletons (`service_locator.dart`)

Used for services and cubits that must be alive for the full app lifetime:
- `Environment`, `NetworkService`, `StorageManager`
- `AuthenticationCubit`, `RouteCubit`, `TokenCubit`
- `ChatWebSocketService`, `AnalyticsManager`, `PendoService`
- `AssetsCubit`, `AssetFilterCubit`
- Some repositories: `LoginRepository`, `AssetsRepository`, `TimeClockRepository`, `WorksheetRepository`

### Tier 2 — Widget-tree (`app_repositories.dart` + `app_providers.dart`)

Used for repositories and cubits scoped to the widget tree (most of them):
- `AppRepositories` registers most feature repositories as `RepositoryProvider`
- `AppProviders` registers most feature cubits as `BlocProvider`

### ⚠️ Known DI Inconsistency

Some repositories are registered in **both** tiers (GetIt AND widget tree):
- `AssetsRepository` — in `service_locator.dart` AND `app_repositories.dart`
- `TimeClockRepository` — registered twice in `app_providers.dart`
- `WorksheetRepository` — in `service_locator.dart` AND `app_repositories.dart`

In `app_repositories.dart`, `_configureWorksheetRepository()` incorrectly registers the concrete type `WorksheetRepositoryImpl` as the key (not the interface). This breaks type-safe `context.read<WorksheetRepository>()`.

---

## Navigation — GoRouter

Navigation is managed through `RouteCubit` which wraps a `GoRouter` instance.

```dart
// Navigate imperatively via RouteCubit
getIt<RouteCubit>().goToRoute(AppRoute.ticketDetail, parameters: params);

// Or via widget context (for in-widget navigation)
context.pushNamed(AppRoute.ticketDetail.name, extra: params);
```

### Shell Branches (Bottom Tabs)

The app uses `StatefulShellRoute.indexedStack` with 5 branches:
1. **Dashboard** (`/home`) — with nested Settings, Customer, TimeClock routes
2. **Appointments** (`/appts`)
3. **Chat** (`/chat`)
4. **Assets** (`/assets`) or **Alerts** (`/alerts`) — toggled by `FeatureFlagManager.showLastAssetFeatures()`
5. **Tickets** (`/tickets`) — most complex, 15+ nested routes

### Route Passing

Data is passed via `state.extra` (typed cast). Always use `_createTypedRoute<T>()` for type-safe route param passing.

### Push Notification Routing

`RouteCubit.redirectOnNotification(PushNotification)` handles deep-linking from FCM notifications. Supported sources: `appointment`, `ticket`, `comment`, `rmmAlert`, `openStruct` (chat).

---

## Networking — NetworkService

All HTTP calls go through `NetworkService.sendRequestCall()`. Repositories never import `Dio` directly.

```dart
// Repository pattern
final result = await networkService.sendRequestCall<TicketDetailsResponse>(
  request: AppRequests.getTicketById.requestOption(),
  deserializer: TicketByIdDeserializer(),
  pathParameters: {'id': ticketId.toString()},
);
```

**Token refresh**: The `RestNetworkServiceImpl` intercepts 401 responses and calls `refreshToken()` (a `Future<String?> Function()` injected at construction). The refresh uses a `Completer` to coalesce concurrent refresh attempts.

**⚠️ Known Issue**: `AuthenticationCubit._refreshToken()` returns `true` always — actual token refresh is NOT implemented. This is a critical gap.

---

## Feature Flags — Firebase Remote Config

Feature flags are accessed through `FeatureFlagManager`:

```dart
if (FeatureFlagManagerImpl().showLastAssetFeatures()) {
  // show assets branch
}
```

`FeatureFlagManagerImpl` reads values from `FirebaseRemoteConfigManager`. Remote Config uses "cache-first" activation at startup to apply cached values immediately.

---

## Environment Configuration

Multiple environments are supported via `.env` files in `assets/env/`:
- `qa`, `ss1` – `ss12` (staging servers), `prod`

Environment is selected at runtime (if `showSwitchEnvironment` feature flag is on) and persisted via `SharedPreferences`. In production builds, always defaults to `prod`.

---

## Real-time Chat — Phoenix Socket

`ChatWebSocketService` is a singleton service managing a Phoenix WebSocket connection:
- Topics: `account:user:{accountId}:{userId}` (main channel) and `account:asset:{accountId}:{assetId}` (message channel)
- Events: `get_interactions_batched`, `message`, `read`, `archive`, `assign_interaction`
- Exposes `Stream` controllers for: chats list, individual chat, messages, new chat
- Includes health check timer (every 5 min), connectivity monitoring, and exponential backoff reconnection

---

## Local Storage — Hive

Hive boxes are managed by `LocalDBService.instance`. Always access Hive via the service, never directly open boxes in feature code.
