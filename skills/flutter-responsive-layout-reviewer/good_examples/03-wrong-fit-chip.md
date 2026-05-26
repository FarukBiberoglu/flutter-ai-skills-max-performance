# İyi örnek: Flexible(loose) ile chip

**Çözüm:** `Expanded` yerine `Flexible`; chip doğal boyutta kalır, gerekirse küçülür.

```dart
Row(
  children: [
    Flexible(child: Chip(label: Text('Etiket'))),
    const SizedBox(width: 8),
    Flexible(child: Chip(label: Text('İkinci'))),
  ],
)
```

Veya hiç sarma:

```dart
Row(
  mainAxisSize: MainAxisSize.min,
  children: const [
    Chip(label: Text('Etiket')),
    SizedBox(width: 8),
    Chip(label: Text('İkinci')),
  ],
)
```

**Notlar:**
- `Flexible` varsayılan `FlexFit.loose` ile gelir → "kalan alana kadar büyüyebilirsin, ama doğal boyutunda kalabilirsin de".
- Çok chip varsa bu da yetmez; o zaman `Wrap` kullan (bkz. [07-row-many-chips.md](07-row-many-chips.md)).

**Karşılaştırma:** [../bad_examples/03-wrong-fit-chip.md](../bad_examples/03-wrong-fit-chip.md)
