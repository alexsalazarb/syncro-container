# Task: infinite_scroll_pagination 4→5 Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 2 — API Rewrites
**Task ID**: task-03
**Task Path**: phase-2/task-03-infinite-scroll-pagination
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Migrar `infinite_scroll_pagination` de la versión 4 a la versión 5. La v5 es una reescritura arquitectural completa: `PagingController` fue rediseñado y `PagedLayoutBuilder` ya no acepta `pagingController` directamente.

## Context

**Breaking changes v5:**
- `appendPage()`, `appendLastPage()`, `retryLastFailedRequest()` **removidos** de `PagingController`
- `firstPageKey`, `itemList`, `error`, `nextPageKey` getters/setters **removidos** de `PagingController`
- Ahora usa `PagingState.copyWith()` para mutación de estado
- `PagedLayoutBuilder` ya no acepta `pagingController` — acepta `state: PagingState` + `fetchNextPage: VoidCallback`
- Nuevo widget `PagingListener` requerido para conectar cubit/bloc con el pager

**10 archivos afectados** (5 features: alerts, assets, appointments, tickets):

```
lib/features/alerts/application/alerts_cubit.dart
lib/features/alerts/presentation/alerts_view.dart
lib/features/assets/application/assets_cubit.dart
lib/features/assets/presentation/assets_view.dart
lib/features/appointments/appointments_home/application/appointments_list_cubit.dart
lib/features/ticket/ticket_home/application/ticket_cubit.dart
lib/features/ticket/ticket_home/domain/ticket_state.dart
lib/features/ticket/ticket_home/presentation/ticket_view.dart
lib/features/appointments/appointments_home/presentation/widget/appointments_paginated_page.dart
lib/features/ticket/ticket_home/presentation/widget/ticket_filter_bar.dart
```

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/alerts/application/alerts_cubit.dart` | modify | Migrar PagingController usage |
| `syncro-flutter/lib/features/alerts/presentation/alerts_view.dart` | modify | Migrar PagedListView |
| `syncro-flutter/lib/features/assets/application/assets_cubit.dart` | modify | Migrar PagingController usage |
| `syncro-flutter/lib/features/assets/presentation/assets_view.dart` | modify | Migrar PagedListView |
| `syncro-flutter/lib/features/appointments/appointments_home/application/appointments_list_cubit.dart` | modify | Migrar PagingController usage |
| `syncro-flutter/lib/features/ticket/ticket_home/application/ticket_cubit.dart` | modify | Migrar PagingController usage |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket_state.dart` | modify | PagingController en estado |
| `syncro-flutter/lib/features/ticket/ticket_home/presentation/ticket_view.dart` | modify | Migrar PagedListView |
| `syncro-flutter/lib/features/appointments/appointments_home/presentation/widget/appointments_paginated_page.dart` | modify | Migrar PagedListView |
| `syncro-flutter/lib/features/ticket/ticket_home/presentation/widget/ticket_filter_bar.dart` | modify | PagingController reference |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- Archivos owned by task-02 (notifications_manager.dart)

## Implementation Steps

### Step 1: Leer todos los archivos afectados

Leer cada uno de los 10 archivos para entender el patrón actual de uso de `PagingController`.

### Step 2: Entender la nueva arquitectura v5

En v5, el patrón típico con BLoC/Cubit es:

**Cubit (application layer):**
```dart
// ANTES (v4):
final PagingController<int, Item> pagingController = PagingController(firstPageKey: 1);
// en el constructor: pagingController.addPageRequestListener(_fetchPage);
// en _fetchPage: pagingController.appendPage(items, nextKey) o appendLastPage(items)

// DESPUÉS (v5):
PagingState<int, Item> _pagingState = const PagingState();
// Exponer state vía getter/stream
// fetchPage() retorna un nuevo PagingState via copyWith()
void fetchNextPage() {
  // load data...
  emit(state.copyWith(pagingState: _pagingState.copyWith(pages: [...])));
}
```

**View (presentation layer):**
```dart
// ANTES (v4):
PagedListView<int, Item>(
  pagingController: cubit.pagingController,
  builderDelegate: PagedChildBuilderDelegate<Item>(itemBuilder: ...),
)

// DESPUÉS (v5):
PagingListener(
  controller: pagingController, // or state-based
  child: PagedListView<int, Item>(
    state: pagingState,
    fetchNextPage: cubit.fetchNextPage,
    builderDelegate: PagedChildBuilderDelegate<Item>(itemBuilder: ...),
  ),
)
```

### Step 3: Migrar cada feature

Para cada feature (alerts, assets, appointments, tickets):
1. Migrar el cubit: reemplazar `PagingController` con `PagingState` en el estado/cubit
2. Migrar la view: reemplazar `pagingController:` por `state:` + `fetchNextPage:` + `PagingListener` si aplica
3. Verificar que `flutter analyze` en esos archivos no tenga errores

### Step 4: Regenerar mocks si es necesario

Si algún cubit tiene mocks generados (mockito/mocktail), regenerar:
```bash
cd syncro-flutter
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Step 5: Verificar compilación

```bash
cd syncro-flutter
fvm flutter analyze lib/features/alerts/ lib/features/assets/ lib/features/appointments/ lib/features/ticket/ticket_home/
```

## Testing

- [ ] `fvm flutter analyze` sin errores en los 10 archivos
- [ ] Tests existentes de cubits paginados pasan (si los hay)
- [ ] Regenerar mocks si el build_runner los genera para estos cubits

## Completion Criteria

- [ ] Los 10 archivos migrados a la API v5 de infinite_scroll_pagination
- [ ] No hay uso de `appendPage`, `appendLastPage`, `pagingController:` como parámetro de PagedListView
- [ ] `fvm flutter analyze` sin errores en las 5 features afectadas
- [ ] Status actualizado en `status.md`
