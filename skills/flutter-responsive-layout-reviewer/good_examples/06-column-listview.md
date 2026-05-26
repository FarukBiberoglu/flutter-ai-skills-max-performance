# İyi örnek: Column + Expanded(ListView)

**Çözüm:** `ListView`'u `Expanded` ile sar — kalan dikey alanı kaplar.

```dart
Column(
  children: [
    const Title(),
    Expanded(
      child: ListView.builder(
        itemBuilder: (_, i) => Text(items[i]),
        itemCount: items.length,
      ),
    ),
  ],
)
```

**Notlar:**
- `ListView.builder` lazy — sadece görünür item'lar build edilir.
- `Expanded` sayesinde `ListView` `Title()` altında kalan tüm dikey alanı kullanır.

**Alternatif — kısa, sabit listeler için:**

```dart
Column(
  children: [
    const Title(),
    ListView(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      children: items.map((i) => Text(i)).toList(),
    ),
  ],
)
```

`shrinkWrap` listeyi doğal yüksekliğine küçültür. Yalnızca **az item** varsa kullan — yoksa performans düşer (lazy değil).

**Karşılaştırma:** [../bad_examples/06-column-listview.md](../bad_examples/06-column-listview.md)
