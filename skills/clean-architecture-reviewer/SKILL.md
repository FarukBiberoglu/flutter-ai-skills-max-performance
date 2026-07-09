---
name: clean-architecture-reviewer
description: Reviews code against the Clean Architecture layers used in Flutter projects (core / app, data / presentation, and the datasource → repository → cubit → view flow). Checks whether a newly added feature follows the folder structure, the DI registration order, the ApiResponseModel → DataResult → CubitDataModel conversion chain, and the naming conventions.
---

# Clean Architecture Review

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution).

This skill takes the architectural template below as a **rule set** and reviews the changes on the current branch against it. It reports deviations with a file path + line number; if things conform, it briefly confirms.

## 1. Top-level folder structure

```
lib/
├── main.dart
├── firebase_options.dart
├── core/                     # framework-agnostic infrastructure, knows no feature
│   ├── dio_manager/          # HTTP client (DioApiManager)
│   │   ├── dio_manager.dart
│   │   ├── api_response_model.dart
│   │   ├── api_error_model.dart
│   │   └── interceptor/
│   ├── result/               # DataResult<T> sealed result type
│   │   ├── result.dart
│   │   ├── data_result.dart
│   │   ├── succes_data_result.dart
│   │   ├── error_data_result.dart
│   │   ├── succes_result.dart
│   │   └── error_result.dart
│   ├── model/
│   │   └── cubit_data_model.dart   # Loading/Done sealed cubit state wrapper
│   ├── extension/
│   ├── logger/
│   ├── network_control/
│   ├── keys/
│   └── assets/
└── app/                      # application code
    ├── common/               # things shared across features
    │   ├── di/get_it.dart    # ServiceLocator
    │   ├── route/app_router.dart
    │   ├── config/app_config.dart
    │   ├── constant/         # AppColorConstant, AppFontConstant ...
    │   ├── enum/             # SvgEnum etc.
    │   ├── service/          # AppPrefsService, AuthService
    │   ├── function/
    │   └── widget/           # button, textfield, dialog, sheet ...
    └── feature/
        ├── data/             # single "data" layer for all features
        │   ├── model/<feature>/
        │   ├── datasource/
        │   │   ├── remote/<feature>/
        │   │   └── local/
        │   └── repository/<feature>/
        └── presentation/
            └── <feature>/
                ├── cubit/
                ├── view/
                ├── widget/
                ├── model/    # presentation-only models (e.g. Kisi)
                ├── enum/
                └── const/
```

**Rule:** `core/` never imports `app/`. `common/` never imports `feature/`. `feature/<x>` never imports another `feature/<y>` (if needed, move the shared type into `common/` or `data/model/`).

## 2. Layers and responsibilities

### 2.1 Data layer

Order: **Model → RemoteDatasource → Repository**. All three come together for each feature.

**Model:**
- `*RequestModel`: contains only `toMap()`, the payload going to the server.
- `*ResponseModel`: extends `Equatable`, contains `factory fromMap(Map<String, dynamic>)`. All fields nullable + safe parsing (`int.tryParse`, `DateTime.tryParse`).
- Location: `lib/app/feature/data/model/<feature>/`.

**RemoteDatasource:**
- `abstract class XxxRemoteDatasource` + `final class XxxRemoteDatasourceImpl implements XxxRemoteDatasource`.
- Return type is always `Future<ApiResponseModel<T>>`.
- Consumes `DioApiManager` directly and does JSON → Model conversion here via the `converter:` parameter. For list returns, use the `whereType<Map<String, dynamic>>().map(...).toList(growable: false)` pattern.
- Endpoint strings live here (`/api/v1/...`) and appear in no other layer.

**Repository:**
- `abstract class XxxRepository` + `class XxxRepositoryImpl implements XxxRepository`.
- Constructor: `XxxRepositoryImpl({required XxxRemoteDatasource remoteDatasource})`. Does not hold `DioApiManager` directly.
- Return type is always `Future<DataResult<T>>` (`SuccessDataResult` / `ErrorDataResult`).
- The `ApiResponseModel → DataResult` conversion happens here. Use a single mapping helper (e.g. `_mapSingle`).
- Every branch calls `AppLogger.instance.log/error`; message format: `"$runtimeType <op> SUCCESS"` or `"$runtimeType <op> <error> Status code: <code>"`.

