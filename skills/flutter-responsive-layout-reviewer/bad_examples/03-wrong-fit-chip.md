# Kötü örnek: Yanlış fit seçimi

**Etiket:** `[FLEX-WRONG-FIT]`
**Sorun:** Doğal boyutta kalması gereken `Chip` `Expanded` ile sarılmış.

```dart
Row(
  children: [
    Expanded(child: Chip(label: Text('Etiket'))),
    SizedBox(width: 8),
    Expanded(child: Chip(label: Text('İkinci'))),
  ],
)
```

**Neden kötü:**
- `Chip` doğal boyutunda küçük ve okunabilir. `Expanded` onu satırın yarısını kaplayacak şekilde **zorla** büyütür.
- Görsel olarak garip — chip kavramı kompakt olmaktır.
- Küçük ekranda da büyük chip'ler ekranı kaplar.

**Çözüm:** [../good_examples/03-wrong-fit-chip.md](../good_examples/03-wrong-fit-chip.md)
