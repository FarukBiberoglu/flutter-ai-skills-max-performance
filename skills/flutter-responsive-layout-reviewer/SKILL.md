---
name: flutter-responsive-layout-reviewer
description: Reviews Flutter layout code for responsive design and especially Flexible / Expanded usage; catches overflow and fixed-size bugs.
---

# Flutter Responsive Layout Reviewer

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution).

## Skill structure

This skill has a modular structure; this SKILL.md holds the full content, and the sub-folders are extended reference:

- [rules/](rules/) — a detailed explanation of each rule
  - [flexible-vs-expanded.md](rules/flexible-vs-expanded.md) **(core rule)**
  - [scroll-and-infinite-constraints.md](rules/scroll-and-infinite-constraints.md)
  - [wrap-and-row.md](rules/wrap-and-row.md)
  - [breakpoints-and-layoutbuilder.md](rules/breakpoints-and-layoutbuilder.md)
  - [safearea-fittedbox-aspectratio.md](rules/safearea-fittedbox-aspectratio.md)
- [bad_examples/](bad_examples/) — real-world bad examples and why they break
- [good_examples/](good_examples/) — the correct usages that correspond to bad_examples
- [checklist/review-checklist.md](checklist/review-checklist.md) — an item-by-item list to apply during review

When reviewing: first walk through [checklist/review-checklist.md](checklist/review-checklist.md), open the relevant `rules/` file when in doubt, and look at the paired `bad_examples/` ↔ `good_examples/` files when you want an example.

## Purpose

Reviews Flutter layout code for **responsiveness**. Main focus: **is `Flexible` and `Expanded` used correctly?** Secondarily, it checks the appropriate use of tools like `MediaQuery`, `LayoutBuilder`, `FittedBox`, `Wrap`, `IntrinsicHeight`, and `AspectRatio`.

Goal: a UI that produces no `RenderFlex overflowed by X pixels` or `BoxConstraints forces an infinite ...` error on **any device**, from small phones to tablet/foldable/web screens.

## When to use

- When a new screen/widget is written
- When you get a `RenderFlex overflowed` warning
- Before adding tablet / web / foldable support
- Before refactoring existing Row / Column / Stack trees
- During code review

## How it works

1. Reads the specified widget files.
2. Scans `Row`, `Column`, `Flex`, `Wrap`, `Stack`, `ListView`, `GridView`, `SingleChildScrollView`, `CustomScrollView` widgets.
3. Checks them against the rule list below.
4. For each finding, gives a file:line reference, a tag, a problem description, and a fix example.

## Output format

```
[FILE:LINE] - [TAG] - [SEVERITY: High/Medium/Low]
Problem: ...
Impact: ... (overflow, infinite constraint, small-screen overflow, etc.)
Suggestion: ...
Example:
```dart
// ...
```
```

### Tag set

