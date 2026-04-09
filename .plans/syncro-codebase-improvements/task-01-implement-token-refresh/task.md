# Task 01 — Implement Token Refresh

**Plan**: syncro-codebase-improvements
**Priority**: 🔴 Critical
**Branch**: `plan/syncro-codebase-improvements/task-01`

## Problem

`AuthenticationCubit._refreshToken()` always returns `true` without refreshing the token:

```dart
Future<bool> _refreshToken() async {
  try {
    // TODO: Implement actual token refresh logic using RefreshTokenUseCase
    return true;  // BUG
  } catch (e) {
    return false;
  }
}
```

When a user's token expires, the `RestNetworkServiceImpl` catches 401, calls `refreshToken()` callback, and retries the request. But since the callback doesn't actually refresh the token, the retry uses the same expired token → fails again → users see errors without being redirected to login.

## Objective

Implement actual OAuth token refresh using the existing `StorageManager` and `LoginRepository` infrastructure.

## Implementation Steps

1. **Investigate**: Check what `StorageManager` stores (access token, refresh token, expiry). Look at `LoginRepositoryImpl.getToken()` and `OAuthWebAuth` for the refresh token flow.

2. **Create `RefreshTokenUseCase`**:
   - File: `lib/features/authentication/infrastructure/refresh_token_usecase.dart`
   - Takes `LoginRepository` and `StorageManager` as deps
   - Calls the OAuth refresh token endpoint with the stored refresh token
   - Updates stored tokens on success
   - Returns `Either<Failure, String>` (new access token)

3. **Wire into `AuthenticationCubit`**:
   - Add `RefreshTokenUseCase` as constructor param
   - Replace the `TODO` body in `_refreshToken()` with actual call
   - On success: update `TokenCubit` with new token
   - On failure: call `_handleUnauthenticated()`

4. **Wire into `RestNetworkServiceImpl`**:
   - The `refreshToken` callback is injected in `service_locator.dart` when `NetworkService` is initialized
   - Update `serviceLocatorInit()` to pass the actual refresh callback

5. **Update `service_locator.dart`**:
   - Register `RefreshTokenUseCase`
   - Pass refresh callback to `NetworkServiceImpl.initService()`

6. **Add tests**:
   - Test `RefreshTokenUseCase` — success and failure cases
   - Test `AuthenticationCubit._refreshToken()` integration

## Files to Modify

- `lib/features/authentication/infrastructure/refresh_token_usecase.dart` (new)
- `lib/features/authentication/application/authentication_cubit.dart`
- `lib/app/dependency/service_locator.dart`
- `lib/core/networking/services/network_service_impl.dart` (possibly)
- `test/features/authentication/infrastructure/refresh_token_usecase_test.dart` (new)

## Acceptance Criteria

- [ ] `_refreshToken()` no longer returns `true` unconditionally
- [ ] On 401, the app attempts to refresh the OAuth token
- [ ] On refresh success: request is retried with new token
- [ ] On refresh failure: user is logged out (redirected to login)
- [ ] Unit tests cover success and failure paths
