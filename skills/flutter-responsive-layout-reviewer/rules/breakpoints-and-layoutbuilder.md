# Kural: Breakpoint ve LayoutBuilder

Tek layout tüm cihazlara uymaz. Tablet, foldable, web ve split-screen senaryolarında ayrı dallanma gerekir.

## MediaQuery vs LayoutBuilder

| Kullanım | Tercih |
|---|---|
| Ekranın tamamının boyutu (status bar, navigation) | `MediaQuery.of(context).size` |
| Widget'ın **kendi** kullanılabilir alanı | `LayoutBuilder` → `constraints.maxWidth` |
| Orientation (portrait/landscape) | `MediaQuery.of(context).orientation` veya `OrientationBuilder` |
| Klavye açık mı | `MediaQuery.of(context).viewInsets.bottom` |

**Kural:** Bir widget'ın layout'unu belirlerken `LayoutBuilder` tercih edilmeli. Split-screen, side panel veya yerleştirildiği yer küçükse `MediaQuery` yanıltıcı bilgi verir.

## Material 3 breakpoint'leri

```
< 600       compact     (telefon, dikey)
600 – 840   medium      (küçük tablet, foldable açık, telefon yatay)
> 840       expanded    (tablet, web, masaüstü)
```

## Tipik kullanım

```dart
LayoutBuilder(
  builder: (context, constraints) {
    final width = constraints.maxWidth;
    if (width >= 840) {
      return _ExpandedLayout(); // master-detail
    }
    if (width >= 600) {
      return _MediumLayout(); // tek kolon + daha geniş padding
    }
    return _CompactLayout(); // tek kolon, telefon
  },
)
```

## Anti-pattern: sihirli sayılar

```dart
// YANLIŞ
if (MediaQuery.of(context).size.width > 768) { ... }
if (MediaQuery.of(context).size.width > 500) { ... }
```

Breakpoint'ler **bir yerde** tanımlanmalı:

```dart
class Breakpoints {
  static const double compact = 600;
  static const double medium = 840;
}
```

## Orientation

Sadece `MediaQuery.of(context).orientation` ile dallanmak yetmez; tablet landscape ≠ telefon landscape. Genişliği de kontrol et.

## Review etiketleri

- `[BREAKPOINT]` — Tablet/web için layout dallanması yok.
- `[ORIENTATION]` — `OrientationBuilder` / `LayoutBuilder` yok, sadece portrait varsayılmış.
- `[MEDIAQUERY-MISUSE]` — Widget kendi alanı yerine ekran genişliği üzerinden karar veriyor.
