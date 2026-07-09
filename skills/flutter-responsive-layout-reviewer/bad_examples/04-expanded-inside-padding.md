# Bad example: Expanded not a direct child of Flex

**Tag:** `[FLEX-MISSING]` (runtime error)
**Problem:** `Expanded` is wrapped in a `Padding`; it's not a direct child of the `Row`.

```dart
Row(
  children: [
    Padding(
      padding: const EdgeInsets.all(8),
      child: Expanded(child: Text(longText)),
    ),
  ],
)
```

**Why it breaks:**
- Runtime error: `Incorrect use of ParentDataWidget. Expanded widgets must be placed directly inside Flex widgets.`
- `Expanded` is a `ParentDataWidget`; it carries flex information read only by a `Flex` (Row/Column).

**Fix:** Reverse the order — `Expanded` outside, `Padding` inside.
[../good_examples/04-expanded-inside-padding.md](../good_examples/04-expanded-inside-padding.md)
