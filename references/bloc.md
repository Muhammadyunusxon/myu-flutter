# Bloc Patterns

## File structure

```
application/orders/
  orders_bloc.dart       ← Bloc class + part declarations
  orders_event.dart      ← part of orders_bloc.dart
  orders_state.dart      ← part of orders_bloc.dart
  orders_bloc.freezed.dart
```

## State

```dart
part of 'orders_bloc.dart';

@freezed
abstract class OrdersState with _$OrdersState {
  const factory OrdersState({
    @Default(false) bool isLoading,
    @Default([]) List<OrderModel> orders,
    @Default(1) int page,
    OrderModel? selectedOrder,
  }) = _OrdersState;
}
```

- One state class per Bloc — never multiple subclasses.
- All fields have `@Default(value)` — no nullable fields where a sane default exists.
- Mutate only via `copyWith`.

## Event

```dart
part of 'orders_bloc.dart';

@freezed
abstract class OrdersEvent with _$OrdersEvent {
  const factory OrdersEvent.fetched(
    BuildContext context, {
    @Default(false) bool isRefresh,
    VoidCallback? onSuccess,
  }) = _Fetched;

  const factory OrdersEvent.selected(OrderModel order) = _Selected;
}
```

- Pass `BuildContext` only when the handler must show UI feedback.
- `onSuccess` / `onError` for side effects the widget needs.
- Event names: past-tense verbs — `fetched`, `selected`, `deleted`.

## Handler pattern

```dart
Future<void> _onFetched(_Fetched event, Emitter<OrdersState> emit) async {
  if (event.isRefresh) emit(state.copyWith(orders: [], page: 1));
  emit(state.copyWith(isLoading: true));

  final res = await ordersRepository.getOrders(page: state.page);
  res.when(
    success: (data) {
      emit(state.copyWith(orders: data.data ?? [], isLoading: false));
      event.onSuccess?.call();
    },
    failure: (msg, status) {
      emit(state.copyWith(isLoading: false));
      if (event.context.mounted) {
        UiHelper.errorSnackBar(event.context, text: msg);
      }
    },
  );
}
```

- `emit(isLoading: true)` before async; `isLoading: false` in both success and failure.
- Check `context.mounted` before any UI call after `await`.
- Error feedback (snackbar/toast) inside the Bloc handler — never in the widget.
- Never `emit` after emitter is closed.

## Bloc constructor

```dart
class OrdersBloc extends Bloc<OrdersEvent, OrdersState> {
  OrdersBloc() : super(const OrdersState()) {
    on<_Fetched>(_onFetched);
    on<_Selected>(_onSelected);
  }
}
```

## Widget consumption

| Widget | When |
|---|---|
| `BlocBuilder<B, S>` | Rebuild UI on state change |
| `BlocListener<B, S>` | Side effects only (navigation, snackbar) — no rebuild |
| `BlocConsumer<B, S>` | Both rebuild + side effects on the same state |

Narrow triggers with `buildWhen` / `listenWhen`:

```dart
BlocBuilder<OrdersBloc, OrdersState>(
  buildWhen: (prev, curr) => prev.isLoading != curr.isLoading,
  builder: (context, state) => ...,
)
```

Use `BlocProvider.value` for dialogs/bottom sheets that share an existing Bloc.
Never create a Bloc inside `build()` — always in `BlocProvider.create` or `route()`.
