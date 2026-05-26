# Kötü örnek: SingleChildScrollView içinde Expanded

**Etiket:** `[SCROLL-EXPANDED]`
**Sorun:** `SingleChildScrollView` → `Column` → `Expanded` zinciri.

```dart
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      Expanded(
        child: ListView.builder(
          itemBuilder: (_, i) => Text('Item $i'),
          itemCount: 100,
        ),
      ),
    ],
  ),
)
```

**Neden kırılır:**
- `SingleChildScrollView` çocuğa **sonsuz yükseklik** verir.
- `Expanded` "kalan alanı al" der ama kalan alan sonsuz → hata: `RenderFlex children have non-zero flex but incoming height constraints are unbounded.`

**Çözüm:** [../good_examples/05-scroll-expanded.md](../good_examples/05-scroll-expanded.md)
