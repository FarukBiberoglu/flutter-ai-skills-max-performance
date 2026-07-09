# Good example: Row + Expanded + ellipsis

**Fix:** Wrap the Text in `Expanded` and specify `overflow`.

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        longUserName,
        overflow: TextOverflow.ellipsis,
        maxLines: 1,
      ),
    ),
  ],
)
```

**Notes:**
- `Icon` and `SizedBox` keep their natural size.
- `Expanded` fits the `Text` into the remaining space; long text is clipped with "...".
- `maxLines: 2` allows two lines, then ellipsis.
- All fixed widgets are marked `const` (no rebuild cost).

**Comparison:** [../bad_examples/01-row-text-overflow.md](../bad_examples/01-row-text-overflow.md)
