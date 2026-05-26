# İyi örnek: Scroll + dinamik liste

**Çözüm A — CustomScrollView (önerilen):**

```dart
CustomScrollView(
  slivers: [
    const SliverToBoxAdapter(child: Header()),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (_, i) => Text('Item $i'),
        childCount: 100,
      ),
    ),
  ],
)
```

**Çözüm B — Tam yükseklik garantisi, scroll opsiyonel:**

```dart
LayoutBuilder(
  builder: (context, constraints) => SingleChildScrollView(
    child: ConstrainedBox(
      constraints: BoxConstraints(minHeight: constraints.maxHeight),
      child: IntrinsicHeight(
        child: Column(
          children: [
            const Header(),
            Expanded(child: Body()),
            const Footer(),
          ],
        ),
      ),
    ),
  ),
)
```

**Çözüm C — Sabit yükseklik (basit ama esnek değil):**

```dart
Column(
  children: [
    const Header(),
    SizedBox(
      height: 320,
      child: ListView.builder(
        itemBuilder: (_, i) => Text('Item $i'),
        itemCount: 100,
      ),
    ),
  ],
)
```

**Hangisi?**
- Liste **uzun ve sayfa scroll'unun parçası** olmalıysa → A.
- Liste **kısa, sayfa scroll edilmemeli, içerik tam ekran kaplamalı** → B.
- Liste **sabit alan kaplayacak** → C.

**Karşılaştırma:** [../bad_examples/05-scroll-expanded.md](../bad_examples/05-scroll-expanded.md)
