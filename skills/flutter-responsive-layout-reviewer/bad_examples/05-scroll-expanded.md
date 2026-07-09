# Bad example: Expanded inside SingleChildScrollView

**Tag:** `[SCROLL-EXPANDED]`
**Problem:** The `SingleChildScrollView` → `Column` → `Expanded` chain.

```dart
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      Expanded(
        child: ListView.builder(
          itemBuilder: (_, i) => Text('Item $i'),
          itemCount: 100,
        ),
      ),
    ],
  ),
)
```

**Why it breaks:**
- `SingleChildScrollView` gives its child **infinite height**.
- `Expanded` says "take the remaining space" but the remaining space is infinite → error: `RenderFlex children have non-zero flex but incoming height constraints are unbounded.`

**Fix:** [../good_examples/05-scroll-expanded.md](../good_examples/05-scroll-expanded.md)
