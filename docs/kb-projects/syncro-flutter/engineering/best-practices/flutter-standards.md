# Flutter Project Standards

Universal quality standards that apply to every Flutter/Dart project regardless of architecture or libraries used. These are non-negotiable defaults — document exceptions in the project's `AGENTS.md`.

---

## Localization

**All user-facing strings must be localized.** Never hardcode display text in widgets.

```dart
// Wrong
Text('Welcome back')

// Right
Text(context.l10n.welcomeBack)
// or Text(AppLocalizations.of(context)!.welcomeBack)
```

How this is implemented varies — check the project KB for the specific setup (`flutter_localizations`, `intl`, `easy_localization`, etc.).

---

## Code Safety

**No `!` (null assertion) on expressions that can reasonably be null.** Use `??`, `?.`, or explicit null checks.

```dart
// Wrong
final name = user!.name;

// Right
final name = user?.name ?? '';
```

**No dynamic types without justification.** Every variable, parameter, and return type should be explicitly typed.

**No `// ignore:` lint suppressions** without a comment explaining why and a plan to fix.

---

## Async / Concurrency

**No `async` functions that silently swallow errors.** Always handle `Future` rejections.

```dart
// Wrong
Future<void> loadData() async {
  final result = await api.fetchData();
}

// Right
Future<void> loadData() async {
  try {
    final result = await api.fetchData();
    ...
  } catch (e, stack) {
    log('Failed to load data', error: e, stackTrace: stack);
    state = ErrorState(e.toString());
  }
}
```

**No blocking isolate work on the main isolate.** Use `compute()` or `Isolate.spawn` for CPU-heavy tasks.

---

## Widget Architecture

**No business logic in widgets.** Delegate to a ViewModel, BLoC, Notifier, or Controller — depending on the project's state management choice.

**One public widget class per file.** Name the file after the widget in `snake_case` (`user_card.dart`, not `UserCard.dart`).

**No direct `setState` calls in deeply nested widget trees.** Lift state up or use a state management solution.

---

## Error Handling

**No empty `catch` blocks.** Every caught error must be logged or surfaced to the user.

```dart
// Wrong
} catch (e) {}

// Right
} catch (e, stack) {
  log('Error', error: e, stackTrace: stack);
  _errorState = e.toString();
}
```

**Use sealed classes or `Either<Failure, T>`** for expected failure states rather than throwing exceptions as control flow.

---

## Security

**No secrets in source code or version control.** Use `--dart-define`, `flutter_dotenv`, or a secrets manager.

**No logging of sensitive data** (tokens, PII, passwords) in release builds.

**Use `flutter_secure_storage`** for sensitive local data — never plain `SharedPreferences` for tokens or credentials.

---

## Testing

**Unit tests for business logic, use cases, and state management classes.** At minimum, test happy path + error state.

**Widget tests for non-trivial UI components.** Use `flutter_test` and `WidgetTester`.

**No production logic changes without a corresponding test update.**
