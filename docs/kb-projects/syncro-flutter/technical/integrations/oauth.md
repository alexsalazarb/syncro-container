# OAuth Login Integration — syncro-flutter

**Last Updated**: September 2026

## Context

Authentication uses OAuth2 via a webview-based flow. The `oauth_webauth` package handles the browser authorization.

## Package

`oauth_webauth: ^5.1.0`

## Flow

1. User taps "Login with [Company]" → opens `LoginWebView` with the OAuth authorization URL
2. Authorization URL: `https://admin.{basePath}/oauth/authorize?client_id={clientID}&redirect_uri={redirectUri}&...`
3. After authorization, the webview receives the redirect with an auth code
4. `GetTokenUseCase` exchanges the auth code for an access token + session token
5. Tokens stored in `StorageManager` (secure storage via `flutter_secure_storage`)
6. `TokenCubit` holds the in-memory token state for request headers
7. `AuthenticationCubit.loggedIn(user:)` is called → navigates to home

## Credentials (from `.env`)

- `CLIENT_ID` → `Environment.clientID`
- `CLIENT_SECRET` → `Environment.clientSecret`
- `REDIRECT_URL` → `Environment.redirectUri`
- `BASE_PATH` → base domain (e.g., `syncromsp.com`)

## Token Storage

Tokens are stored in `StorageManager` which uses `flutter_secure_storage` as the backing store. `TokenCubit` (GetIt singleton) holds the current in-memory token for injection into request headers.

## Token Refresh

`AuthenticationCubit._refreshToken()` delegates to `LoginRepository.refreshToken()`, which performs a real `grant_type=refresh_token` call against `/oauth/token` and persists the new token to both `TokenCubit` and `StorageManager` on success. Two things trigger it:

1. **Reactively**, on any 401 from `RestNetworkServiceImpl`'s interceptor (`_handleError` → `_getOrCreateRefreshToken`, `Completer`-coalesced so concurrent 401s share one refresh).
2. **At cold boot**, from `AuthenticationCubit.checkAuthentication()` → `_getUser()`, when `MeUseCase(isCheckingToken: true)` fails.

The refresh-grant HTTP call itself runs through a dedicated `AuthTokenClient` (`lib/core/networking/services/rest_api/auth_token_client.dart`) — deliberately **not** through `RestNetworkServiceImpl`'s shared Dio instance/interceptor chain. This is load-bearing, not incidental: routing the refresh call through the same interceptor used to cause a circular-await deadlock whenever the refresh-grant request itself came back 401 (the retry interceptor would re-fire for that nested request while the original `_refreshTokenCompleter` was still pending, so it ended up awaiting its own never-completing future). Once stuck, every subsequent 401 for the rest of the app session hung forever, so the app looked "logged in but nothing loads" until a full restart. See historical bug SE-13836 (`.plans/completed/refresh-token-zombie-session/` if archived, or `.plans/refresh-token-zombie-session/` if not yet).

`AuthTokenClient` maps a genuine HTTP 400/401 rejection from `/oauth/token` to `UnauthorizedFailure`; `LoginRepositoryImpl.refreshToken()` forces `AuthenticationCubit.logOut()` when it sees that specific failure type, so a truly-dead refresh token redirects the user to login instead of leaving them in the zombie state above. Transient failures (network error, timeout, 5xx) do NOT force logout — they're left to fail silently and retry on the next request.

## Forgot Password

Handled via external URL: `https://admin.{basePath}/users/password/new` — opened in system browser.
