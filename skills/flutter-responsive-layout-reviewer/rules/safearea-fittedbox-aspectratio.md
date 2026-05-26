# Kural: SafeArea, FittedBox, AspectRatio

Üç yardımcı widget; her biri ayrı bir overflow / kırılma sınıfını çözer.

## SafeArea

Sayfa kökünde **mutlaka** olmalı; aksi hâlde notch, status bar, home indicator altına widget girer.

```dart
Scaffold(
  body: SafeArea(
    child: ...,
  ),
)
```

- `Scaffold` zaten `AppBar` ve `bottomNavigationBar` için inset uygular. `body` için `SafeArea` yine de gerekir.
- `MediaQuery.of(context).padding.top` ile manuel hesap **yasak** — `SafeArea` daha doğru çalışır.
- `SafeArea(minimum: EdgeInsets.only(bottom: 16))` ile minimum padding zorlanabilir.

## AspectRatio

Görsel ya da kart sabit bir oranda kalmalıysa:

```dart
AspectRatio(
  aspectRatio: 16 / 9,
  child: Image.network(url, fit: BoxFit.cover),
)
```

- `aspectRatio = en / boy`. `16 / 9` = `1.78`.
- Parent bounded olmalı (en veya boy bilinmeli). `Column` içinde `AspectRatio` çalışır çünkü `Column` çocuğa sınırlı genişlik verir.
- `ListView` içinde `AspectRatio` çalışmaz (yükseklik sınırsız) — `SizedBox(height: ...)` veya `Expanded` ile bound et.

## FittedBox

İçeriği parent'a sığdırır. Sıklıkla **yanlış** yerde kullanılır.

```dart
FittedBox(
  fit: BoxFit.scaleDown,
  child: Text('Çok uzun başlık'),
)
```

- `BoxFit.scaleDown`: sığmazsa küçültür, sığarsa olduğu gibi bırakır.
- `BoxFit.contain`: her zaman parent'a sığacak şekilde scale eder (küçükse büyütür de).

**Yanlış kullanım:** `Text` overflow'u için. `Text` küçültmek yerine `overflow: TextOverflow.ellipsis` ile kes:

```dart
// Genelde DAHA İYİ
Expanded(
  child: Text(
    longText,
    overflow: TextOverflow.ellipsis,
    maxLines: 1,
  ),
)
```

`FittedBox`'ı yalnızca **boyut bilgisi taşıyan** öğeler için kullan (sayısal göstergeler, logo). Uzun metni küçültmek okunabilirliği bozar.

## IntrinsicHeight / IntrinsicWidth

Pahalı (O(n²)) widget'lar. Yalnızca `Row` içindeki kardeşler aynı yüksekliği almak zorundaysa kullan; alternatif yoksa.

## Review etiketleri

- `[SAFEAREA-MISSING]` — Sayfa kökünde `SafeArea` yok.
- `[ASPECT-RATIO]` — Görsel/kart oranı sabit kalmıyor, container'a göre bozuluyor.
- `[FITTEDBOX-MISUSE]` — Uzun metni `FittedBox` ile küçültme (genelde `ellipsis` daha iyi).
