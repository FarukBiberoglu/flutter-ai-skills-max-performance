# Good example: Expanded outside, Padding inside

**Fix:** `Expanded` must be a direct child of the `Row`; `Padding` goes inside the `Expanded`.

```dart
Row(
  children: [
    Expanded(
      child: Padding(
        padding: const EdgeInsets.all(8),
        child: Text(longText),
      ),
    ),
  ],
)
```

**General rule:**
`Expanded` and `Flexible` are `ParentDataWidget`s — read only by a `Flex` (Row/Column). They must **always** be a direct child of the `Flex`:

```
Row / Column
  └── Expanded   ← here
        └── (Padding, Container, whatever)
              └── (the actual content)
```

**Comparison:** [../bad_examples/04-expanded-inside-padding.md](../bad_examples/04-expanded-inside-padding.md)
