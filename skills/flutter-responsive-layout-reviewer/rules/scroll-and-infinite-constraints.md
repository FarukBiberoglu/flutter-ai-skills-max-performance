# Kural: Scroll ve sonsuz constraint

`Expanded` yalnızca **bounded** (sınırlı) eksende çalışır. Scroll widget'ları çocuklarına **unbounded** eksen verir.

## SingleChildScrollView + Column içinde Expanded

`SingleChildScrollView` çocuğuna **sonsuz yükseklik** verir → içindeki `Column`'da `Expanded` çalışmaz.

Hata: `RenderFlex children have non-zero flex but incoming height constraints are unbounded.`

### Çözüm seçenekleri

**(a) Tam yükseklik, scroll opsiyonel**

```dart
LayoutBuilder(
  builder: (context, constraints) => SingleChildScrollView(
    child: ConstrainedBox(
      constraints: BoxConstraints(minHeight: constraints.maxHeight),
      child: IntrinsicHeight(
        child: Column(
          children: [
            Header(),
            Expanded(child: Body()), // artık çalışır
            Footer(),
          ],
        ),
      ),
    ),
  ),
)
```

**(b) Sliver yapısı (önerilen)**

```dart
CustomScrollView(
  slivers: [
    SliverToBoxAdapter(child: Header()),
    SliverList(delegate: SliverChildBuilderDelegate(...)),
    SliverToBoxAdapter(child: Footer()),
  ],
)
```

**(c) Sabit yükseklik**

```dart
Column(children: [
  Header(),
  SizedBox(height: 240, child: ListView(...)),
  Footer(),
])
```

## Column içinde ListView / başka Column

`Column` çocuklarına **sonsuz yükseklik** verir. İçinde dikey kayan widget varsa hata olur.

```dart
// YANLIŞ
Column(children: [
  Title(),
  ListView(children: [...]), // ❌ infinite constraint
])

// DOĞRU
Column(children: [
  Title(),
  Expanded(child: ListView(children: [...])), // ✅
])
```

### `shrinkWrap` alternatifi

```dart
ListView(
  shrinkWrap: true,
  physics: NeverScrollableScrollPhysics(),
  children: [...],
)
```

- **Avantaj:** `Expanded` gerekmez, `Column` içinde doğal yüksekliği alır.
- **Dezavantaj:** Liste her build'de tüm child'ları layout eder (lazy değildir). Yalnızca **kısa, sabit** listelerde kullan.
- Genel kural: kaydırılacaksa `Expanded(child: ListView.builder(...))`, kaydırılmayacak ve kısaysa `shrinkWrap` veya doğrudan `Column` içinde `...items`.

## Row içinde Row / yatay scroll

Aynı problem yatay eksende:

```dart
// YANLIŞ
Row(children: [
  Icon(...),
  ListView(scrollDirection: Axis.horizontal, ...), // ❌
])

// DOĞRU
Row(children: [
  Icon(...),
  Expanded(
    child: ListView(scrollDirection: Axis.horizontal, ...),
  ),
])
```

## Review etiketleri

- `[SCROLL-EXPANDED]` — `SingleChildScrollView` → `Column` → `Expanded` zinciri.
- `[INFINITE-CONSTRAINT]` — `Column` / `Row` içinde sınırsız eksende kayan widget.

Örnekler: [../bad_examples/05-scroll-expanded.md](../bad_examples/05-scroll-expanded.md), [../bad_examples/06-column-listview.md](../bad_examples/06-column-listview.md).
