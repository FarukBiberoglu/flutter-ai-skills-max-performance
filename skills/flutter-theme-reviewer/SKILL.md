---
name: flutter-theme-reviewer
description: Reviews and scaffolds Flutter theme management with a layered token architecture. Checks the raw color palette → Material 3 ColorScheme → ThemeData → ThemeExtension (brand colors) → context extension chain, plus the typography (TextTheme) and spacing (const EdgeInsets) systems. Catches hardcoded colors/TextStyles, building ThemeData in a getter, and non-token access.
---

# Flutter Theme Reviewer

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). For general UI rules see [../flutter-ui-reviewer/SKILL.md](../flutter-ui-reviewer/SKILL.md). This skill deepens its "Theme and design tokens" section into a **single-source, layered** architecture.

## 0. Purpose and boundary

This skill covers **theme/token management** only: color, typography, spacing, radius, and their distribution from a single center. **Widget state management (Cubit/State) is not this skill's concern** — the project's architecture is `StatelessWidget` + `Cubit` (see [../clean-architecture-reviewer/SKILL.md](../clean-architecture-reviewer/SKILL.md)). The "View/ViewModel + Mixin" pattern from the reference article this theme layer is drawn from is **not applied** to this project; only the theme layer is taken.

## 1. Folder structure — MANDATORY

The theme is gathered in a single directory; color/style constants are not scattered inside screens.

```
lib/core/theme/            # (in the current project, the color constants under common/constant evolve into this)
├── app_colors.dart            # Raw hex source — light/dark palette (single place)
├── app_color_scheme.dart      # Palette → Material 3 ColorScheme
├── app_theme.dart             # Final ThemeData (lightTheme / darkTheme)
├── app_theme_extension.dart   # Brand colors outside the M3 slot (ThemeExtension)
├── app_text_styles.dart       # Typography → TextTheme
├── app_page_padding.dart      # Spacing primitives (const EdgeInsets)
├── app_radius.dart            # (optional) BorderRadius constants
└── theme_context_extension.dart  # context.colorScheme / context.appTheme access
```

> **Current project note:** There are already `AppColorConstant` and `AppFontConstant` under `app/common/constant/`. This skill does **not** rename them in one shot (Surgical Changes). New theme code is written against this layered structure; existing constants are migrated gradually. A bulk rename is done only if the user explicitly asks.

## 2. Layer chain — one direction, single source

```
AppColors (hex)  →  AppColorScheme (M3)  →  AppTheme (ThemeData)
                          +                       +
              AppThemeExtension (brand)   AppTextStyles / AppPagePadding
                          ↓
              context extension (ergonomic access)  →  widgets
```

Each layer knows only the one below it. A widget does not read `AppColors.light.primary` directly; it reads from `context.colorScheme.primary`. `[THEME_DIRECT_PALETTE]`.

### 2.1 `AppColors` — single hex source

```dart
final class AppColors {
  AppColors._();
  static final _LightPalette light = _LightPalette();
  static final _DarkPalette dark = _DarkPalette();
}
```

There is **no raw `Color(0xFF...)` anywhere else** in the app. If hex is seen outside a widget/theme file, that's `[THEME_HARDCODED_COLOR]`.

### 2.2 `AppColorScheme` — Material 3 bridge

Maps the palette to a full M3 `ColorScheme` (`primary`, `onPrimary`, `secondary`, `surface`, `onSurface`, `error`...). This way every screen reads from `Theme.of(context).colorScheme`, and dark mode is a **single property** switch.

```dart
static ColorScheme get light => ColorScheme(
  brightness: Brightness.light,
  primary: AppColors.light.primary,
  onPrimary: AppColors.light.onPrimary,
  surface: AppColors.light.surface,
  error: AppColors.light.error,
  // ... full M3 coverage
);
```

### 2.3 `AppTheme` — final composition — **the getter trap**

`ThemeData` is built **once** and lives for the app's lifetime. A getter that recomputes on every read re-produces the entire `ThemeData` graph on rebuilds.

**Bad (rebuilds on every read):**

```dart
static ThemeData get lightTheme => ThemeData(useMaterial3: true, /* ... */);
```

**Good (canonicalized once):**

```dart
final class AppTheme {
  AppTheme._();
  static final ThemeData lightTheme = _build(AppColorScheme.light, AppThemeExtension.light);
  static final ThemeData darkTheme  = _build(AppColorScheme.dark,  AppThemeExtension.dark);

  static ThemeData _build(ColorScheme scheme, AppThemeExtension ext) => ThemeData(
    useMaterial3: true,
    colorScheme: scheme,
    textTheme: AppTextStyles.textTheme,
    extensions: <ThemeExtension<dynamic>>[ext],
  );
}
```

If a theme built with `static ThemeData get` is seen, that's `[THEME_GETTER]` — it should become a `static final` field.

### 2.4 `AppThemeExtension` — brand colors outside M3

Colors with no counterpart in Material 3's slot system — `success`, `warning`, `shimmer`, `cardBackground` — are carried via `ThemeExtension`. Implement `copyWith` **and** `lerp` (`Color.lerp` for each field) — thanks to `lerp`, theme transitions animate.

