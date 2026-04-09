# Coding Conventions — syncro-flutter

**Last Updated**: April 2026

## Context

Conventions observed across the syncro-flutter codebase. Follow these when adding or modifying code.

---

## File & Directory Naming

- Files: `snake_case` — `ticket_details_page.dart`, `get_ticket_by_id_usecase.dart`
- Directories: `snake_case` — `ticket_details/`, `infrastructure/`
- Test files: mirror source path + `_test.dart` suffix

> **Known debt**: One directory has a space in its name: `ticket/ticket_custom_fields/edit custom fields/`. Do not replicate this pattern — use underscores.

---

## Class Naming

| Type | Convention | Example |
|------|-----------|---------|
| Pages/Screens | `PascalCase` + `Page` | `TicketDetailsPage` |
| Widgets | `PascalCase` + `Widget` or `View` | `TicketCardWidget` |
| Cubits | `PascalCase` + `Cubit` | `TicketDetailsCubit` |
| BLoCs | `PascalCase` + `Bloc` | (not currently used) |
| States | `PascalCase` | `TicketDetailsLoaded`, `TicketDetailsError` |
| Repositories (interface) | `PascalCase` + `Repository` | `TicketsRepository` |
| Repositories (impl) | `PascalCase` + `RepositoryImpl` | `TicketsRepositoryImpl` |
| Use cases | `PascalCase` + `UseCase` | `GetTicketsUseCase` |
| Deserializers | `PascalCase` + `Deserializer` | `TicketByIdDeserializer` |
| Services | `PascalCase` + `Service` | `TimezoneService` |
| Params/Request types | `PascalCase` + `Params` | `GetTicketByIdParams` |
| Response types | `PascalCase` + `Response` | `GetTicketByIdResponse` |

---

## Cubit State Structure

States use `part` files and sealed classes (or Equatable subclasses):

```dart
// cubit file
part 'authentication_state.dart';

class AuthenticationCubit extends Cubit<AuthenticationState> {
  // ...
}

// state file (authentication_state.dart)
part of 'authentication_cubit.dart';

sealed class AuthenticationState extends Equatable { ... }
class AuthenticationLoading extends AuthenticationState { ... }
class AuthenticationAuthenticated extends AuthenticationState { ... }
class AuthenticationUnauthenticated extends AuthenticationState { ... }
```

**Always** use `safeEmit()` (from `core/utils/cubit_extension.dart`) instead of `emit()`:

```dart
safeEmit(AuthenticationAuthenticated(user: user));  // ✅
emit(AuthenticationAuthenticated(user: user));        // ❌
```

---

## Use Case Hierarchy

Pick the right base class from `core/usecases/usecase.dart`:

| Base Class | When |
|-----------|------|
| `UseCase<Type, Params>` | Async, with params, returns Either |
| `NoParamsUseCase<Type>` | Async, no params, returns Either |
| `VoidUseCase<Params>` | Async, with params, returns Either<Failure, void> |
| `VoidUseCaseNoParams` | Async, no params, returns Either<Failure, void> |
| `SyncUseCase<Type, Params>` | Synchronous |

---

## Repository Pattern

1. Define interface in `domain/`
2. Implement in `infrastructure/`
3. Register in widget tree via `AppRepositories` OR in GetIt via `service_locator.dart` (prefer widget tree for feature repos)

```dart
// domain/tickets_repository.dart
abstract class TicketsRepository {
  Future<Either<Failure, GetTicketsResponse>> getTickets(GetTicketsParams params);
}

// infrastructure/tickets_repository_impl.dart
class TicketsRepositoryImpl implements TicketsRepository {
  TicketsRepositoryImpl({required this.networkService});
  final NetworkService networkService;

  @override
  Future<Either<Failure, GetTicketsResponse>> getTickets(GetTicketsParams params) async {
    return networkService.sendRequestCall(
      request: AppRequests.getTickets.requestOption(),
      deserializer: TicketsDeserializer(),
      queryParameters: params.toQueryParams(),
    );
  }
}
```

---

## Async / Context Safety

Always check `mounted` after any `await` that uses `BuildContext`:

```dart
final result = await someAsyncCall();
if (!context.mounted) return;  // ✅ always check
Navigator.of(context).pop();
```

---

## Logging

Always use `logger()` from `core/utils/logger.dart`. Never use `print()` or `debugPrint()`:

```dart
import 'package:syncro/core/utils/logger.dart';

logger('Chat initialized');              // ✅
logger('Error: $e\nStack: $stack');      // ✅
print('something');                      // ❌
debugPrint('something');                 // ❌
```

---

## Error Handling in Cubits

Use `.fold()` on `Either` results. Log errors; never silently ignore failures:

```dart
final result = await useCase.call(params);
result.fold(
  (failure) {
    logger('Error fetching tickets: ${failure.message}');
    safeEmit(TicketError(failure.message));
  },
  (data) => safeEmit(TicketLoaded(data)),
);
```

---

## Widget Structure

Prefer stateless widgets with named private sub-widgets:

```dart
class TicketPage extends StatelessWidget {
  const TicketPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tickets')),
      body: const _TicketBody(),
    );
  }
}

class _TicketBody extends StatelessWidget {
  const _TicketBody();

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<TicketCubit, TicketState>(
      builder: (context, state) => switch (state) {
        TicketLoaded() => _TicketList(tickets: state.tickets),
        TicketError() => _ErrorView(message: state.message),
        _ => const CircularProgressIndicator(),
      },
    );
  }
}
```

---

## Testing Conventions

- Mock `NetworkService` using `@GenerateMocks([NetworkService])` with `mockito`
- Generate mocks: `flutter pub run build_runner build`
- Test files mirror source structure in `test/`
- Use `bloc_test` package for cubit tests
- Arrange / Act / Assert pattern with comments

```dart
@GenerateMocks([NetworkService])
void main() {
  group('TicketsRepositoryImpl', () {
    late MockNetworkService mockNetworkService;
    late TicketsRepositoryImpl repository;

    setUp(() {
      mockNetworkService = MockNetworkService();
      repository = TicketsRepositoryImpl(networkService: mockNetworkService);
    });

    test('should return tickets on success', () async {
      // Arrange
      when(mockNetworkService.sendRequestCall<GetTicketsResponse>(...))
          .thenAnswer((_) async => Right(expectedResponse));

      // Act
      final result = await repository.getTickets(params);

      // Assert
      expect(result, Right(expectedResponse));
    });
  });
}
```

---

## Const Constructors

Add `const` wherever possible to avoid unnecessary rebuilds:

```dart
child: const CircularProgressIndicator()  // ✅
child: CircularProgressIndicator()          // ❌ (if no dynamic data)
```

---

## Hardcoded Strings

The `.hardcoded` extension on `String` marks intentional hardcoded strings (not l10n candidates). All routes and debug labels should use `.hardcoded`.

```dart
GlobalKey<NavigatorState>(debugLabel: 'shellHome'.hardcoded)
```
