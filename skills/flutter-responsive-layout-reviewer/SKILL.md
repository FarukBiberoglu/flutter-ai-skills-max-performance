---
name: flutter-responsive-layout-reviewer
description: Flutter layout kodunu responsive tasarım ve özellikle Flexible / Expanded kullanımı açısından inceler; overflow ve sabit boyut hatalarını yakalar.
---

# Flutter Responsive Layout Reviewer

> Bu skill çalışırken [../../CLAUDE.md](../../CLAUDE.md) içindeki davranış kurallarına (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) uyar.

## Amaç

Flutter layout kodunu **responsive** açıdan inceler. Ana odak: **`Flexible` ve `Expanded` doğru kullanılıyor mu?** İkincil olarak `MediaQuery`, `LayoutBuilder`, `FittedBox`, `Wrap`, `IntrinsicHeight`, `AspectRatio` gibi araçların yerinde kullanımını denetler.

Hedef: küçük telefonlardan tablet/foldable/web ekranlarına kadar **hiçbir cihazda** `RenderFlex overflowed by X pixels` veya `BoxConstraints forces an infinite ...` hatası vermeyecek bir UI.

## Ne zaman kullan

- Yeni bir ekran/widget yazıldığında
- `RenderFlex overflowed` uyarısı alındığında
- Tablet / web / foldable desteği eklenmeden önce
- Mevcut Row / Column / Stack ağaçlarının refactor'undan önce
- Code review sürecinde

## Nasıl çalışır

1. Belirtilen widget dosyalarını okur.
2. `Row`, `Column`, `Flex`, `Wrap`, `Stack`, `ListView`, `GridView`, `SingleChildScrollView`, `CustomScrollView` widget'larını tarar.
3. Aşağıdaki kurallar listesine göre denetler.
4. Her bulgu için dosya:satır referansı, etiket, sorun açıklaması ve düzeltme örneği verir.

## Çıktı formatı

```
[DOSYA:SATIR] - [ETİKET] - [KRİTİKLİK: Yüksek/Orta/Düşük]
Sorun: ...
Etki: ... (overflow, infinite constraint, küçük ekranda taşma, vb.)
Öneri: ...
Örnek:
```dart
// ...
```
```

### Etiket seti

- `[FLEX-MISSING]` — Esneyebilen child `Expanded` / `Flexible` ile sarılmamış.
- `[FLEX-WRONG-FIT]` — `Flexible` yerine `Expanded` (veya tersi) kullanılmış.
- `[FLEX-NESTED]` — Gereksiz iç içe `Expanded` (örn. `Expanded(child: Expanded(...))`).
- `[FIXED-SIZE]` — Sabit `width` / `height` / `SizedBox(width: 200)` ile esnek olması gereken alan kısıtlanmış.
- `[MEDIAQUERY-MISUSE]` — `MediaQuery.of(context).size.width * 0.66` gibi manuel oran hesabı (yerine `Expanded(flex: ...)`).
- `[SCROLL-EXPANDED]` — `SingleChildScrollView` + `Column` içinde `Expanded` kullanılmış (çalışmaz).
- `[INFINITE-CONSTRAINT]` — `Column` içinde sınırsız yükseklik alan `ListView` / `Column` (Expanded veya `shrinkWrap` eksik).
- `[WRAP-MISSING]` — Yan yana çok sayıda chip/buton var; `Wrap` yerine `Row` kullanılmış.
- `[ORIENTATION]` — Sadece portrait varsayılmış; `OrientationBuilder` / `LayoutBuilder` ile dallanma eksik.
- `[BREAKPOINT]` — Tablet/web breakpoint yok; tek layout tüm ekranlara uygulanıyor.

## Zorunlu lint referansı

> **ZORUNLU:** Bu skill çalışmadan önce projenin kök dizininde `analysis_options.yaml` bulunmalı ve [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml) kural setini içermelidir.
>
> 1. Skill ilk adımda projenin kökündeki `analysis_options.yaml` dosyasını okur.
> 2. Dosya **yoksa** → review başlatılmaz; `examples/analysis_options.yaml` dosyasının projeye kopyalanması istenir.
> 3. Dosya **var ama eksik kural** varsa → eksik kurallar listelenir.
> 4. Tüm kurallar mevcutsa review başlar.

