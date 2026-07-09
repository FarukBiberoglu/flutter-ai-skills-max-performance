# Rule: Breakpoints and LayoutBuilder

A single layout doesn't fit every device. Tablet, foldable, web, and split-screen scenarios need separate branching.

## MediaQuery vs LayoutBuilder

| Use | Preference |
|---|---|
| Size of the entire screen (status bar, navigation) | `MediaQuery.of(context).size` |
| The widget's **own** available space | `LayoutBuilder` → `constraints.maxWidth` |
| Orientation (portrait/landscape) | `MediaQuery.of(context).orientation` or `OrientationBuilder` |
| Is the keyboard open | `MediaQuery.of(context).viewInsets.bottom` |

**Rule:** When deciding a widget's layout, prefer `LayoutBuilder`. In split-screen, a side panel, or wherever it's placed in a small area, `MediaQuery` gives misleading information.

## Material 3 breakpoints

```
< 600       compact     (phone, portrait)
600 – 840   medium      (small tablet, foldable open, phone landscape)
> 840       expanded    (tablet, web, desktop)
```

## Typical usage

```dart
LayoutBuilder(
  builder: (context, constraints) {
    final width = constraints.maxWidth;
    if (width >= 840) {
      return _ExpandedLayout(); // master-detail
    }
    if (width >= 600) {
      return _MediumLayout(); // single column + wider padding
    }
    return _CompactLayout(); // single column, phone
  },
)
```

## Anti-pattern: magic numbers

```dart
// WRONG
if (MediaQuery.of(context).size.width > 768) { ... }
if (MediaQuery.of(context).size.width > 500) { ... }
```

Breakpoints must be defined in **one place**:

```dart
class Breakpoints {
  static const double compact = 600;
  static const double medium = 840;
}
```

## Orientation

Branching only on `MediaQuery.of(context).orientation` isn't enough; tablet landscape ≠ phone landscape. Check the width too.

## Review tags

- `[BREAKPOINT]` — no layout branching for tablet/web.
- `[ORIENTATION]` — no `OrientationBuilder` / `LayoutBuilder`, only portrait assumed.
- `[MEDIAQUERY-MISUSE]` — the widget decides based on screen width instead of its own area.
