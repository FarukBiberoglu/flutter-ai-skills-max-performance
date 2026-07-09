---
name: flutter-ui-reviewer
description: Reviews Flutter UI code against Material/Cupertino best practices, responsive design, and widget-tree optimization.
---

# Flutter UI Reviewer

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution).

## Purpose

Reviews Flutter widget code and provides improvement suggestions on:

- Unnecessary widget-tree depth and `const` usage
- Responsive design (MediaQuery, LayoutBuilder, Flex usage)
- Conformance to Material 3 / Cupertino design rules
- Accessibility (Semantics, contrast, touch-target sizes)
- Consistency of theme and color usage
- Extracting repeated widgets into a shared widget

## When to use

- When a new screen/widget is written
- Before refactoring existing UI code
- During code review

## How it works

1. Reads the specified widget files.
2. Analyzes them against the criteria above.
3. For each finding, provides:
   - File and line reference
   - A description of the problem
   - A suggested fix (with a code example)

## Output format

```
[FILE:LINE] - [CATEGORY] - [LINT_RULE]
Problem: ...
Suggestion: ...
Example:
```dart
// ...
```
```

The `LINT_RULE` field must match the rule name in `analysis_options.yaml` (e.g. `prefer_const_constructors`, `use_key_in_widget_constructors`). Findings outside the scope of a lint rule use the `[CUSTOM]` tag.

## Mandatory lint reference

> **MANDATORY:** Before this skill runs, an `analysis_options.yaml` must exist at the project root and must contain the rule set from [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml).
>
> Skill flow:
> 1. As the first step, read the project-root `analysis_options.yaml`.
> 2. If the file is **missing** → do not start the review; tell the user to copy `examples/analysis_options.yaml` into the project.
> 3. If the file **exists but has missing rules** → list the missing rules before the review and ask for them to be added.
> 4. If all rules are present → start the review and report each finding together with **which lint rule it violates**.
>
> This step cannot be skipped; a review done without `analysis_options.yaml` is considered invalid.

This skill is based on the UI-related rules in `examples/analysis_options.yaml`:

- `use_key_in_widget_constructors`
- `sized_box_for_whitespace`
- `avoid_unnecessary_containers`
- `sort_child_properties_last`
- `prefer_const_constructors`
- `prefer_const_constructors_in_immutables`
- `prefer_const_literals_to_create_immutables`
- `use_build_context_synchronously`
- `no_logic_in_create_state`

To add to your project: [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml)

## Responsive rules

Require the use of **`Flexible` / `Expanded`** for children inside `Row`, `Column`, and `Flex`. Layouts built with fixed `width` / `height` or hardcoded `SizedBox(width: 200)` cause overflow (`RenderFlex overflowed by X pixels`) on small screens.

**Rule:**

- If a `Row` / `Column` contains text, an input, or any widget that can stretch, it must be wrapped in `Expanded` or `Flexible`.
- Use `Expanded(flex: 2)` / `Expanded(flex: 1)` for proportional splits instead of manual math like `MediaQuery.of(context).size.width * 0.66`.
- Children that must keep their natural size (`Icon`, small `Image`) are not wrapped; but the stretchable widget next to them must be `Expanded`.
- The `Flexible(fit: FlexFit.loose)` ↔ `Expanded` (== `Flexible(fit: FlexFit.tight)`) difference must be chosen deliberately: use `Flexible` if the child should keep its own size, `Expanded` if it should fill the remaining space.
- In the `SingleChildScrollView` + `Column` pattern, `Expanded` does not work; there, suggest `IntrinsicHeight` or `ConstrainedBox(minHeight: ...)`.

**Bad example:**

```dart
Row(
  children: [
    Icon(Icons.person),
    SizedBox(width: 8),
    Text(longUserName), // overflows on small screens
  ],
)
```

**Good example:**

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

During the review, every `Text`, `TextField`, and stretchable `Container` inside a `Row` / `Column` is checked for `Expanded` / `Flexible`; if missing, it is reported with the `[RESPONSIVE]` tag.

## Accessibility rules

- **Touch target:** Every tappable element (icon button, `InkWell`, `GestureDetector`) must be at least **48×48 dp**. Small icons are wrapped in `IconButton` (48dp by default) or `SizedBox`/`ConstrainedBox(minWidth/minHeight: 48)`. `[A11Y]`.
- **Semantics:** Actions conveyed by an icon alone (`Icon(Icons.delete)`) get a `Semantics(label: ...)` or `IconButton(tooltip: ...)`; a screen reader must not read an empty button.
- **Contrast:** Text/background contrast must meet WCAG AA (normal text ≥ 4.5:1). Hardcoded light-gray text is bound to contrast-verified tokens via `AppColorConstant`.
- **Text scale:** A fixed `fontSize` inside a fixed-height box overflows when the system font scale grows. Text-bearing boxes must allow growth via `MediaQuery.textScalerOf(context)` (don't set a fixed `height`).
- **Visual-only information:** Color alone must not carry meaning (e.g. red = error only); it must be reinforced with an icon/text.

## Theme and design-token consistency

- **Color:** Never write hardcoded `Color(0xFF...)` / `Colors.red`; colors come from `AppColorConstant.*` or `Theme.of(context).colorScheme.*`. `[THEME]`.
- **TextStyle:** Instead of bare `TextStyle(...)`, use `Theme.of(context).textTheme.<entry>?.copyWith(...)`; typography must be managed from a single source.
- **Spacing:** Repeated magic-number paddings (`EdgeInsets.all(16)`) come from project constants if any exist; inconsistent spacing is reported.
- **Material 3:** Component choices assume `useMaterial3: true`; deprecated widgets like `RaisedButton`/`FlatButton` are redirected to `ElevatedButton`/`TextButton`/`FilledButton`.
- **Shared widget:** If the same visual component (button, card, chip) is repeated in two+ places, extract it into a single `StatelessWidget` under `common/widget/` (see the widget rules in [../clean-architecture-reviewer/SKILL.md](../clean-architecture-reviewer/SKILL.md)). `[REUSE]`.

## Checklist (when the skill runs)

- [ ] Are immutable widgets `const`? (`prefer_const_constructors`)
- [ ] Do widget constructors take `Key? key`? (`use_key_in_widget_constructors`)
- [ ] `SizedBox` instead of a `Container` for whitespace/unnecessary containers? (`avoid_unnecessary_containers`, `sized_box_for_whitespace`)
- [ ] Are stretchable children inside `Row`/`Column` wrapped in `Expanded`/`Flexible`? (`[RESPONSIVE]`)
- [ ] Are icon buttons ≥ 48dp with `Semantics`/`tooltip`? (`[A11Y]`)
- [ ] Do colors come from `AppColorConstant`/`colorScheme` and typography from `textTheme`? (`[THEME]`)
- [ ] Is there a `mounted` guard when using `context` after an await? (`use_build_context_synchronously`)
- [ ] Are repeated visual components extracted into a shared widget? (`[REUSE]`)

## Skill execution protocol

1. Get the UI files via `git status` + `git diff main...HEAD` (or the files the user specified).
2. First apply the **Mandatory lint reference** step (see above); if `analysis_options.yaml` is missing, do not start the review.
3. Evaluate each widget file against the checklist.
4. Output format: `path:line — [CATEGORY/LINT] — problem — suggestion (with code example)`.
5. Group findings by category (Const/rebuild / Responsive / A11Y / Theme / Reuse). Don't print empty groups.
6. Don't make code edits on your own; flag ambiguity explicitly (Think Before Coding — see [../../CLAUDE.md](../../CLAUDE.md)).
