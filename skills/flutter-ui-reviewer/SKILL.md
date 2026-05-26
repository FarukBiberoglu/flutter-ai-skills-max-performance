---
name: flutter-ui-reviewer
description: Flutter UI kodunu Material/Cupertino best practice'lerine, responsive tasarıma ve widget ağaç optimizasyonuna göre inceler.
---

# Flutter UI Reviewer

> Bu skill çalışırken [../../CLAUDE.md](../../CLAUDE.md) içindeki davranış kurallarına (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) uyar.

## Amaç

Flutter widget kodunu inceleyerek aşağıdaki konularda iyileştirme önerileri sunar:

- Widget ağacının gereksiz derinliği ve `const` kullanımı
- Responsive tasarım (MediaQuery, LayoutBuilder, Flex kullanımı)
- Material 3 / Cupertino tasarım kurallarına uygunluk
- Accessibility (Semantics, contrast, touch target boyutları)
- Theme ve renk kullanımının tutarlılığı
- Tekrar eden widget'ların ortak bir widget'a çıkarılması

## Ne zaman kullan

- Yeni bir ekran/widget yazıldığında
- Mevcut UI kodunda refactor öncesi
- Code review sürecinde

## Nasıl çalışır

1. Belirtilen widget dosyalarını okur.
2. Yukarıdaki kriterlere göre analiz eder.
3. Her bulgu için:
   - Dosya ve satır referansı
   - Sorunun açıklaması
   - Önerilen çözüm (kod örneği ile)
   verir.

## Çıktı formatı

```
[DOSYA:SATIR] - [KATEGORİ] - [LINT_KURALI]
Sorun: ...
Öneri: ...
Örnek:
```dart
// ...
```
```

`LINT_KURALI` alanı `analysis_options.yaml` içindeki kural adıyla (örn. `prefer_const_constructors`, `use_key_in_widget_constructors`) eşleşmelidir. Lint kuralı kapsamı dışındaki bulgular için `[CUSTOM]` etiketi kullanılır.

## Zorunlu lint referansı

> **ZORUNLU:** Bu skill çalışmadan önce projenin kök dizininde `analysis_options.yaml` dosyası bulunmalı ve [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml) içindeki kural setini içermelidir.
>
> Skill akışı:
> 1. İlk adımda projenin kökündeki `analysis_options.yaml` okunur.
> 2. Dosya **yoksa** → review başlatılmaz, kullanıcıya `examples/analysis_options.yaml` dosyasını projeye kopyalaması söylenir.
> 3. Dosya **var ama eksik kural** varsa → review öncesi eksik kurallar listelenir ve eklenmesi istenir.
> 4. Tüm kurallar mevcutsa review başlar ve her bulgu **hangi lint kuralını ihlal ettiği** ile birlikte raporlanır.
>
> Bu adım atlanamaz; `analysis_options.yaml` olmadan yapılan review geçersiz sayılır.

Bu skill, `examples/analysis_options.yaml` içindeki UI ile ilgili kuralları temel alır:

- `use_key_in_widget_constructors`
- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `prefer_const_constructors`
- `prefer_const_constructors_in_immutables`
- `prefer_const_literals_to_create_immutables`
- `use_build_context_synchronously`
- `no_logic_in_create_state`

Projeye eklemek için: [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml)

## Responsive kuralları

`Row`, `Column` ve `Flex` içindeki çocuklar için **`Flexible` / `Expanded`** kullanımını zorunlu kıl. Sabit `width` / `height` veya hardcoded `SizedBox(width: 200)` ile yapılan layout'lar küçük ekranlarda taşmaya (`RenderFlex overflowed by X pixels`) yol açar.

**Kural:**

- `Row` / `Column` içinde metin, input veya esneyebilen bir widget varsa `Expanded` veya `Flexible` ile sarılmalı.
- Oransal bölme için `Expanded(flex: 2)` / `Expanded(flex: 1)` kullanılmalı, `MediaQuery.of(context).size.width * 0.66` gibi manuel hesap yerine.
- Esnek olmayıp doğal boyutunda kalması gereken çocuklar (`Icon`, küçük `Image`) sarılmaz; ama yanındaki esneyen widget mutlaka `Expanded` olur.
- `Flexible(fit: FlexFit.loose)` ↔ `Expanded` (== `Flexible(fit: FlexFit.tight)`) farkı bilinçli seçilmeli: çocuk kendi boyutunu korumalıysa `Flexible`, kalan alanı doldurmalıysa `Expanded`.
- `SingleChildScrollView` + `Column` deseninde `Expanded` çalışmaz; bu durumda `IntrinsicHeight` veya `ConstrainedBox(minHeight: ...)` önerilmeli.

**Kötü örnek:**

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(uzunKullaniciAdi), // küçük ekranda overflow eder
  ],
)
```

**İyi örnek:**

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        uzunKullaniciAdi,
        overflow: TextOverflow.ellipsis,
      ),
    ),
  ],
)
```

Review sırasında `Row` / `Column` içindeki her `Text`, `TextField`, esnek `Container` için `Expanded` / `Flexible` kontrolü yapılır; eksikse `[RESPONSIVE]` etiketiyle raporlanır.