```dart
final class AppThemeExtension extends ThemeExtension<AppThemeExtension> {
  const AppThemeExtension({required this.success, required this.warning, required this.cardBackground});
  final Color success;
  final Color warning;
  final Color cardBackground;

  @override
  AppThemeExtension copyWith({Color? success, Color? warning, Color? cardBackground}) => AppThemeExtension(
    success: success ?? this.success,
    warning: warning ?? this.warning,
    cardBackground: cardBackground ?? this.cardBackground,
  );

  @override
  AppThemeExtension lerp(ThemeExtension<AppThemeExtension>? other, double t) {
    if (other is! AppThemeExtension) return this;
    return AppThemeExtension(
      success: Color.lerp(success, other.success, t)!,
      warning: Color.lerp(warning, other.warning, t)!,
      cardBackground: Color.lerp(cardBackground, other.cardBackground, t)!,
    );
  }

  static const light = AppThemeExtension(/* ... */);
  static const dark  = AppThemeExtension(/* ... */);
}
```

If `lerp` is missing or half-implemented (some fields return a constant instead of using `t`), that's `[THEME_EXTENSION_LERP]`.

### 2.5 Context extension — access ergonomics

```dart
extension ThemeContext on BuildContext {
  ColorScheme get colorScheme => Theme.of(this).colorScheme;
  TextTheme get textTheme => Theme.of(this).textTheme;
  AppThemeExtension get appTheme => Theme.of(this).extension<AppThemeExtension>()!;
}
// usage: context.colorScheme.primary  /  context.appTheme.success
```

## 3. Typography — `AppTextStyles` → `TextTheme`

- The typography scale is mapped to `TextTheme` entries (`displayLarge`, `headlineSmall`, `bodyMedium`, `labelSmall`...).
- Widgets use `context.textTheme.bodyMedium?.copyWith(...)`; **bare `TextStyle(...)` is not written.** `[THEME_BARE_TEXTSTYLE]`.
- Color override is done via `copyWith(color: context.colorScheme.error)`, not a hardcoded color.

## 4. Spacing — `AppPagePadding` (const `EdgeInsets`)

An `EdgeInsets` subclass with `const` constructors → canonicalized at compile time, zero allocation. A 4-multiple ladder is enforced.

```dart
final class AppPagePadding extends EdgeInsets {
  const AppPagePadding.all16() : super.all(_m);
  const AppPagePadding.horizontalSymmetric() : super.symmetric(horizontal: _l);
  static const double _m = 16;
  static const double _l = 20;
}
```

Magic-number paddings sprinkled across screens like `EdgeInsets.all(16)` are redirected to these constants. `[THEME_SPACING]`.

## 5. Review tags

```
[THEME_DIRECT_PALETTE]  — a widget accesses the palette/AppColors directly (should use context.colorScheme)
[THEME_HARDCODED_COLOR] — raw Color(0xFF...) / Colors.* outside the theme directory
[THEME_GETTER]          — ThemeData built with a static getter (should be static final)
[THEME_EXTENSION_LERP]  — lerp/copyWith missing or half-done in a ThemeExtension
[THEME_BARE_TEXTSTYLE]  — bare TextStyle(...) instead of textTheme
[THEME_SPACING]         — magic-number padding that should bind to a spacing constant
[THEME_M3]              — deprecated widget (RaisedButton/FlatButton) or missing useMaterial3
```

## 6. Review checklist (when the skill runs)

1. **Single source:** Is raw hex only inside `app_colors.dart`? Is there any `Color(0xFF...)` elsewhere?
2. **Access:** Do widgets read colors from `context.colorScheme.*` / `context.appTheme.*`, not directly from the palette?
3. **ThemeData lifetime:** Is `AppTheme.lightTheme` a `static final` (not a getter)?
4. **ThemeExtension:** Are brand colors in an extension without being forced into an M3 slot? Are `copyWith` + `lerp` complete?
5. **Typography:** Is it `context.textTheme.<entry>?.copyWith(...)`, with no bare `TextStyle`?
6. **Spacing:** Are paddings from `AppPagePadding` constants, not magic numbers?
7. **Dark mode:** Are both light and dark defined; are `MaterialApp`'s `theme` + `darkTheme` + `themeMode` wired correctly?
8. **M3:** Is `useMaterial3: true` present, with no deprecated components?

## 7. Skill execution protocol

1. Get the theme/UI files via `git status` + `git diff main...HEAD` (or the specified files).
2. Evaluate each changed file against the section 6 checklist; skip unrelated files.
3. Output: `path:line — [TAG] — problem — suggestion (with code example)`.
4. Group findings (Source/Access / ThemeData / Extension / Typography / Spacing). Don't print empty groups.
5. Don't do a **bulk rename** of existing constants like `AppColorConstant`; suggest a gradual migration and leave the decision to the user (Surgical Changes — see [../../CLAUDE.md](../../CLAUDE.md)).
