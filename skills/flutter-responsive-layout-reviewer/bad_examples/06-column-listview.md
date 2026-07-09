# Bad example: ListView inside a Column (without Expanded)

**Tag:** `[INFINITE-CONSTRAINT]`
**Problem:** An unwrapped `ListView` inside a `Column`.

```dart
Column(
  children: [
    Title(),
    ListView(
      children: [
        for (final item in items) Text(item),
      ],
    ),
  ],
)
```

**Why it breaks:**
- A `Column` gives its child infinite height.
- `ListView` by default wants an infinite scroll area; unbounded parent + unbounded child → error.
- Error: `Vertical viewport was given unbounded height.`

**Fix:** [../good_examples/06-column-listview.md](../good_examples/06-column-listview.md)
