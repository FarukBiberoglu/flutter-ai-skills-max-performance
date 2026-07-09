# Rule: Scroll and infinite constraints

`Expanded` only works on a **bounded** axis. Scroll widgets give their children an **unbounded** axis.

## Expanded inside SingleChildScrollView + Column

`SingleChildScrollView` gives its child **infinite height** → an `Expanded` inside its `Column` doesn't work.

Error: `RenderFlex children have non-zero flex but incoming height constraints are unbounded.`

### Solution options

**(a) Full height, scroll optional**

```dart
LayoutBuilder(
  builder: (context, constraints) => SingleChildScrollView(
    child: ConstrainedBox(
      constraints: BoxConstraints(minHeight: constraints.maxHeight),
      child: IntrinsicHeight(
        child: Column(
          children: [
            Header(),
            Expanded(child: Body()), // now it works
            Footer(),
          ],
        ),
      ),
    ),
  ),
)
```

**(b) Sliver structure (recommended)**

```dart
CustomScrollView(
  slivers: [
    SliverToBoxAdapter(child: Header()),
    SliverList(delegate: SliverChildBuilderDelegate(...)),
    SliverToBoxAdapter(child: Footer()),
  ],
)
```

**(c) Fixed height**

```dart
Column(children: [
  Header(),
  SizedBox(height: 240, child: ListView(...)),
  Footer(),
])
```

## ListView / another Column inside a Column

A `Column` gives its children **infinite height**. If there's a vertically scrolling widget inside, it errors.

```dart
// WRONG
Column(children: [
  Title(),
  ListView(children: [...]), // ❌ infinite constraint
])

// CORRECT
Column(children: [
  Title(),
  Expanded(child: ListView(children: [...])), // ✅
])
```

### The `shrinkWrap` alternative

```dart
ListView(
  shrinkWrap: true,
  physics: NeverScrollableScrollPhysics(),
  children: [...],
)
```

- **Advantage:** No `Expanded` needed; it takes its natural height inside the `Column`.
- **Disadvantage:** The list lays out all its children on every build (it's not lazy). Use only for **short, fixed** lists.
- General rule: if it will scroll, `Expanded(child: ListView.builder(...))`; if it won't scroll and is short, `shrinkWrap` or `...items` directly inside the `Column`.

## Row inside a Row / horizontal scroll

The same problem on the horizontal axis:

```dart
// WRONG
Row(children: [
  Icon(...),
  ListView(scrollDirection: Axis.horizontal, ...), // ❌
])

// CORRECT
Row(children: [
  Icon(...),
  Expanded(
    child: ListView(scrollDirection: Axis.horizontal, ...),
  ),
])
```

## Review tags

- `[SCROLL-EXPANDED]` — the `SingleChildScrollView` → `Column` → `Expanded` chain.
- `[INFINITE-CONSTRAINT]` — a widget scrolling on an unbounded axis inside a `Column` / `Row`.

Examples: [../bad_examples/05-scroll-expanded.md](../bad_examples/05-scroll-expanded.md), [../bad_examples/06-column-listview.md](../bad_examples/06-column-listview.md).
