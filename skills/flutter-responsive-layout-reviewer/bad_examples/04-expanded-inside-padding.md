# Kötü örnek: Expanded Flex'in doğrudan child'ı değil

**Etiket:** `[FLEX-MISSING]` (runtime error)
**Sorun:** `Expanded` `Padding` içinde sarılı; `Row`'un doğrudan çocuğu değil.

```dart
Row(
  children: [
    Padding(
      padding: const EdgeInsets.all(8),
      child: Expanded(child: Text(uzunMetin)),
    ),
  ],
)
```

**Neden kırılır:**
- Runtime hata: `Incorrect use of ParentDataWidget. Expanded widgets must be placed directly inside Flex widgets.`
- `Expanded` `ParentDataWidget`'tır; yalnızca `Flex` (Row/Column) tarafından okunan flex bilgisini taşır.

**Çözüm:** Sıralamayı ters çevir — `Expanded` dışta, `Padding` içte.
[../good_examples/04-expanded-inside-padding.md](../good_examples/04-expanded-inside-padding.md)
