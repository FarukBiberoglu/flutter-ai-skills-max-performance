# İyi örnek: Expanded dışta, Padding içte

**Çözüm:** `Expanded` `Row`'un doğrudan çocuğu olmalı; `Padding` `Expanded`'ın içine.

```dart
Row(
  children: [
    Expanded(
      child: Padding(
        padding: const EdgeInsets.all(8),
        child: Text(uzunMetin),
      ),
    ),
  ],
)
```

**Genel kural:**
`Expanded` ve `Flexible` `ParentDataWidget`'tır — yalnızca `Flex` (Row/Column) tarafından okunur. **Daima** `Flex`'in doğrudan child'ı olmalı:

```
Row / Column
  └── Expanded   ← burada
        └── (Padding, Container, ne olursa)
              └── (gerçek içerik)
```

**Karşılaştırma:** [../bad_examples/04-expanded-inside-padding.md](../bad_examples/04-expanded-inside-padding.md)
