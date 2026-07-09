# Good example: Dynamic chip list with Wrap

**Solution A — Wrap (moves to the next line automatically):**

```dart
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: categories
      .map((c) => Chip(label: Text(c)))
      .toList(),
)
```

**Solution B — horizontal scroll (if it must stay on one line):**

```dart
SizedBox(
  height: 40,
  child: ListView.separated(
    scrollDirection: Axis.horizontal,
    itemCount: categories.length,
    separatorBuilder: (_, __) => const SizedBox(width: 8),
    itemBuilder: (_, i) => Chip(label: Text(categories[i])),
  ),
)
```

**Which one?**
- All chips must be **visible at once** → A.
- There are many chips, **browse by scrolling** → B.

**Comparison:** [../bad_examples/07-row-many-chips.md](../bad_examples/07-row-many-chips.md)
