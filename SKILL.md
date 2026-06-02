---
name: myu-flutter
description: >
  Personal Flutter/Dart coding standards for Muhammad Yunus — Mobile Lead developer (iOS, Android, Flutter Desktop).
  Auto-trigger whenever the user is writing, reviewing, generating, or discussing Flutter or Dart code, creating new
  features, fixing bugs, or setting up a Flutter project. Apply these conventions silently and consistently without
  asking — they override any generic Flutter advice. Also trigger on requests like "create a Bloc", "add a repository",
  "write a test", "create a model", "set up DI", or any task that touches the project's architecture layers.
---

# myu-flutter — Personal Flutter Standards

These are Muhammad Yunus's personal conventions extracted from real projects (damda_seller, demand_seller, cmed, etc.).
Apply them to every Flutter task without being asked. When a project has its own `CLAUDE.md`, that wins on any conflict.

---

## About this developer

- Mobile Lead, Flutter developer.
- Platforms: **iOS, Android, Flutter Desktop only.**
- No backend, no Flutter Web — don't generate those unless explicitly asked.

---

## Default tech stack

- **Architecture**: Clean Architecture — `presentation / application / domain / infrastructure`.
- **State management**: Bloc + Freezed (new projects). Riverpod + Freezed (older projects — keep existing pattern, never migrate without asking).
- **HTTP**: `dio` with interceptors (`TokenInterceptor → AuthInterceptor → ConnectivityInterceptor → DebugInterceptor → LogInterceptor`).
- **Backend**: REST API + Firebase (Auth, Firestore, FCM).
- **DI**: `get_it` — see `references/infrastructure.md`.
- **IDE**: Android Studio / IntelliJ. Source control: GitHub.

---

## Folder structure

```
lib/
  app_constants.dart        ← AppConstants (all compile-time constants)
  main.dart
  application/              ← Blocs, feature-first
  domain/
    di/dependency_manager.dart   ← GetIt + top-level getters
    handlers/                    ← ApiResult, HttpService, interceptors
    interface/                   ← abstract facades only
  infrastructure/
    local_storage/               ← LocalStorage + StorageKeys
    models/                      ← plain Dart data models
    repository/                  ← facade implementations
    services/                    ← AppLogger, AppHelpers, enums, tr_keys
  presentation/
    app_assets.dart
    app_widget.dart
    components/
    pages/                       ← feature-first
    route/app_route.dart
    style/style.dart             ← CustomStyle + theme
```

Rules: `domain/` must not import Flutter or any external package. No business logic in `presentation/`.

---

## Code style

- Follow Effective Dart; keep `flutter analyze` clean (zero warnings).
- `const` constructors everywhere possible.
- Trailing commas in all widget trees and multi-line argument lists.
- Files: `snake_case.dart`. Classes: `UpperCamelCase`. Members: `lowerCamelCase`.
- Package imports (`package:app/...`) over relative imports.
- No `dynamic`; sound null safety. Avoid nullable where a sane default exists.
- Extract sub-widgets into separate classes — not helper methods returning `Widget`.
- Freezed for State/Event/ApiResult only. Data models use plain Dart (manual `fromJson/toJson/copyWith`).
- No business logic in widgets — only in Bloc / use cases.
- No `print()` or `debugPrint()` — use `AppLogger` (see Logging section).

---

## Error handling — ApiResult\<T>

Every repository method returns `Future<ApiResult<T>>`.

```dart
// domain/handlers/api_result.dart
@freezed
class ApiResult<T> with _$ApiResult<T> {
  const factory ApiResult.success({required T data}) = Success<T>;
  const factory ApiResult.failure({
    required String error,
    required int statusCode,
  }) = Failure<T>;
}
```

Repository template:

```dart
@override
Future<ApiResult<OrderModel>> getOrder(int id) async {
  try {
    final client = dioHttp.client(requireAuth: true);
    final response = await client.get('/api/v1/orders/$id');
    return ApiResult.success(data: OrderModel.fromJson(response.data));
  } catch (e) {
    AppLogger.e('getOrder failure', error: e);
    return ApiResult.failure(
      error: AppHelpers.errorHandler(e),
      statusCode: NetworkExceptions.getDioStatus(e),
    );
  }
}
```

- Every method has `try/catch`.
- Void responses: `const ApiResult.success(data: null)` — type `ApiResult<void>`.
- Never leak `DioException` to presentation layer.
- In Bloc: `result.when(success: ..., failure: ...)`.

---

## Logging — AppLogger

```dart
// lib/infrastructure/services/app_logger.dart
class AppLogger {
  AppLogger._();
  static final Logger _logger = Logger(
    filter: _AppLogFilter(),
    printer: PrettyPrinter(
      methodCount: 0, errorMethodCount: 6,
      lineLength: 100, colors: true, printEmojis: false,
      dateTimeFormat: DateTimeFormat.onlyTimeAndSinceStart,
    ),
  );
  static void d(Object? msg, {Object? error, StackTrace? stackTrace}) => _logger.d(msg, error: error, stackTrace: stackTrace);
  static void i(Object? msg, {Object? error, StackTrace? stackTrace}) => _logger.i(msg, error: error, stackTrace: stackTrace);
  static void w(Object? msg, {Object? error, StackTrace? stackTrace}) => _logger.w(msg, error: error, stackTrace: stackTrace);
  static void e(Object? msg, {Object? error, StackTrace? stackTrace}) => _logger.e(msg, error: error, stackTrace: stackTrace);
}
class _AppLogFilter extends LogFilter {
  @override bool shouldLog(LogEvent event) => kDebugMode;
}
```

Use: `AppLogger.e('getOrder failure', error: e)` in catch blocks. Silent in release builds.

---

## Commit messages

Follow Conventional Commits: `<type>(<scope>): <summary>`

| Type | Use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | No behavior change |
| `style` | Formatting only |
| `chore` | Build / deps / config |
| `test` | Tests |
| `docs` | Docs only |
| `perf` | Performance |

Rules: lowercase, imperative mood, no period, max 72 chars. Example: `feat(auth): add biometric login`.
Breaking change: add `!` — `refactor(api)!: rename response models`.
Never: `fix bug`, `update`, `changes`, `wip`.

---

## PR description template

```markdown
## What
## Why
## Changes
-
## Screenshots / recordings
## Checklist
- [ ] flutter analyze clean
- [ ] Tested on iOS
- [ ] Tested on Android
- [ ] No hardcoded strings / magic values
- [ ] No print() statements
```

---

## Documentation

**Write dartdoc on:** Bloc classes (scope + lifecycle), non-obvious event handlers, custom exceptions/interceptors, non-trivial parameters, constant groups.

**Skip dartdoc on:** Widget `build()`, repository impls that mirror the facade, self-explanatory CRUD methods, generated files.

Style: English, verb/noun phrase (no "This class..."), max 3 lines class-level, 1-2 lines method/param. Use `//` for single-line "why" comments only.

---

## Reference files

Load these when the task needs detailed patterns:

| File | Load when |
|---|---|
| `references/bloc.md` | Writing or reviewing Bloc (State, Event, handler, BlocBuilder/Listener/Consumer) |
| `references/infrastructure.md` | DI setup, HttpService, LocalStorage, Navigation, AppConstants |
| `references/testing.md` | Writing tests (bloc_test, mocktail) |
| `references/ui.md` | UI widgets, CustomStyle, screenutil, Freezed models, file upload, localization |
