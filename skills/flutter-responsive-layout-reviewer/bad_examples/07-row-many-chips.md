# Bad example: Dynamic chip list inside a Row

**Tag:** `[WRAP-MISSING]`
**Problem:** A variable number of chips laid out side by side with a `Row`.

```dart
Row(
  children: categories
      .map((c) => Padding(
            padding: const EdgeInsets.only(right: 8),
            child: Chip(label: Text(c)),
          ))
      .toList(),
)
```

**Why it breaks:**
- As `categories.length` grows, the `Row` doesn't fit the screen → overflow.
- Wrapping with `Expanded` doesn't help; if all chips shrink equally, they become unreadable.
- Manual `if (count > 4) ...` logic is fragile.

**Fix:** [../good_examples/07-row-many-chips.md](../good_examples/07-row-many-chips.md)
