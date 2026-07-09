# Good example: Chip with Flexible(loose)

**Fix:** `Flexible` instead of `Expanded`; the chip keeps its natural size and shrinks if needed.

```dart
Row(
  children: [
    Flexible(child: Chip(label: Text('Label'))),
    const SizedBox(width: 8),
    Flexible(child: Chip(label: Text('Second'))),
  ],
)
```

Or don't wrap at all:

```dart
Row(
  mainAxisSize: MainAxisSize.min,
  children: const [
    Chip(label: Text('Label')),
    SizedBox(width: 8),
    Chip(label: Text('Second')),
  ],
)
```

**Notes:**
- `Flexible` comes with the default `FlexFit.loose` → "you can grow up to the remaining space, but you can also stay at your natural size".
- If there are many chips this still isn't enough; then use `Wrap` (see [07-row-many-chips.md](07-row-many-chips.md)).

**Comparison:** [../bad_examples/03-wrong-fit-chip.md](../bad_examples/03-wrong-fit-chip.md)
