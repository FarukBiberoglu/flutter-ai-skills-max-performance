# Rule: Wrap or Row?

If many small widgets (chips, labels, buttons) are laid out side by side, a `Row` overflows. `Wrap` automatically moves to the next line when the row doesn't fit.

## Decision table

| Situation | Widget |
|---|---|
| A fixed number (2–4) of widgets, all must fit | `Row` + `Expanded` / `Flexible` |
| A variable number of widgets, move to the next line on overflow | `Wrap` |
| Should scroll horizontally | `SingleChildScrollView(scrollDirection: Axis.horizontal)` or `ListView.builder(scrollDirection: ...)` |
| Equal-column grid | `Wrap` or `GridView` |

## Wrap parameters

```dart
Wrap(
  spacing: 8,        // horizontal gap
  runSpacing: 4,     // vertical gap when moving to the next line
  alignment: WrapAlignment.start,
  crossAxisAlignment: WrapCrossAlignment.center,
  children: [...],
)
```

## Bad → Good

```dart
// WRONG
Row(
  children: categories.map((c) => Chip(label: Text(c))).toList(),
)

// CORRECT
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: categories.map((c) => Chip(label: Text(c))).toList(),
)
```

## Horizontal scroll alternative

If the list is long and you want it to stay on one line:

```dart
SizedBox(
  height: 48,
  child: ListView.separated(
    scrollDirection: Axis.horizontal,
    itemCount: categories.length,
    separatorBuilder: (_, __) => const SizedBox(width: 8),
    itemBuilder: (_, i) => Chip(label: Text(categories[i])),
  ),
)
```

`Wrap` doesn't scroll; `ListView` does. Choose based on the UX.

## Review tag

- `[WRAP-MISSING]` — a dynamic chip/button array via `.map(...).toList()` inside a `Row`.