### 2.2 Presentation layer

**Cubit:**
- Extends `flutter_bloc` `Cubit<XxxState>`.
- The repository is injected via the constructor (`required ContactRepository contactRepository`). The Cubit never sees a datasource or `DioApiManager`.
- The state class is `Equatable`, immutable, updated via `copyWith`.
- When an async job starts, set `isLoading: true` / `isSaving: true`; when it finishes, set `clearLoadError: true` or `loadError: ...`. Instead of per-feature loading flags, prefer `CubitDataModel<T>` (Loading/Done sealed) for async-wrapped data.
- If a `TextEditingController` is held in the Cubit, override `close()` and dispose it.

**View / Widget:**
- View files live under `view/`, small extracted pieces under `widget/`.
- The Cubit is provided inside the view via `BlocProvider(create: (_) => getIt<XxxCubit>())`.

### 2.4 Per-page (feature) internal folder structure — MANDATORY

Each page's own cubit + state, its own mixins, and its own views/widgets live under the **same feature folder**. A cubit/state is not shared with another feature.

```
presentation/home/
├── cubit/
│   ├── home_cubit.dart
│   └── home_state.dart
├── mixin/                   # optional — only if there is repeated logic
│   └── home_form_mixin.dart
├── view/
│   ├── home_view.dart
│   └── home_detail_view.dart
└── widget/
    ├── home_header.dart
    └── home_list_tile.dart
```

**Rules:**

- A `cubit/` folder is created for each page; inside it the **state file and cubit file are separate** (`home_cubit.dart`, `home_state.dart`).
- A cubit belongs to exactly one feature. `ProfileView` cannot import `HomeCubit`; if there's a shared need, move it into `common/service` or `data/repository`.
- `view/` contains only screen (route-target) widgets.
- `widget/` contains the extracted, small UI components used by the view.

### 2.5 Widget requirements — MANDATORY

#### Stateless by default — STRICT RULE

- **All widgets must be `StatelessWidget`.** The only way to hold state is Cubit + `BlocBuilder` / `BlocSelector`.
- Controllers like `TextEditingController`, `ScrollController`, `FocusNode`, `PageController` are held **inside the Cubit**. The Cubit overrides `close()` to dispose them. These controllers are passed to the widget via `context.read<XxxCubit>().nameController`.
- `StatefulWidget` is accepted **only in these exceptional cases**:
  - **Animations** — `AnimationController` + `TickerProviderStateMixin` already require `State`. (Moving it into the Cubit is not possible because `vsync` is bound to the widget.)
  - **External (3rd-party) package requirement** — if the package's API expects a `StatefulWidget` or a `State` reference.

Any `StatefulWidget` outside these two exceptions is reported with the `[STATELESS]` tag and asked to be moved into the cubit.

**Bad:**

```dart
class LoginView extends StatefulWidget {
  @override
  State<LoginView> createState() => _LoginViewState();
}

class _LoginViewState extends State<LoginView> {
  final emailController = TextEditingController(); // WRONG — should be in the cubit
  final passwordController = TextEditingController();

  @override
  void dispose() {
    emailController.dispose();
    passwordController.dispose();
    super.dispose();
  }
  // ...
}
```

**Good:**

```dart
class LoginCubit extends Cubit<LoginState> {
  LoginCubit({required this.authRepository}) : super(const LoginState());

  final AuthRepository authRepository;
  final emailController = TextEditingController();
  final passwordController = TextEditingController();

  @override
  Future<void> close() {
    emailController.dispose();
    passwordController.dispose();
    return super.close();
  }
}

class LoginView extends StatelessWidget {
  const LoginView({super.key});

  @override
  Widget build(BuildContext context) {
    final cubit = context.read<LoginCubit>();
    return Column(
      children: [
        TextField(controller: cubit.emailController),
        TextField(controller: cubit.passwordController),
      ],
    );
  }
}
```

#### Only view work in the view file

- A view file contains nothing beyond the **build tree** and the **BlocProvider / BlocListener setup**.
- **No business-logic functions/methods are defined** inside a view. API calls, computation, validation, and format conversion move into the cubit.
- Inside a view, only these kinds of helpers are allowed:
  - Side-effect triggers like `Navigator` / `SnackBar` inside a `BlocListener` callback
  - Small `Widget _buildHeader()`-style methods split out for build-tree readability — but if they have reuse potential, extract them into a separate `StatelessWidget` under `widget/`.
