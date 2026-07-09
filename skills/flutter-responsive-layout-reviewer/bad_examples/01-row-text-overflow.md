# Bad example: Unwrapped Text inside a Row

**Tag:** `[FLEX-MISSING]`
**Problem:** A `Text` is not wrapped inside a `Row`. On small screens, `RenderFlex overflowed by X pixels`.

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(longUserName),
  ],
)
```

**Why it breaks:** A `Row` gives its child unbounded width. `Text` can't line-break and overflows if its natural width is larger than the parent.

**Fix:** [../good_examples/01-row-text-overflow.md](../good_examples/01-row-text-overflow.md)