İlgili lint kuralları:

- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `use_key_in_widget_constructors`
- `prefer_const_constructors`

Layout doğruluğu lint ile yakalanamayan bir alandır; bu skill `[CUSTOM]` ve yukarıdaki etiket setini kullanır.

---

## 1. Flexible vs Expanded — TEMEL KURAL

> `Expanded` ≡ `Flexible(fit: FlexFit.tight)`.
> Yani `Expanded` çocuğa **"kalan tüm alanı doldur"** der; `Flexible(fit: FlexFit.loose)` ise **"en fazla şu kadar yer kapla, ama küçük kalmak istiyorsan kal"** der.

### Karar şeması

| Çocuk ne yapmalı? | Kullan |
|---|---|
| Kalan alanı **tamamen** doldurmalı | `Expanded` |
| Kalan alanı doldurabilir ama **doğal boyutunu koruyabilir** | `Flexible` (varsayılan `loose`) |
| Sabit, doğal boyutunda kalmalı (`Icon`, küçük `Image`) | Sarılmaz |
| Birden fazla esnek çocuk, oransal bölünmeli | `Expanded(flex: 2)` / `Expanded(flex: 1)` |

### Zorunlu kurallar

1. **`Row` / `Column` / `Flex` içinde** `Text`, `TextField`, `TextFormField`, esnek `Container`, `Card`, `ListTile` gibi büyüyebilen her child `Expanded` veya `Flexible` ile sarılmalı.
2. **Oransal bölme** için `MediaQuery.of(context).size.width * 0.5` yerine `Expanded(flex: 1)` kullanılmalı.
3. **`Icon`, `IconButton`, küçük `Image.asset`, `CircleAvatar`** gibi doğal boyutlu widget'lar sarılmaz; ancak yanındaki esnek widget mutlaka `Expanded` olmalı.
4. **`Flexible(fit: FlexFit.tight)` yazmak yerine doğrudan `Expanded` kullanılmalı** (okunabilirlik).
5. **Gereksiz iç içe `Expanded` yasak**: `Expanded(child: Expanded(...))` derleme hatası verir; `Expanded(child: Column(children: [Expanded(...)]))` gibi yapılar mantıklı olabilir ama gözden geçirilmelidir.
6. **`Expanded`/`Flexible` yalnızca `Flex`'in doğrudan child'ı olabilir.** `Row(children: [Padding(child: Expanded(...))])` çalışmaz — `Expanded` üstte olmalı.

