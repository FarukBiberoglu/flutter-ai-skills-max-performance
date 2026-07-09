# Responsive Layout Review Checklist

Apply this list top to bottom during review. If an item fails, report the relevant tag.

## Flex (Row / Column / Flex)

- [ ] For each `Row` / `Column` child: is it stretchable? → is there an `Expanded` / `Flexible`? `[FLEX-MISSING]`
- [ ] Do `Text` widgets overflow on small screens? → `Expanded` + `overflow: TextOverflow.ellipsis` `[FLEX-MISSING]`
- [ ] Is there a natural-size widget (Chip, Icon) wrapped in `Expanded`? → `Flexible` or don't wrap `[FLEX-WRONG-FIT]`
- [ ] Is there a nested `Expanded`? → flatten it `[FLEX-NESTED]`
- [ ] Is `Expanded` a direct child of `Flex`? (not wrapped in a Padding/Container) `[FLEX-MISSING]`
- [ ] Is it `Expanded` instead of `Flexible(fit: FlexFit.tight)`? → change it

## Fixed size & MediaQuery

- [ ] Is there a fixed width via `SizedBox(width: 200)`? Should it be flexible? `[FIXED-SIZE]`
- [ ] Is there manual ratio via `MediaQuery.of(context).size.width * 0.x`? → `Expanded(flex: x)` `[MEDIAQUERY-MISUSE]`
- [ ] Are width decisions made with `MediaQuery` instead of `LayoutBuilder.constraints`? `[MEDIAQUERY-MISUSE]`

## Scroll & infinite constraint

- [ ] Is there an `Expanded` in a `Column` inside a `SingleChildScrollView`? → remove it or change the structure `[SCROLL-EXPANDED]`
- [ ] Is there a `ListView` / another `Column` inside a `Column`? → wrap with `Expanded` `[INFINITE-CONSTRAINT]`
- [ ] Is there a horizontal `ListView` inside a `Row`? → wrap with `Expanded` `[INFINITE-CONSTRAINT]`

## Wrap & dynamic lists

- [ ] Is there a dynamic chip/button array via `.map(...).toList()` inside a `Row`? → `Wrap` or a horizontal `ListView` `[WRAP-MISSING]`

## Breakpoint & layout branching

- [ ] Is there a breakpoint for tablet / web? (branching with `LayoutBuilder`) `[BREAKPOINT]`
- [ ] Are there magic numbers (`if (width > 768)`)? → a single `Breakpoints` class `[BREAKPOINT]`
- [ ] Is only portrait assumed? → branch for orientation / aspect ratio `[ORIENTATION]`

## SafeArea & boundary

- [ ] Is there a `SafeArea` at the page root? `[SAFEAREA-MISSING]`
- [ ] Is there manual inset math via `MediaQuery.padding.top`? → `SafeArea` `[SAFEAREA-MISSING]`

## FittedBox / AspectRatio

- [ ] Is `FittedBox` used for long text? → `ellipsis` is usually better `[FITTEDBOX-MISUSE]`
- [ ] Is there an `AspectRatio` for a ratio-based image card? `[ASPECT-RATIO]`

## Lint reference

The following rules must be active in `examples/analysis_options.yaml`:

- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `use_key_in_widget_constructors`
- `prefer_const_constructors`

Detail: [../rules/](../rules/), [../bad_examples/](../bad_examples/), [../good_examples/](../good_examples/).
