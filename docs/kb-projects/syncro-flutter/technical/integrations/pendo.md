# Pendo Integration — syncro-flutter

**Last Updated**: April 2026

## Context

Pendo SDK is used for product analytics (screen tracking, user behavior). It is fire-and-forget — it must never block app initialization.

## Package

`pendo_sdk: ^3.7.1`

## Initialization

`PendoService` (registered in GetIt) is initialized **after** `runApp` using `unawaited()`:

```dart
unawaited(
  _retryOperation(
    operation: () => getIt<PendoService>().init(),
    operationName: 'Pendo SDK initialization',
  ),
);
```

**Critical**: Never `await` Pendo init in the startup pipeline. It is non-blocking by design.

## GoRouter Integration

The router is configured with Pendo listener automatically:

```dart
final _goRouter = GoRouter(...)..addPendoListenerToDelegate();
```

This registers the GoRouter navigation delegate with Pendo so screen changes are tracked automatically.

## Usage

Access `PendoService` through GetIt:

```dart
getIt<PendoService>().someMethod();
```

The Pendo key is loaded from the `.env` file via `Environment.pendoKey`.
