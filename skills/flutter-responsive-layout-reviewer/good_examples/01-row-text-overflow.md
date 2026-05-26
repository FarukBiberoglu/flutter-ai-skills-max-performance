# İyi örnek: Row + Expanded + ellipsis

**Çözüm:** Text'i `Expanded` ile sar, `overflow` belirt.

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        uzunKullaniciAdi,
        overflow: TextOverflow.ellipsis,
        maxLines: 1,
      ),
    ),
  ],
)
```

**Notlar:**
- `Icon` ve `SizedBox` doğal boyutta kalır.
- `Expanded` `Text`'i kalan alana sığdırır; uzun metin "..." ile kesilir.
- `maxLines: 2` iki satıra izin verir, sonra ellipsis.
- Tüm sabit widget'lar `const` ile işaretli (rebuild maliyeti yok).

**Karşılaştırma:** [../bad_examples/01-row-text-overflow.md](../bad_examples/01-row-text-overflow.md)
