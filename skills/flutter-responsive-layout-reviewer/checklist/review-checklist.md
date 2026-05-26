# Responsive Layout Review Checklist

Review sırasında bu listeyi yukarıdan aşağıya uygula. Her madde başarısızsa ilgili etiketi raporla.

## Flex (Row / Column / Flex)

- [ ] Her `Row` / `Column` çocuğu için: esneyebilen mi? → `Expanded` / `Flexible` var mı? `[FLEX-MISSING]`
- [ ] `Text` widget'ları küçük ekranda overflow eder mi? → `Expanded` + `overflow: TextOverflow.ellipsis` `[FLEX-MISSING]`
- [ ] `Expanded` ile sarılmış doğal-boyutlu widget (Chip, Icon) var mı? → `Flexible` veya hiç sarma `[FLEX-WRONG-FIT]`
- [ ] İç içe `Expanded` var mı? → düzleştir `[FLEX-NESTED]`
- [ ] `Expanded` doğrudan `Flex`'in çocuğu mu? (Padding/Container içine sarılmamış) `[FLEX-MISSING]`
- [ ] `Flexible(fit: FlexFit.tight)` yerine `Expanded` mı? → değiştir

## Sabit boyut & MediaQuery

- [ ] `SizedBox(width: 200)` ile sabit genişlik var mı? Esnek olmalı mı? `[FIXED-SIZE]`
- [ ] `MediaQuery.of(context).size.width * 0.x` ile manuel oran var mı? → `Expanded(flex: x)` `[MEDIAQUERY-MISUSE]`
- [ ] Genişlik kararları `LayoutBuilder.constraints` yerine `MediaQuery` ile mi alınıyor? `[MEDIAQUERY-MISUSE]`

## Scroll & sonsuz constraint

- [ ] `SingleChildScrollView` içindeki `Column`'da `Expanded` var mı? → kaldır veya yapıyı değiştir `[SCROLL-EXPANDED]`
- [ ] `Column` içinde `ListView` / başka `Column` var mı? → `Expanded` ile sar `[INFINITE-CONSTRAINT]`
- [ ] `Row` içinde yatay `ListView` var mı? → `Expanded` ile sar `[INFINITE-CONSTRAINT]`

## Wrap & dinamik listeler

- [ ] `Row` içinde `.map(...).toList()` ile dinamik chip/buton dizisi var mı? → `Wrap` veya yatay `ListView` `[WRAP-MISSING]`

## Breakpoint & layout dallanma

- [ ] Tablet / web için breakpoint var mı? (`LayoutBuilder` ile dallanma) `[BREAKPOINT]`
- [ ] Sihirli sayılar (`if (width > 768)`) var mı? → tek bir `Breakpoints` sınıfı `[BREAKPOINT]`
- [ ] Sadece portrait varsayılmış mı? → orientation / aspect ratio için dallanma `[ORIENTATION]`

## SafeArea & boundary

- [ ] Sayfa kökünde `SafeArea` var mı? `[SAFEAREA-MISSING]`
- [ ] `MediaQuery.padding.top` ile manuel inset hesabı var mı? → `SafeArea` `[SAFEAREA-MISSING]`

## FittedBox / AspectRatio

- [ ] Uzun metin için `FittedBox` mı kullanılmış? → çoğunlukla `ellipsis` daha iyi `[FITTEDBOX-MISUSE]`
- [ ] Görsel oranlı kart için `AspectRatio` var mı? `[ASPECT-RATIO]`

## Lint referansı

`examples/analysis_options.yaml` içinde aşağıdaki kurallar aktif olmalı:

- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `use_key_in_widget_constructors`
- `prefer_const_constructors`

Detay: [../rules/](../rules/), [../bad_examples/](../bad_examples/), [../good_examples/](../good_examples/).
