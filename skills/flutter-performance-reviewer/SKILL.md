---
name: flutter-performance-reviewer
description: Detects performance bottlenecks (rebuilds, jank, memory leaks) in Flutter apps and suggests optimizations.
---

# Flutter Performance Reviewer

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution).

## Purpose

Reviews Flutter code for performance:

- Unnecessary `setState` and rebuild chains
- Missing `const` constructors
- List performance mistakes like using `ListView` instead of `ListView.builder`
- Image cache, asset size, and `precacheImage` usage
- Misuse of `FutureBuilder` / `StreamBuilder`
- Unnecessary listening with state management (Provider, Riverpod, Bloc)
- `Controller`, `StreamSubscription`, `AnimationController` that are not disposed
- Heavy work in the build method (JSON parse, sorting, computation)

## When to use

- When the app experiences jank
- When examining DevTools profiler output
- When writing list/scroll screens

## Output format

```
[SEVERITY: High/Medium/Low]
[FILE:LINE]
Problem: ...
Impact: ... (FPS drop, memory leak, etc.)
Fix: ...
```

## Checklist

- [ ] Are all immutable widgets `const`?
- [ ] Do list widgets use `builder`?
- [ ] Are controllers disposed?
- [ ] Is the build method pure?
- [ ] Are images loaded at the right size?

## Reference lint rules

This skill is based on the performance and memory rules in `examples/analysis_options.yaml`:

**Performance:**
- `avoid_print`, `avoid_slow_async_io`
- `prefer_const_constructors`, `prefer_const_constructors_in_immutables`
- `prefer_const_declarations`, `prefer_const_literals_to_create_immutables`
- `unnecessary_const`, `unnecessary_lambdas`
- `avoid_unnecessary_containers`, `sized_box_for_whitespace`
- `prefer_final_fields`, `prefer_final_in_for_each`, `prefer_final_locals`

**Memory management:**
- `close_sinks` — closing StreamController
- `cancel_subscriptions` — cancelling subscriptions
- `avoid_returning_null_for_future`

**Flutter-specific:**
- `use_build_context_synchronously`
- `no_logic_in_create_state`
- `avoid_web_libraries_in_flutter`

Full config: [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml)

## State management (Cubit + State) rules

This skill reviews `flutter_bloc` `Cubit` usage against the following rules. The same principles (immutability, equality, selective rebuild) apply to projects using Riverpod / Provider.

### 1. The state class must be `Equatable`

When a Cubit calls `emit(newState)`, `Bloc`/`Cubit` compares the new state to the old one with `==`. Without `Equatable`, every `emit` counts as **new** due to reference inequality, and `BlocBuilder` rebuilds unnecessarily.

**Bad:**

```dart
class CounterState {
  final int count;
  final bool isLoading;
  CounterState({required this.count, required this.isLoading});
}
```

**Good:**

```dart
class CounterState extends Equatable {
  const CounterState({
    required this.count,
    required this.isLoading,
  });

  final int count;
  final bool isLoading;

  @override
  List<Object?> get props => [count, isLoading];
}
```

**All** of the state's fields must be added to the `props` list. A forgotten field = a silent "UI not updating" bug.

### 2. State must be immutable + updated via `copyWith`

State fields must be `final` and the class must have a `const` constructor. Updates only via `copyWith`:

```dart
class HomeState extends Equatable {
  const HomeState({
    this.isLoading = false,
    this.items = const [],
    this.errorMessage,
  });

  final bool isLoading;
  final List<Item> items;
  final String? errorMessage;

  HomeState copyWith({
    bool? isLoading,
    List<Item>? items,
    String? errorMessage,
    bool clearError = false,
  }) {
    return HomeState(
      isLoading: isLoading ?? this.isLoading,
      items: items ?? this.items,
      errorMessage: clearError ? null : (errorMessage ?? this.errorMessage),
    );
  }

  @override
  List<Object?> get props => [isLoading, items, errorMessage];
}
```

**Rules:**

- **Mutation is forbidden** inside the Cubit, e.g. `state.items.add(x)` — create a new list: `[...state.items, x]`.
- Setting a field to `null` via `copyWith(errorMessage: null)` doesn't work (`?? this.errorMessage` swallows it). Use an explicit flag like `clearError: true`.
- Default values must be `const []`, `const {}`; creating a new empty list on every constructor call produces unnecessary allocation.

### 3. State comparison before `emit`

`Cubit` already skips notifying listeners if the same state is re-emitted; with `Equatable` set up correctly, that's **free performance**. There's no need to write a manual `if (state == newState) return;` — but without `Equatable`, this mechanism does not work.

### 4. Selective listening instead of `BlocBuilder`

Instead of a `BlocBuilder` that listens to the whole state, rebuild only for the field of interest:

- `BlocSelector<C, S, T>` — for a single field.
- `BlocBuilder` + `buildWhen: (prev, curr) => prev.x != curr.x`.
- `BlocListener` for side effects only (snackbar, navigation); no rebuild needed.

### 5. Loading / Done sealed wrapper (optional but recommended)

Instead of per-feature `bool isLoading` flags, prefer a sealed wrapper for async data (e.g. `CubitDataModel<T>` → `Loading<T>` / `Done<T>`). This lets the UI branch from a single point with `state.data.when(loading: ..., done: ...)` and makes the "loading stuck at true" bug impossible.

### 6. Cubit lifecycle

- `TextEditingController`, `ScrollController`, `StreamSubscription` held inside a Cubit must be disposed by overriding `close()`.
- `lint: close_sinks` and `cancel_subscriptions` catch this, but it's also checked manually during review.
- The Cubit must be created inside `BlocProvider` `create:`; a Cubit passed via `value:` remains the disposal responsibility of its owner.

### Review tags

State-management findings are reported with this tag:

```
[STATE] - [Missing Equatable | No copyWith | Mutation | Missing selective rebuild | Missing dispose]
```
