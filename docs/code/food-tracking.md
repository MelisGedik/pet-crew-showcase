# Food tracking: exact stock, visible progress

[Back to the engineering case study](../engineering.md)

## Problem and decision

Owners need to subtract portions repeatedly, including very small bird portions. The app stores food amounts as integer milligrams instead of repeatedly subtracting floating-point kilograms. Input accepts either a comma or a full stop as a decimal separator. This keeps the stored quantity exact at the supported precision.

The remaining/original quantity ratio also drives a pixel container window. The container's food level drops from the top as stock is used, making the amount easier to scan.

## Actual source excerpts

From `lib/features/food/domain/food_batch.dart`:

```dart
/// Integer milligrams keep repeated subtractions exact, including small bird portions.
int? parseFoodWeight(String text, String unit) {
  if (!{'g', 'kg'}.contains(unit)) return null;
  final value = text.trim().replaceAll(',', '.');
  if (!RegExp(r'^\d{1,7}(?:\.\d{1,6})?$').hasMatch(value)) return null;
  final parts = value.split('.');
  final digits = unit == 'kg' ? 6 : 3, factor = unit == 'kg' ? 1000000 : 1000;
  if (parts.length > 1 && parts[1].length > digits) return null;
  final mg =
      int.parse(parts[0]) * factor +
      int.parse(parts.length == 1 ? '0' : parts[1].padRight(digits, '0'));
  return mg <= 1000000000 ? mg : null;
}

String foodWeight(int mg, String unit) {
  final digits = unit == 'kg' ? 6 : 3, factor = unit == 'kg' ? 1000000 : 1000;
  final fraction = (mg % factor)
      .toString()
      .padLeft(digits, '0')
      .replaceFirst(RegExp(r'0+$'), '');
  return '${mg ~/ factor}${fraction.isEmpty ? '' : '.$fraction'}';
}
```

From the nested `window` drawing function in `lib/features/food/pixel_food_container.dart`:

```dart
      final level = fraction == 0
          ? 0
          : (h * fraction).round().clamp(1, h.toInt());
      final top = y + h - level;
      canvas.save();
      canvas.clipRect(
        Rect.fromLTWH(x, top, w, level.toDouble()),
        doAntiAlias: false,
      );
      rect(x, top, w, level.toDouble(), food);
```

The drawing fragment relies on its surrounding painter, palette, `rect` helper and clamped stock fraction. The rest of the painter, UI and persistence code are intentionally omitted. These excerpts do not form a standalone application.

## Validation and limits

The full project's food tests cover stock changes and repository behavior. The visual fraction represents recorded stock; it does not measure the bag or recommend feeding quantities. Owners enter changes themselves.
