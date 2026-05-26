# Kötü örnek: Row içinde dinamik chip listesi

**Etiket:** `[WRAP-MISSING]`
**Sorun:** Değişken sayıda chip `Row` ile yan yana diziliyor.

```dart
Row(
  children: kategoriler
      .map((k) => Padding(
            padding: const EdgeInsets.only(right: 8),
            child: Chip(label: Text(k)),
          ))
      .toList(),
)
```

**Neden kırılır:**
- `kategoriler.length` arttığında `Row` ekrana sığmaz → overflow.
- `Expanded` ile sarmak işe yaramaz; tüm chip'ler eşit küçülürse okunamaz hâle gelir.
- Manuel `if (count > 4) ...` mantığı kırılgan.

**Çözüm:** [../good_examples/07-row-many-chips.md](../good_examples/07-row-many-chips.md)