### Kötü örnek 1 — Overflow

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(uzunKullaniciAdi), // küçük ekranda overflow
  ],
)
```

**İyi:**

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

### Kötü örnek 2 — MediaQuery ile manuel oran

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

**İyi:**

```dart
Row(
  children: [
    Expanded(flex: 2, child: TextField()),
    const SizedBox(width: 8),
    Expanded(flex: 1, child: ElevatedButton(onPressed: () {}, child: const Text('Ara'))),
  ],
)
```

### Kötü örnek 3 — Yanlış fit seçimi

`Chip`, doğal boyutunda kalmalı ama gerekirse küçülebilmeli:

```dart
Row(
  children: [
    Expanded(child: Chip(label: Text('Etiket'))), // gereksiz yere tüm alanı kaplar
  ],
)
```

**İyi:**

```dart
Row(
  children: [
    Flexible(child: Chip(label: Text('Etiket'))), // doğal boyutta kalır, gerekirse küçülür
  ],
)
```

### Kötü örnek 4 — `Expanded` `Flex`'in doğrudan child'ı değil

```dart
Row(
  children: [
    Padding(
      padding: const EdgeInsets.all(8),
      child: Expanded(child: Text(uzunMetin)), // RUNTIME ERROR
    ),
  ],
)
```

**İyi:**

```dart
Row(
  children: [
    Expanded(
      child: Padding(
        padding: const EdgeInsets.all(8),
        child: Text(uzunMetin),
      ),
    ),
  ],
)
```

---

## 2. SingleChildScrollView + Column

`SingleChildScrollView` çocuğuna **sonsuz yükseklik** verir. Bu yüzden içindeki `Column`'da `Expanded` **çalışmaz** (`Expanded works only when ... has a bounded height`).

**Kötü:**

```dart
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      Expanded(child: ListView(...)), // HATA
    ],
  ),
)
```

**İyi seçenekler:**

- Tam yükseklik gerekiyorsa: `LayoutBuilder` + `ConstrainedBox(minHeight: constraints.maxHeight)` + `IntrinsicHeight`.
- Liste kaydırılabilir olacaksa: `CustomScrollView` + `SliverToBoxAdapter` + `SliverList`.
- Sabit yükseklikli bir liste yeterliyse: `SizedBox(height: 240, child: ListView(...))`.

---

## 3. Column içinde sınırsız yükseklik (ListView / Column)

`Column` çocuğuna sınırsız yükseklik verir; içine `ListView` koymak hata verir.

**Kötü:**

```dart
Column(
  children: [
    Title(),
    ListView(children: [...]), // INFINITE CONSTRAINT
  ],
)
```

**İyi:**

```dart
Column(
  children: [
    Title(),
    Expanded(child: ListView(children: [...])),
  ],
)
```

Alternatif: `ListView(shrinkWrap: true, physics: NeverScrollableScrollPhysics())` — ama performansı kötüdür, yalnızca küçük/sabit liste için.

---

## 4. Wrap kullanımı

Çok sayıda chip / buton / etiket yan yana dizilecekse `Row` taşar. `Wrap` satır sığmadığında otomatik alt satıra geçer.

**Kötü:**

```dart
Row(
  children: kategoriler.map((k) => Chip(label: Text(k))).toList(),
)
```

**İyi:**

```dart
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: kategoriler.map((k) => Chip(label: Text(k))).toList(),
)
```

---

## 5. Breakpoint ve LayoutBuilder

Tek bir layout tüm ekranlara uymaz. Tablet/web için breakpoint kullanılmalı:

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth >= 900) {
      return _TabletLayout();
    }
    return _PhoneLayout();
  },
)
```

**Önerilen breakpoint'ler** (Material 3):
- `< 600` → compact (telefon)
- `600 – 840` → medium (küçük tablet / foldable)
- `> 840` → expanded (tablet / web)

`MediaQuery.of(context).size.width` yerine **`LayoutBuilder.constraints.maxWidth`** tercih edilmeli; çünkü widget'ın **kendi** kullanılabilir alanını verir, ekranın tamamını değil (split-screen, side panel vb.).

---

## 6. FittedBox & AspectRatio

- Yazı boyutu container'a göre küçülmeli/büyümeli mi → `FittedBox(fit: BoxFit.scaleDown, child: Text(...))`.
- Görsel oran (16:9, 1:1) sabit kalmalı mı → `AspectRatio(aspectRatio: 16 / 9, child: ...)`.
- Tek satıra sığması garanti edilmeli mi → `Text(..., maxLines: 1, overflow: TextOverflow.ellipsis)` (FittedBox değil).

---

## 7. SafeArea & Padding

- Sayfa kökünde `SafeArea` eksikse notch / status bar / home indicator altına widget girer.
- `MediaQuery.of(context).padding.top` yerine `SafeArea` kullanılmalı; manuel hesap kırılgandır.

---

## Kontrol listesi (review sırasında)

- [ ] Her `Row` / `Column` / `Flex` çocuğu için: esnek mi? → `Expanded` / `Flexible` var mı?
- [ ] `Text` widget'ları küçük ekranda overflow eder mi? → `Expanded` + `overflow: TextOverflow.ellipsis`
- [ ] `MediaQuery....width * 0.x` ile manuel oran var mı? → `Expanded(flex: x)`
- [ ] `SingleChildScrollView` içindeki `Column`'da `Expanded` var mı? → kaldır.
- [ ] `Column` içinde `ListView` / başka `Column` var mı? → `Expanded` ile sar.
- [ ] Chip / buton dizileri `Row` mu, `Wrap` mı?
- [ ] Tablet / web için breakpoint var mı? (`LayoutBuilder`)
- [ ] Sayfa kökünde `SafeArea` var mı?
- [ ] `Expanded` doğrudan `Flex`'in child'ı mı? (Padding/Container içine sarılmamış)
- [ ] `Flexible(fit: FlexFit.tight)` yerine `Expanded` mi yazılmış?
