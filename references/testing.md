# Testing

## Stack

- `flutter_test` — core
- `bloc_test` — `blocTest<B, S>()` helper
- `mocktail` — mocking (not mockito)

## File location

Mirror `lib/` under `test/`:
```
test/
  application/orders/orders_bloc_test.dart
  infrastructure/models/order_model_test.dart
```

## Bloc test template

```dart
class _MockOrdersRepo extends Mock implements OrdersFacade {}

void main() {
  setUpAll(() async {
    TestWidgetsFlutterBinding.ensureInitialized();
    SharedPreferences.setMockInitialValues({});
    await LocalStorage.init();
    registerFallbackValue(OrderModel(id: 0, status: ''));
  });

  late _MockOrdersRepo repo;
  setUp(() => repo = _MockOrdersRepo());

  group('_Fetched', () {
    blocTest<OrdersBloc, OrdersState>(
      'success → emits isLoading:true then loaded list',
      setUp: () {
        when(() => repo.getOrders(page: any(named: 'page')))
            .thenAnswer((_) async => ApiResult.success(
                data: OrdersResponse(data: [OrderModel(id: 1, status: 'new')])));
      },
      build: () => OrdersBloc(ordersRepo: repo),
      act: (bloc) => bloc.add(const OrdersEvent.fetched(isRefresh: true)),
      expect: () => [
        const OrdersState(orders: [], isLoading: true),
        isA<OrdersState>()
            .having((s) => s.isLoading, 'isLoading', false)
            .having((s) => s.orders, 'orders', hasLength(1)),
      ],
    );

    blocTest<OrdersBloc, OrdersState>(
      'failure → emits isLoading:false',
      setUp: () {
        when(() => repo.getOrders(page: any(named: 'page')))
            .thenAnswer((_) async =>
                const ApiResult.failure(error: 'server error', statusCode: 500));
      },
      build: () => OrdersBloc(ordersRepo: repo),
      act: (bloc) => bloc.add(const OrdersEvent.fetched()),
      expect: () => [
        const OrdersState(isLoading: true),
        const OrdersState(isLoading: false),
      ],
    );
  });
}
```

## Rules

- Mock only the **facade** (interface), never the repository impl.
- One `group` per event type; multiple cases per group.
- `isA<State>().having(...)` for partial assertions — don't compare full state when only a few fields matter.
- `setUpAll` for global one-time init; `setUp` for fresh mock instances per test.
- Test file suffix: `_test.dart`.
- Don't test private methods — test observable state transitions only.
- Inject dependencies via constructor (not `getIt`) so they can be mocked:
  ```dart
  OrdersBloc({OrdersFacade? ordersRepo})
      : _repo = ordersRepo ?? ordersRepository,
        super(const OrdersState());
  ```