- If business logic like `if (state.x) { doSomething(); }` is seen inside a view, it's reported with the `[VIEW_LOGIC]` tag and suggested to move into the cubit.

#### Repeated operations → `mixin/`

- If, within the same feature, a behavior is **repeated across more than one view or widget** (a form-validation helper, a dialog-opening flow, a snackbar template, etc.), create a `mixin/` folder under that feature.
- The mixin class is written with a type constraint via `on State<T>` or `on Cubit<S>`; it is not written as a loosely-coupled global helper.
- A mixin is used **only within the same feature**. If it's needed in more than one feature, move it into `common/`.
- **Do not create a mixin** for a behavior used in a single place — `Simplicity First` (see [../../CLAUDE.md](../../CLAUDE.md)).

**Review tags:**

```
[FOLDER]        — feature folder structure (cubit/mixin/view/widget) missing or wrong
[STATELESS]     — unnecessary StatefulWidget usage
[VIEW_LOGIC]    — business logic in a view / should move into the cubit
[MIXIN]         — repeated code that should be extracted into a mixin / mixin in the wrong place
```

### 2.3 Core contracts

- **`ApiResponseModel<T>`** = `{ data, error, isSuccess }` — only the datasource layer sees it.
- **`DataResult<T>`** = `SuccessDataResult<T>` / `ErrorDataResult<T>` — the single result type that crosses the repository boundary outward. Cubits look at `result.success`, `result.data`, `result.message`; they never see `ApiResponseModel`.
- **`CubitDataModel<T>`** = `CubitDataLoadingModel<T>` / `CubitDataDoneModel<T>` — the async-data wrapper in cubit states.
- **`DioApiManager`** is the single HTTP entry point. `BaseOptions(baseUrl: Config.apiBaseUrl)`, three interceptors: `AuthInterceptor`, `LogInterceptor`, `ErrorLoggingInterceptor`. Error mapping in `_handleDioError` recognizes the NestJS `{ message }` body.

## 3. Dependency Injection

`lib/app/common/di/get_it.dart` — `ServiceLocator.setup()` is the single entry point, with **ordered phases:**

```
_setupRouter()      → AppRouter
_setupNetwork()     → (DioApiManager singleton, if needed)
_setupService()     → AuthService, AppPrefsService ...
_setupDataSource()  → XxxRemoteDatasource (with Impl)
_setupRepository()  → XxxRepository (datasource injected)
_setupCubit()       → XxxCubit (repository / service injected)
```

**Rules:**
- Datasources and repositories are usually `registerLazySingleton`.
- Form/flow cubits (those that want a fresh state on every open: Onboarding, Auth, YeniKisi) are `registerFactory`.
- Cubits that must keep list/state alive (MainCubit, KisilerCubit) are `registerLazySingleton`.
- When a new feature is added, the **datasource → repository → cubit** registrations must all be added across the three phases; adding two and forgetting one is the most common mistake.

## 4. Routing

- `auto_route`, `@AutoRouterConfig(replaceInRouteName: 'View|Page,Route')`.
- View file name `xxx_view.dart`, class name `XxxView`, generated route name `XxxRoute`.
- New route: add the `@RoutePage()` annotation to the view → add `AutoRoute(page: XxxRoute.page)` into `AppRouter.routes` → `dart run build_runner build --delete-conflicting-outputs`.
- The default transition has no effect (returns `child`); override per-route if needed.

## 5. Asset and design tokens

- SVGs via `assets/svg/` + `SvgEnum` (`SvgEnum.foo.svgWidget(...)`). Don't write `SvgPicture.asset(...)` directly.
- Check that the path is added under `pubspec.yaml` → `flutter.assets`.
- Colors come from `AppColorConstant.*`. Hardcoded `Color(0x...)` is flagged in review. `neturalWhite` (typo) is deliberate; don't rename it.
- TextStyle always through `Theme.of(context).textTheme.<entry>?.copyWith(...)` — not bare `TextStyle(...)`.
- `AppButton` disabled state via Flutter's `onPressed: null` idiom + `disabledBackgroundColor/disabledForegroundColor`; don't add a separate `bool enabled` parameter.

