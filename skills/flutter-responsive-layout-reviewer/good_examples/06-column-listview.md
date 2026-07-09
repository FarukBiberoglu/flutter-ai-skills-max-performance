# Good example: Column + Expanded(ListView)

**Fix:** Wrap the `ListView` in `Expanded` — it fills the remaining vertical space.

```dart
Column(
  children: [
    const Title(),
    Expanded(
      child: ListView.builder(
        itemBuilder: (_, i) => Text(items[i]),
        itemCount: items.length,
      ),
    ),
  ],
)
```

**Notes:**
- `ListView.builder` is lazy — only visible items are built.
- Thanks to `Expanded`, the `ListView` uses all the vertical space left under `Title()`.

**Alternative — for short, fixed lists:**

```dart
Column(
  children: [
    const Title(),
    ListView(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      children: items.map((i) => Text(i)).toList(),
    ),
  ],
)
```

`shrinkWrap` shrinks the list to its natural height. Use it only when there are **few items** — otherwise performance drops (it's not lazy).

**Comparison:** [../bad_examples/06-column-listview.md](../bad_examples/06-column-listview.md)
