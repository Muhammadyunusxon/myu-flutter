# UI, Models, Upload, Localization

## Freezed conventions

**Data models** — plain Dart, NOT Freezed:

```dart
class OrderModel {
  final int id;
  final String status;
  final double? totalPrice;

  const OrderModel({required this.id, required this.status, this.totalPrice});

  factory OrderModel.fromJson(Map<String, dynamic> json) => OrderModel(
    id: json['id'] as int,
    status: json['status'] as String,
    totalPrice: (json['total_price'] as num?)?.toDouble(),
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'status': status,
    if (totalPrice != null) 'total_price': totalPrice,
  };

  OrderModel copyWith({int? id, String? status, double? totalPrice}) =>
      OrderModel(id: id ?? this.id, status: status ?? this.status,
                 totalPrice: totalPrice ?? this.totalPrice);
}
```

- Freezed only for: Bloc State/Event, `ApiResult<T>`.
- List fields default to `[]`, not `null`.
- Omit null fields from `toJson` with `if (field != null) 'key': field`.
- No `@JsonSerializable` unless the project already uses it.

---

## UI conventions

### Colors — CustomStyle

Never write `Color(0xFF...)` in a widget. Always use `CustomStyle`:

```dart
color: CustomStyle.primary       // correct
color: Color(0xFFF19204)         // wrong
```

### Text styles

```dart
CustomStyle.interExtraBold(size: 20, color: CustomStyle.black)  // w800
CustomStyle.interBold(size: 18, color: CustomStyle.black)       // w700
CustomStyle.interSemi(size: 16, color: CustomStyle.black)       // w600
CustomStyle.interNormal(size: 14, color: CustomStyle.black)     // w500
CustomStyle.interRegular(size: 14, color: CustomStyle.black)    // w400
```

Font is `GoogleFonts.nunitoSans` via CustomStyle — don't use `GoogleFonts.*` directly.

### Sizing (flutter_screenutil)

```dart
16.sp   // font sizes
12.r    // border radii, icon sizes, symmetric padding
80.w    // widths
48.h    // heights
```

Never raw `double` literals for sizes — always suffix `.sp / .r / .w / .h`.

### Shadows & gradients

```dart
CustomStyle.primaryShadow   // primary-colored glow
CustomStyle.greyShadow      // neutral card shadow
CustomStyle.walletGradient  // branded gradient surface
```

### Icons & assets

- Icons: `Remix.icon_name` from `remixicon` package.
- SVG: `SvgPicture.asset(AppAssets.fooIcon)`.
- Asset paths in `lib/presentation/app_assets.dart` as `static const String` — never hardcode inline.

### Opacity

`color.withValues(alpha: 0.5)` — not the deprecated `color.withOpacity(0.5)`.

---

## Localization (TrKeys)

```dart
// lib/infrastructure/services/tr_keys.dart
abstract class TrKeys {
  TrKeys._();
  static const String save   = 'save';
  static const String cancel = 'cancel';
  static const String orderStatus = 'order.status';  // dot notation
}
```

Usage:
```dart
Text(AppHelpers.getTranslation(TrKeys.save))
UiHelper.errorSnackBar(context, text: AppHelpers.getTranslation(TrKeys.noInternetConnection));
```

- Keys loaded from backend at startup, cached in `LocalStorage`.
- Fallback: user locale → English → empty string.
- Always reference `TrKeys` constants — never pass raw strings to `getTranslation`.
- Add constant to `TrKeys` first, then use it.

---

## File upload (FormData)

Single file:
```dart
final data = FormData.fromMap({
  'image': await MultipartFile.fromFile(filePath),
  'type': uploadType.name,
});
final client = dioHttp.client(requireAuth: true);
final response = await client.post('/api/v1/dashboard/galleries', data: data);
```

Multiple files:
```dart
final data = FormData.fromMap({
  for (int i = 0; i < files.length; i++)
    'images[$i]': await MultipartFile.fromFile(files[i]),
  'type': uploadType.name,
});
```

- Upload type is always an `enum` — never raw strings.
- Always `requireAuth: true`.
- Multiple files: indexed bracket notation `images[0]`, `images[1]`.
- Response goes through `ApiResult<T>` like any other call.
- Don't upload from presentation layer — always delegate to a repository.
