# İyi örnek: Expanded(flex:) ile oransal bölme

**Çözüm:** `MediaQuery` yerine `Expanded(flex: ...)`.

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: TextField(),
    ),
    const SizedBox(width: 8),
    Expanded(
      flex: 1,
      child: ElevatedButton(
        onPressed: () {},
        child: const Text('Ara'),
      ),
    ),
  ],
)
```

**Notlar:**
- `flex: 2` ve `flex: 1` → kalan alan 2:1 oranında bölünür.
- `SizedBox(width: 8)` sabit; flex hesabına dahil değil.
- Padding, parent container, split-screen — hiçbiri kırmaz çünkü oran **kalan alan üzerinden** hesaplanır.

**Karşılaştırma:** [../bad_examples/02-mediaquery-ratio.md](../bad_examples/02-mediaquery-ratio.md)
