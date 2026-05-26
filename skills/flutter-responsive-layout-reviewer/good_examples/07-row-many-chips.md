# İyi örnek: Wrap ile dinamik chip listesi

**Çözüm A — Wrap (alt satıra otomatik geçer):**

```dart
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: kategoriler
      .map((k) => Chip(label: Text(k)))
      .toList(),
)
```

**Çözüm B — Yatay scroll (tek satır kalmalıysa):**

```dart
SizedBox(
  height: 40,
  child: ListView.separated(
    scrollDirection: Axis.horizontal,
    itemCount: kategoriler.length,
    separatorBuilder: (_, __) => const SizedBox(width: 8),
    itemBuilder: (_, i) => Chip(label: Text(kategoriler[i])),
  ),
)
```

**Hangisi?**
- Tüm chip'ler **aynı anda görünür** olmalı → A.
- Çok fazla chip var, **kaydırarak gezilsin** → B.

**Karşılaştırma:** [../bad_examples/07-row-many-chips.md](../bad_examples/07-row-many-chips.md)
