# OAuth Login Integration — syncro-flutter

**Last Updated**: April 2026

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

## ⚠️ Token Refresh Gap

`AuthenticationCubit._refreshToken()` is not implemented — it always returns `true`. The `RestNetworkServiceImpl` intercepts 401s and calls `refreshToken()`, but the callback does not actually refresh the token. This is a known critical issue.

## Forgot Password

Handled via external URL: `https://admin.{basePath}/users/password/new` — opened in system browser.
