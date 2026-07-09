# Bad example: Wrong fit choice

**Tag:** `[FLEX-WRONG-FIT]`
**Problem:** A `Chip` that should keep its natural size is wrapped in `Expanded`.

```dart
Row(
  children: [
    Expanded(child: Chip(label: Text('Label'))),
    SizedBox(width: 8),
    Expanded(child: Chip(label: Text('Second'))),
  ],
)
```

**Why it's bad:**
- A `Chip` is small and readable at its natural size. `Expanded` **forces** it to grow and take up half the row.
- Visually odd — the concept of a chip is to be compact.
- On small screens too, the large chips fill the screen.

**Fix:** [../good_examples/03-wrong-fit-chip.md](../good_examples/03-wrong-fit-chip.md)