- `[FLEX-MISSING]` — A stretchable child is not wrapped in `Expanded` / `Flexible`.
- `[FLEX-WRONG-FIT]` — `Expanded` used instead of `Flexible` (or vice versa).
- `[FLEX-NESTED]` — Unnecessary nested `Expanded` (e.g. `Expanded(child: Expanded(...))`).
- `[FIXED-SIZE]` — An area that should be flexible is constrained with a fixed `width` / `height` / `SizedBox(width: 200)`.
- `[MEDIAQUERY-MISUSE]` — Manual ratio math like `MediaQuery.of(context).size.width * 0.66` (use `Expanded(flex: ...)` instead).
- `[SCROLL-EXPANDED]` — `Expanded` used inside `SingleChildScrollView` + `Column` (doesn't work).
- `[INFINITE-CONSTRAINT]` — A `ListView` / `Column` with unbounded height inside a `Column` (missing `Expanded` or `shrinkWrap`).
- `[WRAP-MISSING]` — Many chips/buttons side by side; `Row` used instead of `Wrap`.
- `[ORIENTATION]` — Only portrait assumed; missing branching with `OrientationBuilder` / `LayoutBuilder`.
- `[BREAKPOINT]` — No tablet/web breakpoint; a single layout applied to all screens.

## Mandatory lint reference

> **MANDATORY:** Before this skill runs, an `analysis_options.yaml` must exist at the project root and must contain the rule set from [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml).
>
> 1. As the first step, the skill reads the project-root `analysis_options.yaml`.
> 2. If the file is **missing** → the review is not started; the user is asked to copy `examples/analysis_options.yaml` into the project.
> 3. If the file **exists but has missing rules** → the missing rules are listed.
> 4. If all rules are present → the review starts.

Relevant lint rules:

- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `use_key_in_widget_constructors`
- `prefer_const_constructors`

Layout correctness is an area lint cannot catch; this skill uses `[CUSTOM]` and the tag set above.

---

## 1. Flexible vs Expanded — CORE RULE

> `Expanded` ≡ `Flexible(fit: FlexFit.tight)`.
> That is, `Expanded` tells the child **"fill all the remaining space"**; `Flexible(fit: FlexFit.loose)` says **"take up to this much space, but stay small if you want to"**.

### Decision table

| What should the child do? | Use |
|---|---|
| Fill the remaining space **completely** | `Expanded` |
| Can fill the remaining space but may **keep its natural size** | `Flexible` (default `loose`) |
| Must stay fixed at its natural size (`Icon`, small `Image`) | Don't wrap |
| Multiple flexible children, split proportionally | `Expanded(flex: 2)` / `Expanded(flex: 1)` |

### Mandatory rules

1. **Inside a `Row` / `Column` / `Flex`,** every child that can grow — `Text`, `TextField`, `TextFormField`, a stretchable `Container`, `Card`, `ListTile` — must be wrapped in `Expanded` or `Flexible`.
2. For **proportional splits**, use `Expanded(flex: 1)` instead of `MediaQuery.of(context).size.width * 0.5`.
3. Natural-size widgets like **`Icon`, `IconButton`, small `Image.asset`, `CircleAvatar`** are not wrapped; but the flexible widget next to them must be `Expanded`.
4. **Use `Expanded` directly instead of writing `Flexible(fit: FlexFit.tight)`** (readability).
5. **Unnecessary nested `Expanded` is forbidden**: `Expanded(child: Expanded(...))` is a compile error; structures like `Expanded(child: Column(children: [Expanded(...)]))` can be valid but should be reviewed.
6. **`Expanded`/`Flexible` can only be a direct child of a `Flex`.** `Row(children: [Padding(child: Expanded(...))])` doesn't work — `Expanded` must be on top.

### Bad example 1 — Overflow

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(longUserName), // overflows on small screens
  ],
)
```

**Good:**

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        longUserName,
        overflow: TextOverflow.ellipsis,
      ),
    ),
  ],
)
```

### Bad example 2 — Manual ratio with MediaQuery

```dart
Row(
  children: [
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.66,
      child: TextField(),
    ),
    SizedBox(
      width: MediaQuery.of(context).size.width * 0.34,
      child: ElevatedButton(onPressed: () {}, child: Text('Search')),
    ),
  ],
)
```

**Good:**

```dart
Row(
  children: [
    Expanded(flex: 2, child: TextField()),
    const SizedBox(width: 8),
    Expanded(flex: 1, child: ElevatedButton(onPressed: () {}, child: const Text('Search'))),
  ],
)
```

### Bad example 3 — Wrong fit choice

A `Chip` should keep its natural size but be able to shrink if needed:

```dart
Row(
  children: [
    Expanded(child: Chip(label: Text('Label'))), // needlessly fills the whole space
  ],
)
```

**Good:**

```dart
Row(
  children: [
    Flexible(child: Chip(label: Text('Label'))), // keeps natural size, shrinks if needed
  ],
)
```

### Bad example 4 — `Expanded` not a direct child of `Flex`

```dart
Row(
  children: [
    Padding(
      padding: const EdgeInsets.all(8),
      child: Expanded(child: Text(longText)), // RUNTIME ERROR
    ),
  ],
)
```

**Good:**

