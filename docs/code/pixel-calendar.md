# A date picker that belongs to the interface

[Back to the engineering case study](../engineering.md)

## Problem and decision

Typed dates were awkward on a phone: a numeric keyboard did not reliably expose date separators. A reusable date picker replaced manual entry while keeping the app's handheld-game visual style.

Dates display as year–month–day. The picker returns a value only after confirmation, so canceling does not change the form. Past-date fields and appointment fields can use different future-date rules. The calendar grid also adapts its cell height to text scaling.

## Actual source excerpt

Dialog entry point from `lib/design_system/pixel_date_picker.dart`:

```dart
/// Date-only picker: cancellation never changes the caller's value.
Future<DateTime?> showPixelDatePicker({
  required BuildContext context,
  required String title,
  DateTime? initialDate,
  DateTime? today,
  bool allowFuture = false,
}) {
  FocusManager.instance.primaryFocus?.unfocus();
  final now = today ?? DateTime.now();
  return showDialog<DateTime>(
    context: context,
    builder: (_) => _PixelCalendar(
      title: title,
      initial: initialDate,
      currentDate: DateTime(now.year, now.month, now.day),
      lastDate: allowFuture
          ? DateTime(
              math.max(now.year + 100, initialDate?.year ?? now.year),
              12,
              31,
            )
          : DateTime(now.year, now.month, now.day),
    ),
  );
}
```

The private `_PixelCalendar` widget, localized labels, design system and caller are intentionally omitted. This excerpt illustrates the dialog contract rather than providing a runnable calendar or the full app.

## Validation and limits

The full project has widget tests for date selection and record forms, including compact/enlarged-text layouts. Date selection uses local calendar dates; notification delivery is handled separately by the Android scheduling layer.
