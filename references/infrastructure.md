# Infrastructure Patterns

## Dependency Injection (GetIt)

`lib/domain/di/dependency_manager.dart` — the single DI file.

```dart
final GetIt getIt = GetIt.instance;

Future<void> setUpDependencies() async {
  getIt.registerLazySingleton<HttpService>(HttpService.new);
  getIt.registerSingleton<AuthFacade>(AuthRepository());
  getIt.registerSingleton<OrdersFacade>(OrdersRepository());
  // all facades registered here
}

// Top-level getters — used in repositories
final dioHttp          = getIt.get<HttpService>();
final authRepository   = getIt.get<AuthFacade>();
final ordersRepository = getIt.get<OrdersFacade>();
```

- `HttpService` → `registerLazySingleton`.
- All facades → `registerSingleton` (eager).
- Never call `getIt.get<>()` in widgets or Blocs.
- `setUpDependencies()` called in `main()` before `runApp()`.

---

## HttpService

`lib/domain/handlers/http_service.dart` — fresh `Dio` instance per call.

```dart
class HttpService {
  Dio client({bool requireAuth = false}) =>
      Dio(BaseOptions(
        baseUrl: AppConstants.isDemo ? AppConstants.demoUrl : AppConstants.baseUrl,
        connectTimeout: const Duration(seconds: 60),
        receiveTimeout: const Duration(seconds: 60),
        sendTimeout:    const Duration(seconds: 60),
        headers: {'Accept': 'application/json', 'Content-type': 'application/json'},
      ))
      ..interceptors.add(TokenInterceptor(requireAuth: requireAuth))
      ..interceptors.add(AuthInterceptor())
      ..interceptors.add(ConnectivityInterceptor())
      ..interceptors.add(DebugInterceptor(debugLogService))
      ..interceptors.add(LogInterceptor(requestBody: true, responseBody: true, responseHeader: false));
}
```

- Never instantiate `Dio` directly — always use `dioHttp.client(requireAuth: true/false)`.
- Interceptor order is fixed: `Token → Auth → Connectivity → Debug → Log`.
- `requireAuth: false` for public endpoints; `true` for everything else.
- Timeouts: always 60 s for all three.

---

## LocalStorage

`lib/infrastructure/local_storage/local_storage.dart` — static abstract class.

```dart
abstract class LocalStorage {
  LocalStorage._();
  static SharedPreferences? _preferences;
  static const FlutterSecureStorage _secure = FlutterSecureStorage(
    iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
  );
  static String _tokenCache = ''; // sync cache for dio interceptor

  static Future<void> init() async {
    _preferences = await SharedPreferences.getInstance();
    await _hydrateToken();
  }

  static String getToken() => _tokenCache;
  static Future<void> setToken(String token) async {
    _tokenCache = token;
    await _secure.write(key: StorageKeys.keyToken, value: token);
  }
  static Future<void> deleteToken() async {
    _tokenCache = '';
    await _secure.delete(key: StorageKeys.keyToken);
  }
}
```

`lib/infrastructure/local_storage/storage_keys.dart`:

```dart
abstract class StorageKeys {
  StorageKeys._();
  static const String keyToken    = 'token';
  static const String keyLanguage = 'language';
  static const String keyUser     = 'user';
}
```

- Token in `FlutterSecureStorage`; everything else in `SharedPreferences`.
- `_tokenCache` for synchronous reads in the interceptor.
- `LocalStorage.init()` in `main()` after `setUpDependencies()`.
- Always use `StorageKeys` constants — never raw strings.

---

## AppConstants

`lib/app_constants.dart`:

```dart
class AppConstants {
  AppConstants._();

  static const bool isDemo       = false;
  static const bool appStoreMode = true;

  /// api urls
  static const String baseUrl = 'https://api.example.com/';
  static const String demoUrl = 'https://test-api.example.com/';

  /// Border radius tokens
  static const double radiusSm     = 8;
  static const double radius       = 12;
  static const double radiusLg     = 16;

  /// Animation duration tokens
  static const Duration animShort  = Duration(milliseconds: 150);
  static const Duration animMedium = Duration(milliseconds: 300);
  static const Duration animLong   = Duration(milliseconds: 500);
}
```

- `isDemo` switches base URL; `appStoreMode` hides dev-only features.
- Group constants with `///` comment headings.
- Design tokens (radius, durations) live here — don't scatter magic numbers.

---

## Navigation

`lib/presentation/route/app_route.dart` — single navigation hub.

```dart
abstract class AppRoute {
  const AppRoute._();

  static Future<void> goMain(BuildContext context) =>
      Navigator.of(context).pushAndRemoveUntil(MainPage.route(context), (_) => false);

  static Future<T?> goOrders<T>(BuildContext context, {required int shopId}) =>
      _push<T>(context, OrdersPage(shopId: shopId));

  static Future<T?> _push<T>(BuildContext context, Widget child) =>
      Navigator.of(context).push<T>(MaterialPageRoute<T>(builder: (_) => child));
}
```

Page `route()` factory (BlocProvider inside):

```dart
class OrdersPage extends StatefulWidget {
  static Route route(BuildContext context, {required int shopId}) =>
      MaterialPageRoute(
        builder: (_) => BlocProvider(
          create: (_) => OrdersBloc(),
          child: OrdersPage(shopId: shopId),
        ),
      );
}
```

- All navigation through `AppRoute` — no bare `Navigator.push` in widgets.
- `pushAndRemoveUntil` only for auth/splash stack-clearing transitions.
- Sub-widgets go in `pages/feature/widgets/`.
