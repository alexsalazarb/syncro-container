# Flutter Project Standards (Compact)

Non-negotiable quality defaults for every Flutter/Dart project. Covers localization, code safety, async, widget architecture, errors, security, and testing. Document exceptions in `AGENTS.md`.

## Rules at a Glance

| Area | Rule |
|------|------|
| Localization | All user-facing strings via `context.l10n.*` — never hardcoded `Text('...')` |
| Null safety | No `!` on nullable expressions; use `??`, `?.`, or explicit checks |
| Types | No `dynamic` without justification; explicitly type everything |
| Lint suppression | No `// ignore:` without explanation comment and fix plan |
| Async errors | Every `async` function must handle failures — no silent `Future` rejections |
| Isolates | CPU-heavy work in `compute()` or `Isolate.spawn`, not main isolate |
| Widgets | No business logic in widgets; delegate to ViewModel/BLoC/Notifier |
| File naming | One public widget per file, `snake_case.dart` |
| State | No deep `setState`; lift state up or use state management |
| Errors | No empty `catch` blocks; log with `error:` and `stackTrace:` params |
| Failure states | Use sealed classes or `Either<Failure, T>` over exceptions as control flow |
| Secrets | No secrets in source; use `--dart-define`, `flutter_dotenv`, or secrets manager |
| Storage | `flutter_secure_storage` for tokens — never plain `SharedPreferences` |
| Testing | Unit tests for business logic (happy + error); widget tests for non-trivial UI |

## Key Code Patterns

```dart
// Safe null
final name = user?.name ?? '';

// Async error handling
try { final result = await api.fetchData(); }
catch (e, stack) { log('Failed', error: e, stackTrace: stack); }
```

## Gotchas

- `dynamic` types bypass the analyzer — bugs surface only at runtime
- No production logic changes without a corresponding test update
- Sensitive data in `SharedPreferences` is stored as plaintext on disk

> **Read full doc when** you need detailed code examples for widget architecture, async patterns, or security setup.
