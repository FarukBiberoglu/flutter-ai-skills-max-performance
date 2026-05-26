# Kural: Wrap mı, Row mu?

Çok sayıda küçük widget (chip, etiket, buton) yan yana dizilecekse `Row` taşar. `Wrap` satır sığmadığında otomatik alt satıra geçer.

## Karar şeması

| Durum | Widget |
|---|---|
| Sabit sayıda (2–4) widget, hepsi sığmalı | `Row` + `Expanded` / `Flexible` |
| Değişken sayıda widget, taşarsa alt satıra geçsin | `Wrap` |
| Yatay scroll edilecek | `SingleChildScrollView(scrollDirection: Axis.horizontal)` veya `ListView.builder(scrollDirection: ...)` |
| Eşit sütunlu grid | `Wrap` veya `GridView` |

## Wrap parametreleri

```dart
Wrap(
  spacing: 8,        // yatay aradaki boşluk
  runSpacing: 4,     // alt satıra geçtiğinde dikey boşluk
  alignment: WrapAlignment.start,
  crossAxisAlignment: WrapCrossAlignment.center,
  children: [...],
)
```

## Kötü → İyi

```dart
// YANLIŞ
Row(
  children: kategoriler.map((k) => Chip(label: Text(k))).toList(),
)

// DOĞRU
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: kategoriler.map((k) => Chip(label: Text(k))).toList(),
)
```

## Yatay scroll alternatifi

Liste uzunsa ve tek satırda kalmasını istiyorsan:

```dart
SizedBox(
  height: 48,
  child: ListView.separated(
    scrollDirection: Axis.horizontal,
    itemCount: kategoriler.length,
    separatorBuilder: (_, __) => const SizedBox(width: 8),
    itemBuilder: (_, i) => Chip(label: Text(kategoriler[i])),
  ),
)
```

`Wrap` kaydırılmaz; `ListView` kaydırılır. UX'e göre seç.

## Review etiketi

- `[WRAP-MISSING]` — `Row` içinde `.map(...).toList()` ile dinamik chip/buton dizisi.