```dart
Row(
  children: [
    Expanded(
      child: Padding(
        padding: const EdgeInsets.all(8),
        child: Text(longText),
      ),
    ),
  ],
)
```

---

## 2. SingleChildScrollView + Column

`SingleChildScrollView` gives its child **infinite height**. So `Expanded` **doesn't work** inside a `Column` there (`Expanded works only when ... has a bounded height`).

**Bad:**

```dart
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      Expanded(child: ListView(...)), // ERROR
    ],
  ),
)
```

**Good options:**

- If full height is needed: `LayoutBuilder` + `ConstrainedBox(minHeight: constraints.maxHeight)` + `IntrinsicHeight`.
- If the list should be scrollable: `CustomScrollView` + `SliverToBoxAdapter` + `SliverList`.
- If a fixed-height list is enough: `SizedBox(height: 240, child: ListView(...))`.

---

## 3. Unbounded height inside a Column (ListView / Column)

A `Column` gives its child unbounded height; putting a `ListView` inside it errors out.

**Bad:**

```dart
Column(
  children: [
    Title(),
    ListView(children: [...]), // INFINITE CONSTRAINT
  ],
)
```

**Good:**

```dart
Column(
  children: [
    Title(),
    Expanded(child: ListView(children: [...])),
  ],
)
```

Alternative: `ListView(shrinkWrap: true, physics: NeverScrollableScrollPhysics())` — but it has poor performance, only for a small/fixed list.

---

## 4. Using Wrap

If many chips / buttons / labels are laid out side by side, a `Row` overflows. `Wrap` automatically moves to the next line when the row doesn't fit.

**Bad:**

```dart
Row(
  children: categories.map((c) => Chip(label: Text(c))).toList(),
)
```

**Good:**

```dart
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: categories.map((c) => Chip(label: Text(c))).toList(),
)
```

---

## 5. Breakpoints and LayoutBuilder

A single layout doesn't fit every screen. Use breakpoints for tablet/web:

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

**Recommended breakpoints** (Material 3):
- `< 600` → compact (phone)
- `600 – 840` → medium (small tablet / foldable)
- `> 840` → expanded (tablet / web)

Prefer **`LayoutBuilder.constraints.maxWidth`** over `MediaQuery.of(context).size.width`; because it gives the widget's **own** available space, not the entire screen (split-screen, side panel, etc.).

---

## 6. FittedBox & AspectRatio

- Should the text scale down/up with the container → `FittedBox(fit: BoxFit.scaleDown, child: Text(...))`.
- Should a visual ratio (16:9, 1:1) stay fixed → `AspectRatio(aspectRatio: 16 / 9, child: ...)`.
- Must it be guaranteed to fit on one line → `Text(..., maxLines: 1, overflow: TextOverflow.ellipsis)` (not FittedBox).

---

## 7. SafeArea & Padding

- If `SafeArea` is missing at the page root, widgets slide under the notch / status bar / home indicator.
- Use `SafeArea` instead of `MediaQuery.of(context).padding.top`; manual math is fragile.

---

## Checklist (during review)

- [ ] For each `Row` / `Column` / `Flex` child: is it flexible? → is there an `Expanded` / `Flexible`?
- [ ] Do `Text` widgets overflow on small screens? → `Expanded` + `overflow: TextOverflow.ellipsis`
- [ ] Is there manual ratio via `MediaQuery....width * 0.x`? → `Expanded(flex: x)`
- [ ] Is there an `Expanded` in a `Column` inside a `SingleChildScrollView`? → remove it.
- [ ] Is there a `ListView` / another `Column` inside a `Column`? → wrap with `Expanded`.
- [ ] Are chip / button arrays a `Row` or a `Wrap`?
- [ ] Is there a breakpoint for tablet / web? (`LayoutBuilder`)
- [ ] Is there a `SafeArea` at the page root?
- [ ] Is `Expanded` a direct child of `Flex`? (not wrapped in a Padding/Container)
- [ ] Is `Expanded` written instead of `Flexible(fit: FlexFit.tight)`?
