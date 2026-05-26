# Kötü örnek: Column içinde ListView (Expanded'sız)

**Etiket:** `[INFINITE-CONSTRAINT]`
**Sorun:** `Column` içinde sarılmamış `ListView`.

```dart
Column(
  children: [
    Title(),
    ListView(
      children: [
        for (final item in items) Text(item),
      ],
    ),
  ],
)
```

**Neden kırılır:**
- `Column` çocuğa sonsuz yükseklik verir.
- `ListView` varsayılan olarak sonsuz scroll alanı ister; sınırsız parent + sınırsız çocuk → hata.
- Hata: `Vertical viewport was given unbounded height.`

**Çözüm:** [../good_examples/06-column-listview.md](../good_examples/06-column-listview.md)
