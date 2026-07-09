# Good example: Scroll + dynamic list

**Solution A — CustomScrollView (recommended):**

```dart
CustomScrollView(
  slivers: [
    const SliverToBoxAdapter(child: Header()),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (_, i) => Text('Item $i'),
        childCount: 100,
      ),
    ),
  ],
)
```

**Solution B — full-height guarantee, scroll optional:**

```dart
LayoutBuilder(
  builder: (context, constraints) => SingleChildScrollView(
    child: ConstrainedBox(
      constraints: BoxConstraints(minHeight: constraints.maxHeight),
      child: IntrinsicHeight(
        child: Column(
          children: [
            const Header(),
            Expanded(child: Body()),
            const Footer(),
          ],
        ),
      ),
    ),
  ),
)
```

**Solution C — fixed height (simple but not flexible):**

```dart
Column(
  children: [
    const Header(),
    SizedBox(
      height: 320,
      child: ListView.builder(
        itemBuilder: (_, i) => Text('Item $i'),
        itemCount: 100,
      ),
    ),
  ],
)
```

**Which one?**
- If the list is **long and part of the page scroll** → A.
- If the list is **short, the page shouldn't scroll, and the content should fill the screen** → B.
- If the list **occupies a fixed area** → C.

**Comparison:** [../bad_examples/05-scroll-expanded.md](../bad_examples/05-scroll-expanded.md)
