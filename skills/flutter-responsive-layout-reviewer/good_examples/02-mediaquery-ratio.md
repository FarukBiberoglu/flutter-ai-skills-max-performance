# Good example: Proportional split with Expanded(flex:)

**Fix:** `Expanded(flex: ...)` instead of `MediaQuery`.

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: TextField(),
    ),
    const SizedBox(width: 8),
    Expanded(
      flex: 1,
      child: ElevatedButton(
        onPressed: () {},
        child: const Text('Search'),
      ),
    ),
  ],
)
```

**Notes:**
- `flex: 2` and `flex: 1` → the remaining space is split 2:1.
- `SizedBox(width: 8)` is fixed; not part of the flex math.
- Padding, parent container, split-screen — none of them break it, because the ratio is computed **over the remaining space**.

**Comparison:** [../bad_examples/02-mediaquery-ratio.md](../bad_examples/02-mediaquery-ratio.md)