## 6. Import style

- Always the `package:<app_name>/...` form inside `lib/`. No relative imports.

## 7. Review checklist (to follow when the skill runs)

Verify the following in order; report each deviation with a `path:line` reference.

1. **Folder placement:** Are new files in the right layer? Datasource in `data/datasource/remote/<feature>/`, repository in `data/repository/<feature>/`, model in `data/model/<feature>/`, cubit/view in `presentation/<feature>/cubit|view|widget/`.
2. **Naming/contract:**
   - `XxxRemoteDatasource` + `XxxRemoteDatasourceImpl` (Impl is a `final class`, interface is `abstract`).
   - `XxxRepository` + `XxxRepositoryImpl`.
   - Model files `xxx_request_model.dart` / `xxx_response_model.dart`.
   - View `xxx_view.dart`, cubit `xxx_cubit.dart`, state `xxx_state.dart`.
3. **Return types:**
   - Datasource → `Future<ApiResponseModel<T>>`. The repository must not leak `ApiResponseModel` anywhere.
   - Repository → `Future<DataResult<T>>`. The cubit must look at `result.success`/`result.data`/`result.message`, not `isSuccess` or `error.message`.
4. **DioApiManager usage:** Is it called only from datasources? Is `Dio` imported from within a cubit or view (an error if so)?
5. **DI registration order:** Are all three of `datasource → repository → cubit` added into `get_it.dart`? Does the `registerFactory` vs `registerLazySingleton` choice make sense (form cubit → factory, living cubit → singleton)?
6. **State management:**
   - Is the state `Equatable`, with `copyWith`?
   - Are loading/error fields reset correctly (`clearXError: true`)?
   - Is `CubitDataModel<T>` appropriate instead of a per-feature loading flag?
   - If a `TextEditingController` is held in the cubit, is it disposed in the `close()` override?
7. **Logger:** Do error branches follow the `AppLogger.instance.error("$runtimeType <op> ...")` format?
8. **Routing:** Is the view `@RoutePage`-annotated, added to the `AppRouter.routes` list, and is `app_router.gr.dart` regenerated?
9. **Design tokens / assets:**
   - Are SVGs via `SvgEnum`?
   - Was the `pubspec.yaml` asset entry added?
   - Any hardcoded color or bare `TextStyle()`?
10. **Import style:** No relative paths in new files?
11. **Cross-layer violations:** `core/` → `app/` import, `feature/<x>` → `feature/<y>` import, presentation → datasource direct access — any of these is a red flag.

## 8. Reference template when adding a new feature (example: `contact`)

```
lib/app/feature/
├── data/
│   ├── model/contact/
│   │   ├── contact_request_model.dart      # toMap()
│   │   └── contact_response_model.dart     # Equatable + fromMap()
│   ├── datasource/remote/contact/
│   │   └── contact_remote_datasource.dart  # abstract + Impl, uses DioApiManager
│   └── repository/contact/
│       └── contact_repository.dart         # abstract + Impl, ApiResponseModel → DataResult
└── presentation/kisiler/
    ├── cubit/
    │   ├── kisiler_cubit.dart              # repository injected, copyWith state
    │   └── kisiler_state.dart              # Equatable
    ├── view/
    ├── widget/
  
```

On the DI side:

```dart
getIt.registerLazySingleton<ContactRemoteDatasource>(
  () => ContactRemoteDatasourceImpl(),
);
getIt.registerLazySingleton<ContactRepository>(
  () => ContactRepositoryImpl(remoteDatasource: getIt<ContactRemoteDatasource>()),
);
getIt.registerLazySingleton<KisilerCubit>(
  () => KisilerCubit(contactRepository: getIt<ContactRepository>()),
);
```

## 9. Skill execution protocol

When the skill is invoked:

1. Get the changed files via `git status` + `git diff main...HEAD`.
2. Evaluate each changed file against the checklist in section 7.
3. Output format:
   - **Conforms:** a short confirmation sentence.
   - **Deviation:** `path:line — description of the violation — suggested fix.`
4. Group the results by section heading (Layer violation / DI / State / Routing / Design tokens / Import). Don't print empty sections.
5. End with a one-sentence overall summary; explicitly flag "I'm stuck on this, let the user decide" ambiguities and do not make code edits on your own.
