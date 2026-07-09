# Rule: SafeArea, FittedBox, AspectRatio

Three helper widgets; each solves a distinct class of overflow / breakage.

## SafeArea

**Must** be present at the page root; otherwise widgets slide under the notch, status bar, or home indicator.

```dart
Scaffold(
  body: SafeArea(
    child: ...,
  ),
)
```

- `Scaffold` already applies insets for `AppBar` and `bottomNavigationBar`. `SafeArea` is still needed for the `body`.
- Manual math with `MediaQuery.of(context).padding.top` is **forbidden** — `SafeArea` works more correctly.
- A minimum padding can be enforced with `SafeArea(minimum: EdgeInsets.only(bottom: 16))`.

## AspectRatio

When an image or card must stay at a fixed ratio:

```dart
AspectRatio(
  aspectRatio: 16 / 9,
  child: Image.network(url, fit: BoxFit.cover),
)
```

- `aspectRatio = width / height`. `16 / 9` = `1.78`.
- The parent must be bounded (width or height known). `AspectRatio` inside a `Column` works because the `Column` gives the child a bounded width.
- `AspectRatio` doesn't work inside a `ListView` (unbounded height) — bound it with `SizedBox(height: ...)` or `Expanded`.

## FittedBox

Fits the content to the parent. Often used in the **wrong** place.

```dart
FittedBox(
  fit: BoxFit.scaleDown,
  child: Text('A very long title'),
)
```

- `BoxFit.scaleDown`: shrinks if it doesn't fit, leaves it as-is if it does.
- `BoxFit.contain`: always scales to fit the parent (grows it too if it's small).

**Wrong usage:** for `Text` overflow. Instead of shrinking the `Text`, clip it with `overflow: TextOverflow.ellipsis`:

```dart
// Usually BETTER
Expanded(
  child: Text(
    longText,
    overflow: TextOverflow.ellipsis,
    maxLines: 1,
  ),
)
```

Use `FittedBox` only for elements that **carry size information** (numeric indicators, a logo). Shrinking long text hurts readability.

## IntrinsicHeight / IntrinsicWidth

Expensive (O(n²)) widgets. Use only when siblings inside a `Row` must take the same height and there's no alternative.

## Review tags

- `[SAFEAREA-MISSING]` — no `SafeArea` at the page root.
- `[ASPECT-RATIO]` — an image/card ratio doesn't stay fixed and distorts with the container.
- `[FITTEDBOX-MISUSE]` — shrinking long text with `FittedBox` (usually `ellipsis` is better).
