# Kötü örnek: Row içinde sarmalanmamış Text

**Etiket:** `[FLEX-MISSING]`
**Sorun:** `Text` `Row` içinde sarmalanmamış. Küçük ekranlarda `RenderFlex overflowed by X pixels`.

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(uzunKullaniciAdi),
  ],
)
```

**Neden kırılır:** `Row` çocuğa sınırsız genişlik verir. `Text` satır sonu yapamaz ve doğal genişliği parent'tan büyükse taşar.

**Çözüm:** [../good_examples/01-row-text-overflow.md](../good_examples/01-row-text-overflow.md)
