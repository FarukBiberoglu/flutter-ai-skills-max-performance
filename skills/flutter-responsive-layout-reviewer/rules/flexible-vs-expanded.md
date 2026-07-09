# Rule: Flexible vs Expanded

> This is the skill's **core** rule. It's the expanded version of the summary in SKILL.md.

## Definition

| Widget | Equivalent | Behavior |
|---|---|---|
| `Expanded` | `Flexible(fit: FlexFit.tight)` | **Forces** filling the remaining space. The child is grown even if it wants to stay small. |
| `Flexible` | default `fit: FlexFit.loose` | **Allows** growth up to the remaining space. The child can keep its natural size. |
| Unwrapped | — | The child keeps its natural size; if the total width doesn't fit, it overflows. |

## Decision flow

```
For a child inside a Row / Column:

  Should the child fill the remaining space COMPLETELY?
    YES → Expanded
    NO ↓

  Can the child grow up to the remaining space but also stay at its natural size?
    YES → Flexible
    NO ↓

  Should the child stay fixed at its natural size? (Icon, small Image, CircleAvatar)
    YES → don't wrap
    NO → reconsider the design
```

## Proportional splitting

Manual `MediaQuery` ratios are **forbidden**. Use the flex system:

```dart
Row(
  children: [
    Expanded(flex: 2, child: A()), // 2/3
    Expanded(flex: 1, child: B()), // 1/3
  ],
)
```

The default `flex` value is `1`; no need to write it for an equal split.

## Critical warnings

### 1. `Expanded` must be a direct child of `Flex`

```dart
// WRONG — runtime error
Row(children: [
  Padding(
    padding: EdgeInsets.all(8),
    child: Expanded(child: Text(...)), // ❌
  ),
])

// CORRECT
Row(children: [
  Expanded(
    child: Padding(
      padding: EdgeInsets.all(8),
      child: Text(...),
    ),
  ),
])
```

Same rule for: `Column`, `Flex`, `ListView` (an `Expanded` inside it doesn't work because `ListView` is not a `Flex`).

### 2. `Expanded` doesn't nest

`Expanded(child: Expanded(...))` — no compile error but pointless. It's usually a refactor leftover; remove it.

### 3. Don't write `Flexible(fit: FlexFit.tight)`

For readability, use `Expanded` directly. It means the same thing.

### 4. Flex math with multiple Expanded

```dart
Row(children: [
  Expanded(flex: 2, child: A()), // total space / 3 * 2
  Expanded(flex: 1, child: B()), // total space / 3 * 1
])
```

If you want a gap in between with `SizedBox(width: ...)`, **the gap is not part of the flex**:

```dart
Row(children: [
  Expanded(flex: 2, child: A()),
  const SizedBox(width: 8), // fixed, not part of the flex math
  Expanded(flex: 1, child: B()),
])
```

## Commonly confused cases

### `Text` overflow

If there's a long `Text` inside a `Row`, `Expanded` + `overflow` is **required**:

```dart
Expanded(
  child: Text(
    longText,
    overflow: TextOverflow.ellipsis,
    maxLines: 1,
  ),
)
```

`Text` can't line-break on its own because a `Row` gives its children unbounded width.

### A natural-size widget like `Chip` / `Badge`

Wrapping with `Expanded` → looks ugly (fills the whole row).
Wrap with `Flexible` → the chip stays visible even if the text shrinks on small screens.

### `Spacer`

`Spacer(flex: 1)` ≡ `Expanded(flex: 1, child: SizedBox.shrink())`. Handy for a gap between two widgets.

## Review tags

- `[FLEX-MISSING]` — A stretchable child is not wrapped.
- `[FLEX-WRONG-FIT]` — `Flexible` instead of `Expanded`, or vice versa.
- `[FLEX-NESTED]` — Nested `Expanded`.
- `[MEDIAQUERY-MISUSE]` — Manual ratio math.

Examples: [../bad_examples/](../bad_examples/) and [../good_examples/](../good_examples/).
