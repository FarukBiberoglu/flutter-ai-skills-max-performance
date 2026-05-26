# Kötü örnek: MediaQuery ile manuel oran

**Etiket:** `[MEDIAQUERY-MISUSE]`
**Sorun:** Ekran genişliğinin yüzdesi ile manuel bölme.

```dart
Row(
  children: [
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.66,
      child: TextField(),
    ),
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.34,
      child: ElevatedButton(onPressed: () {}, child: Text('Ara')),
    ),
  ],
)
```

**Neden kırılır:**
- Padding, scaffold drawer, side panel hesaba katılmaz.
- Split-screen veya farklı parent container'da yanlış genişlik döner.
- `0.66 + 0.34 = 1.0` ama padding/spacing eklendiğinde toplam 1.0'ı geçer → taşma.
- Hardcoded yüzdeler tasarım değişince kırılır.

**Çözüm:** [../good_examples/02-mediaquery-ratio.md](../good_examples/02-mediaquery-ratio.md)
