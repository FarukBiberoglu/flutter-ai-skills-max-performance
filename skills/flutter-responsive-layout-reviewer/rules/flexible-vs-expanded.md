# Kural: Flexible vs Expanded

> Bu skill'in **temel** kuralıdır. SKILL.md'deki özetin genişletilmiş halidir.

## Tanım

| Widget | Eş değeri | Davranış |
|---|---|---|
| `Expanded` | `Flexible(fit: FlexFit.tight)` | Kalan alanı **zorla** doldurur. Çocuk küçük kalmak istese de büyütülür. |
| `Flexible` | varsayılan `fit: FlexFit.loose` | Kalan alana kadar **izin verir**. Çocuk doğal boyutunu koruyabilir. |
| Sarılmamış | — | Çocuk doğal boyutunda kalır; toplam genişlik sığmazsa overflow olur. |

## Karar şeması

```
Row / Column içindeki bir child için:

  child kalan alanı TAMAMEN doldurmalı mı?
    EVET → Expanded
    HAYIR ↓

  child kalan alana kadar büyüyebilir ama doğal boyutunda da kalabilir mi?
    EVET → Flexible
    HAYIR ↓

  child sabit, doğal boyutunda kalmalı mı? (Icon, küçük Image, CircleAvatar)
    EVET → sarma
    HAYIR → tasarımı gözden geçir
```

## Oransal bölme

Manuel `MediaQuery` oranı **yasak**. Flex sistemini kullan:

```dart
Row(
  children: [
    Expanded(flex: 2, child: A()), // 2/3
    Expanded(flex: 1, child: B()), // 1/3
  ],
)
```

`flex` varsayılan değeri `1`'dir; eşit bölme için yazmaya gerek yok.

## Kritik uyarılar

### 1. `Expanded` doğrudan `Flex`'in child'ı olmalı

```dart
// YANLIŞ — runtime error
Row(children: [
  Padding(
    padding: EdgeInsets.all(8),
    child: Expanded(child: Text(...)), // ❌
  ),
])

// DOĞRU
Row(children: [
  Expanded(
    child: Padding(
      padding: EdgeInsets.all(8),
      child: Text(...),
    ),
  ),
])
```

Aynı kural: `Column`, `Flex`, `ListView` (içindeki `Expanded` çalışmaz çünkü `ListView` `Flex` değildir).

### 2. `Expanded` iç içe geçmez

`Expanded(child: Expanded(...))` — derleme hatası yok ama mantıksız. Genelde refactor artığıdır, kaldır.

### 3. `Flexible(fit: FlexFit.tight)` yazma

Okunabilirlik için doğrudan `Expanded` kullan. Aynı şey demektir.

### 4. Birden fazla Expanded varsa flex hesabı

```dart
Row(children: [
  Expanded(flex: 2, child: A()), // toplam alan / 3 * 2
  Expanded(flex: 1, child: B()), // toplam alan / 3 * 1
])
```

`SizedBox(width: ...)` ile araya boşluk koymak istersen, **boşluk flex'e dahil değildir**:

```dart
Row(children: [
  Expanded(flex: 2, child: A()),
  const SizedBox(width: 8), // sabit, flex hesabına dahil değil
  Expanded(flex: 1, child: B()),
])
```

## Sık karıştırılan durumlar

### `Text` overflow

`Row` içinde uzun bir `Text` varsa **mutlaka** `Expanded` + `overflow` lazım:

```dart
Expanded(
  child: Text(
    longText,
    overflow: TextOverflow.ellipsis,
    maxLines: 1,
  ),
)
```

`Text` tek başına satır sonu yapamaz çünkü `Row` çocuklarına sınırsız genişlik verir.

### `Chip` / `Badge` gibi doğal-boyutlu widget

`Expanded` ile sarma → çirkin görünür (tüm satırı kaplar).
`Flexible` ile sar → küçük ekranda metin küçülse de chip görünür kalır.

### `Spacer`

`Spacer(flex: 1)` ≡ `Expanded(flex: 1, child: SizedBox.shrink())`. İki widget arasında boşluk için pratik.

## Review etiketleri

- `[FLEX-MISSING]` — Esneyebilen child sarılmamış.
- `[FLEX-WRONG-FIT]` — `Expanded` yerine `Flexible` ya da tersi.
- `[FLEX-NESTED]` — İç içe `Expanded`.
- `[MEDIAQUERY-MISUSE]` — Manuel oran hesabı.

Örnekler: [../bad_examples/](../bad_examples/) ve [../good_examples/](../good_examples/).
