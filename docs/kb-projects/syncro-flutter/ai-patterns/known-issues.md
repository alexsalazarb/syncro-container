# Known Issues — syncro-flutter

**Last Updated**: April 2026

## Context

Problems found during the April 2026 audit (Class B — partially structured codebase). Severity: 🔴 Critical, 🟠 Major, 🟡 Minor.

---

## 🔴 Critical

### Token Refresh Not Implemented

**File**: `lib/features/authentication/application/authentication_cubit.dart`

`AuthenticationCubit._refreshToken()` always returns `true` without actually refreshing the token:

```dart
Future<bool> _refreshToken() async {
  try {
    // TODO: Implement actual token refresh logic using RefreshTokenUseCase
    return true;  // ← BUG: returns true without doing anything
  } catch (e) {
    return false;
  }
}
```

**Impact**: When a token expires (401 from server), the app pretends to refresh but silently fails. Users may see unexpected errors or stale data without being redirected to login.

**Fix needed**: Implement `RefreshTokenUseCase` and wire it into `_refreshToken()`.

---

## 🟠 Major

### Double Repository Registration (DI Inconsistency)

> ✅ **Resolved** by SE-11671 (`9df940f2`) — "Complete DI migration — remove remaining repository registrations from GetIt". All repositories now have a single canonical source; GetIt singletons are delegated to by `RepositoryProvider` where needed.

---

### WorksheetRepository Registered Under Wrong Type

> ✅ **Resolved** by SE-11671 (`9df940f2`). `app_repositories.dart` now registers `RepositoryProvider<WorksheetRepository>` (the interface), so `context.read<WorksheetRepository>()` works correctly.

---

### Use Cases in Infrastructure Layer (Architecture Violation)

In most features, use cases are placed in `infrastructure/` rather than `domain/`. This is an established pattern in this codebase, but it violates Clean Architecture (use cases belong to the domain). New features should follow the existing convention unless a refactor is planned.

---

## 🟡 Minor

### Space in Directory Name

**Path**: `lib/features/ticket/ticket_custom_fields/edit custom fields/`

This path contains a space, which can cause issues on case-sensitive filesystems, CI tooling, and shell scripts. The import in `route_cubit.dart` uses URL-encoded path `edit%20custom%20fields`.

**Fix**: Rename to `edit_custom_fields/`.

---

### Free Function Outside Class in app_providers.dart

**File**: `lib/app/dependency/app_providers.dart`

`_configureUnlockApp()` is defined as a free function outside the `AppProviders` class, while all other configure methods are class members. This is an inconsistency.

---

### No Localization

Despite the AGENTS.md mentioning `AppLocalizations`, there is no `l10n/` directory and no ARB files. All user-facing strings are hardcoded. The `.hardcoded` extension marks intentional hardcoding but this is a debt item for internationalization.

---

### Inter-Cubit Dependencies in Constructor

Some cubits receive other cubits as constructor dependencies:

```dart
TechniciansCubit(context.read<TicketsSettingsCubit>())
TicketFilterCubit(context.read<TicketsSettingsCubit>())
```

This creates implicit ordering requirements in `AppProviders` (dependent cubit must be registered after its dependency). If `TicketsSettingsCubit` is removed or renamed, silent runtime errors occur.

---

### Navigation Uses Magic Delays

Several navigation methods use `await Future.delayed(const Duration(milliseconds: 100))` or 500ms/900ms delays to sequence navigations. These are fragile timing assumptions that may break on slow devices.

**Affected**: `RouteCubit.goToCurrentRoute()`, `_handlePushNotificationNavigation()`, `handleRedirect()`
