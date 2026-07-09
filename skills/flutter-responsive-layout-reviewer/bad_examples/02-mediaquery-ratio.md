# Bad example: Manual ratio with MediaQuery

**Tag:** `[MEDIAQUERY-MISUSE]`
**Problem:** Manual splitting via a percentage of the screen width.

```dart
Row(
  children: [
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.66,
      child: TextField(),
    ),
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.34,
      child: ElevatedButton(onPressed: () {}, child: Text('Search')),
    ),
  ],
)
```

**Why it breaks:**
- Padding, scaffold drawer, and side panel are not accounted for.
- On split-screen or a different parent container, it returns the wrong width.
- `0.66 + 0.34 = 1.0`, but once padding/spacing is added the total exceeds 1.0 → overflow.
- Hardcoded percentages break when the design changes.

**Fix:** [../good_examples/02-mediaquery-ratio.md](../good_examples/02-mediaquery-ratio.md)
