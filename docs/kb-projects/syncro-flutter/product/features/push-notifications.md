# Push Notifications Feature — syncro-flutter

**Last Updated**: April 2026

## Context

Firebase Cloud Messaging (FCM) delivers push notifications to technicians. Tapping a notification deep-links into the relevant app screen.

## Service

`NotificationManager` is a singleton that wraps FCM via `firebase_messaging`. It must be initialized sequentially after GetIt setup:

```dart
await NotificationManager.instance.init();
await NotificationManager.instance.register();
```

`init()` must complete before `register()` — this ordering is enforced in `main.dart`.

## Notification Sources

| Source String | Destination | Tab Index |
|--------------|------------|-----------|
| `appointment` | `AppRoute.appointmentUpdate` (with `int` appointmentId) | 1 |
| `ticket` | `AppRoute.ticketDetail` (with `TicketDetailsParams`) | 4 |
| `comment` | `AppRoute.ticketDetail` (ticket ID parsed from URL last segment) | 4 |
| `rmmAlert` | `AppRoute.alerts` | 0 |
| `openStruct` | `AppRoute.chatDetail` (async — waits for WebSocket) | 2 |

## Deep Link Flow

`RouteCubit.redirectOnNotification(PushNotification)` is the entry point:

1. Parse `notification.source` → `NotificationSource` enum
2. Build `CurrentRoute` with target route + params + tab index
3. Call `goToCurrentRoute(showInternalPage: true)`:
   - Navigate to home tab first (with `tabIndex` query param)
   - Wait 900ms for tab to initialize
   - Push the specific detail route

## Chat Notification (openStruct) Special Flow

Chat notifications require the WebSocket to be initialized before navigation can happen (to look up the chat by ID from the socket stream):

1. Wait for `ChatWebSocketService.isInitialized`
2. Subscribe to `chatsSocketStream`
3. Find the chat matching `notification.objectId`
4. Fetch the associated asset
5. Build `ChatDetailParameters` → navigate to chat detail

The socket subscription is cancelled after successful navigation or if chat is not found.

## Background App Launch

When the app is launched cold from a notification, `RouteCubit` stores the `CurrentRoute` and `goToCurrentRoute()` re-checks authentication before navigating. If not authenticated, redirects to login.

## Logout

`NotificationManager.instance.logout()` is called in `AuthenticationCubit.logOut()` to unregister the device token.
