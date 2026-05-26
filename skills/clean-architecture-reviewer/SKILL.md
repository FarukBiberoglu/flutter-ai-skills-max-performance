---
name: clean-arc-review
description: Flutter projelerinde kullanılan Clean Architecture katmanlarını (core / app, data / presentation, datasource → repository → cubit → view akışı) referans alarak kod review yapar. Yeni eklenen feature'ın klasör yapısına, DI kayıt sırasına, ApiResponseModel → DataResult → CubitDataModel dönüşüm zincirine ve isimlendirme kurallarına uyup uymadığını kontrol eder.
---

# Clean Architecture Review

> Bu skill çalışırken [../../CLAUDE.md](../../CLAUDE.md) içindeki davranış kurallarına (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) uyar.

Bu skill, aşağıdaki mimari şablonu **kural kümesi** olarak alır ve mevcut branch'teki değişiklikleri bu kurallara göre review eder. Sapma varsa dosya yolu + satır numarası ile birlikte raporlar; uyum varsa kısaca onaylar.

## 1. Top-level klasör yapısı

```
lib/
├── main.dart
├── firebase_options.dart
├── core/                     # framework-agnostic altyapı, feature bilmez
│   ├── dio_manager/          # HTTP istemcisi (DioApiManager)
│   │   ├── dio_manager.dart
│   │   ├── api_response_model.dart
│   │   ├── api_error_model.dart
│   │   └── interceptor/
│   ├── result/               # DataResult<T> sealed sonuç tipi
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
└── app/                      # uygulama kodu
    ├── common/               # feature'lar arası ortak şeyler
    │   ├── di/get_it.dart    # ServiceLocator
    │   ├── route/app_router.dart
    │   ├── config/app_config.dart
    │   ├── constant/         # AppColorConstant, AppFontConstant ...
    │   ├── enum/             # SvgEnum vb.
    │   ├── service/          # AppPrefsService, AuthService
    │   ├── function/
    │   └── widget/           # button, textfield, dialog, sheet ...
    └── feature/
        ├── data/             # tüm feature'lar için tek "data" katmanı
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
                ├── model/    # presentation-only modeller (örn. Kisi)
                ├── enum/
                └── const/
```

**Kural:** `core/` asla `app/` import etmez. `common/` asla `feature/` import etmez. `feature/<x>` başka bir `feature/<y>` import etmez (gerek varsa ortak tip `common/` veya `data/model/`'e taşınır).

## 2. Katmanlar ve sorumluluklar

### 2.1 Data katmanı

Sıra: **Model → RemoteDatasource → Repository**. Her feature için üçü birlikte gelir.

**Model:**
- `*RequestModel`: yalnızca `toMap()` içerir, sunucuya gidecek payload.
- `*ResponseModel`: `Equatable` extend eder, `factory fromMap(Map<String, dynamic>)` içerir. Tüm alanlar nullable + güvenli parse (`int.tryParse`, `DateTime.tryParse`).
- Konum: `lib/app/feature/data/model/<feature>/`.

**RemoteDatasource:**
- `abstract class XxxRemoteDatasource` + `final class XxxRemoteDatasourceImpl implements XxxRemoteDatasource`.
- Dönüş tipi her zaman `Future<ApiResponseModel<T>>`.
- `DioApiManager`'ı doğrudan tüketir, `converter:` parametresi ile JSON → Model dönüşümünü burada yapar. List dönüşlerinde `whereType<Map<String, dynamic>>().map(...).toList(growable: false)` deseni kullanılır.
- Endpoint string'leri burada (`/api/v1/...`), başka katmanda görünmez.

**Repository:**
- `abstract class XxxRepository` + `class XxxRepositoryImpl implements XxxRepository`.
- Constructor: `XxxRepositoryImpl({required XxxRemoteDatasource remoteDatasource})`. Doğrudan `DioApiManager` tutmaz.
- Dönüş tipi her zaman `Future<DataResult<T>>` (`SuccessDataResult` / `ErrorDataResult`).
- `ApiResponseModel → DataResult` çevrimi burada olur. Tek tip mapping helper'ı (örn. `_mapSingle`) kullanılır.
- Her dalda `AppLogger.instance.log/error` çağrılır; mesaj formatı: `"$runtimeType <op> SUCCESS"` veya `"$runtimeType <op> <hata> Status code: <kod>"`.

### 2.2 Presentation katmanı

**Cubit:**
- `flutter_bloc` `Cubit<XxxState>` extend eder.
- Repository constructor ile inject edilir (`required ContactRepository contactRepository`). Cubit datasource veya `DioApiManager` görmez.
- State sınıfı `Equatable`'dır, immutable, `copyWith` ile güncellenir.
- Async iş başlarken `isLoading: true` / `isSaving: true`, biterken `clearLoadError: true` veya `loadError: ...` set edilir. Per-feature loading bayrağı yerine async/wrap edilen veri için `CubitDataModel<T>` (Loading/Done sealed) tercih edilir.
- `TextEditingController` cubit içinde tutuluyorsa `close()` override edip dispose eder.

**View / Widget:**
- View dosyaları `view/` altında, parçalanmış küçük parçalar `widget/` altında.
- Cubit, view içinde `BlocProvider(create: (_) => getIt<XxxCubit>())` ile sağlanır.

### 2.4 Sayfa (feature) iç klasör yapısı — ZORUNLU

Her sayfanın kendi cubit + state'i, kendi mixin'leri ve kendi view/widget'ları **aynı feature klasörü** altında olur. Cubit/state başka feature'la paylaşılmaz.

```
presentation/home/
├── cubit/
│   ├── home_cubit.dart
│   └── home_state.dart
├── mixin/                   # opsiyonel — sadece tekrar eden işlemler varsa
│   └── home_form_mixin.dart
├── view/
│   ├── home_view.dart
│   └── home_detail_view.dart
└── widget/
    ├── home_header.dart
    └── home_list_tile.dart
```

**Kurallar:**

- Her sayfa için `cubit/` klasörü açılır; içinde **state dosyası ve cubit dosyası ayrı** durur (`home_cubit.dart`, `home_state.dart`).
- Bir cubit yalnızca bir feature'a aittir. `HomeCubit`'i `ProfileView` import edemez; ortak ihtiyaç varsa `common/service` veya `data/repository`'e taşınır.
- `view/` yalnızca ekran (route hedefi) widget'larını içerir.
- `widget/` view tarafından kullanılan, parçalanmış küçük UI bileşenlerini içerir.

### 2.5 Widget zorunlulukları — ZORUNLU

#### Stateless varsayılan — KATI KURAL

- **Tüm widget'lar `StatelessWidget` olmak zorundadır.** State tutmak için tek yol Cubit + `BlocBuilder` / `BlocSelector`'dır.
- `TextEditingController`, `ScrollController`, `FocusNode`, `PageController` gibi controller'lar **Cubit içinde** tutulur. Cubit `close()` override edip bunları dispose eder. Bu controller'lar widget'a `context.read<XxxCubit>().nameController` üzerinden geçirilir.
- `StatefulWidget` **yalnızca şu istisnai durumlarda** kabul edilir:
  - **Animasyonlar** — `AnimationController` + `TickerProviderStateMixin` zaten `State` gerektirir. (Cubit'e taşımak mümkün değil çünkü `vsync` widget'a bağlıdır.)
  - **Harici (3rd-party) paket zorunluluğu** — paketin API'si bir `StatefulWidget` veya `State` referansı bekliyorsa.

Bu iki istisna dışındaki her `StatefulWidget` `[STATELESS]` etiketiyle raporlanır ve cubit'e taşınması istenir.

**Kötü:**

```dart
class LoginView extends StatefulWidget {
  @override
  State<LoginView> createState() => _LoginViewState();
}

class _LoginViewState extends State<LoginView> {
  final emailController = TextEditingController(); // YANLIŞ — cubit'te olmalı
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

**İyi:**

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

#### View dosyasında yalnızca view işi

- View dosyası **build ağacı** ve **BlocProvider / BlocListener kurulumu** dışında iş içermez.
- View içinde **iş mantığı yapan fonksiyon / metot tanımlanmaz**. API çağrısı, hesaplama, validasyon, format dönüşümü cubit'e taşınır.
- View içinde yalnızca şu tip yardımcılar olabilir:
  - `BlocListener` callback'i içinde `Navigator` / `SnackBar` gibi side-effect tetikleyiciler
  - Build ağacının okunabilirliği için ayrılmış küçük `Widget _buildHeader()` benzeri metotlar — ancak bunların tekrar potansiyeli varsa `widget/` altına ayrı `StatelessWidget` olarak çıkarılır.
- View içinde `if (state.x) { doSomething(); }` gibi iş mantığı görülürse `[VIEW_LOGIC]` etiketiyle raporlanır ve cubit'e taşıma önerilir.

#### Tekrar eden işlemler → `mixin/`

- Aynı feature içinde **birden fazla view veya widget'ta tekrar eden** davranış (form validasyon helper'ı, dialog açma akışı, snackbar şablonu vs.) varsa o feature'ın altında `mixin/` klasörü açılır.
- Mixin sınıfı `on State<T>` veya `on Cubit<S>` ile tip kısıtlanarak yazılır; gevşek bağlı global helper olarak yazılmaz.
- Mixin yalnızca **aynı feature** içinde kullanılır. Birden fazla feature'da gerekiyorsa `common/` altına taşınır.
- Tek bir yerde kullanılan davranış için mixin **açılmaz** — `Simplicity First` (bkz. [../../CLAUDE.md](../../CLAUDE.md)).

**Review etiketleri:**

```
[FOLDER]        — feature klasör yapısı (cubit/mixin/view/widget) eksik veya yanlış
[STATELESS]     — gereksiz StatefulWidget kullanımı
[VIEW_LOGIC]    — view içinde iş mantığı / cubit'e taşınmalı
[MIXIN]         — tekrar eden kod var, mixin'e çıkarılmalı / yanlış konumda mixin
```

### 2.3 Core sözleşmeleri

- **`ApiResponseModel<T>`** = `{ data, error, isSuccess }` — sadece datasource katmanı görür.
- **`DataResult<T>`** = `SuccessDataResult<T>` / `ErrorDataResult<T>` — repository sınırından dışarı çıkan tek sonuç tipi. Cubit'ler `result.success`, `result.data`, `result.message`'a bakar; `ApiResponseModel`'i görmez.
- **`CubitDataModel<T>`** = `CubitDataLoadingModel<T>` / `CubitDataDoneModel<T>` — cubit state'lerinde async veri sarmalayıcısı.
- **`DioApiManager`** tek HTTP giriş noktası. `BaseOptions(baseUrl: Config.apiBaseUrl)`, üç interceptor: `AuthInterceptor`, `LogInterceptor`, `ErrorLoggingInterceptor`. Hata haritalama `_handleDioError` içinde NestJS `{ message }` body'sini tanır.

## 3. Dependency Injection

`lib/app/common/di/get_it.dart` — `ServiceLocator.setup()` tek giriş, **sıralı fazlar:**

```
_setupRouter()      → AppRouter
_setupNetwork()     → (gerekirse DioApiManager singleton)
_setupService()     → AuthService, AppPrefsService ...
_setupDataSource()  → XxxRemoteDatasource (Impl ile)
_setupRepository()  → XxxRepository (datasource inject)
_setupCubit()       → XxxCubit (repository / service inject)
```

**Kurallar:**
- Datasource ve Repository genelde `registerLazySingleton`.
- Form/akış cubit'leri (her açılışta sıfır state isteyenler: Onboarding, Auth, YeniKisi) `registerFactory`.
- Liste/state'i yaşatması gereken cubit'ler (MainCubit, KisilerCubit) `registerLazySingleton`.
- Yeni feature eklendiğinde **datasource → repository → cubit** kayıtları üç fazda da eklenmiş olmalı; ikisini eklerken birini unutmak en sık görülen hata.

## 4. Routing

- `auto_route`, `@AutoRouterConfig(replaceInRouteName: 'View|Page,Route')`.
- View dosya adı `xxx_view.dart`, sınıf adı `XxxView`, üretilen route adı `XxxRoute`.
- Yeni route: view'a `@RoutePage()` annotasyonu → `AppRouter.routes` içine `AutoRoute(page: XxxRoute.page)` → `dart run build_runner build --delete-conflicting-outputs`.
- Default geçiş efektsiz (`child` döner); ihtiyaç olursa per-route override.

## 5. Asset ve tasarım token'ları

- SVG'ler `assets/svg/` + `SvgEnum` üzerinden (`SvgEnum.foo.svgWidget(...)`). Doğrudan `SvgPicture.asset(...)` yazılmaz.
- `pubspec.yaml` → `flutter.assets` altına path eklenmiş mi kontrol edilir.
- Renkler `AppColorConstant.*`'tan gelir. Hardcoded `Color(0x...)` review'da işaretlenir. `neturalWhite` (typo) bilinçli; rename yapılmaz.
- TextStyle her zaman `Theme.of(context).textTheme.<entry>?.copyWith(...)` üzerinden — bare `TextStyle(...)` değil.
- `AppButton` disabled state'i Flutter'ın `onPressed: null` idiom'u + `disabledBackgroundColor/disabledForegroundColor` ile; ayrı `bool enabled` parametresi eklenmez.

## 6. Import stili

- `lib/` içinde her zaman `package:<app_name>/...` formu. Göreceli import yok.

## 7. Review checklist (skill çalıştığında izlenecek liste)

Aşağıdakileri sırayla doğrula; her sapmayı `path:line` referansıyla raporla.

1. **Klasör yerleşimi:** Yeni dosyalar doğru katmanda mı? Datasource `data/datasource/remote/<feature>/`, repository `data/repository/<feature>/`, model `data/model/<feature>/`, cubit/view `presentation/<feature>/cubit|view|widget/`.
2. **İsim/sözleşme:**
   - `XxxRemoteDatasource` + `XxxRemoteDatasourceImpl` (Impl `final class`, interface `abstract`).
   - `XxxRepository` + `XxxRepositoryImpl`.
   - Model dosyaları `xxx_request_model.dart` / `xxx_response_model.dart`.
   - View `xxx_view.dart`, cubit `xxx_cubit.dart`, state `xxx_state.dart`.
3. **Dönüş tipleri:**
   - Datasource → `Future<ApiResponseModel<T>>`. Repository hiçbir yerden `ApiResponseModel` sızdırmamalı.
   - Repository → `Future<DataResult<T>>`. Cubit `result.success`/`result.data`/`result.message`'a bakmalı, `isSuccess` veya `error.message`'a değil.
4. **DioApiManager kullanımı:** Sadece datasource'lardan çağrılıyor mu? Cubit veya view içinden `Dio` import edilmiş mi (varsa hata).
5. **DI kayıt sırası:** `get_it.dart` içine `datasource → repository → cubit` üçü de eklenmiş mi? `registerFactory` vs `registerLazySingleton` seçimi mantıklı mı (form cubit → factory, yaşayan cubit → singleton)?
6. **State yönetimi:**
   - State `Equatable`, `copyWith`'li mi?
   - Loading/error alanları doğru sıfırlanıyor mu (`clearXError: true`)?
   - Per-feature loading bayrağı yerine `CubitDataModel<T>` uygun mu?
   - `TextEditingController` cubit'te tutuluyorsa `close()` override'ında dispose ediliyor mu?
7. **Logger:** Hata dallarında `AppLogger.instance.error("$runtimeType <op> ...")` formatına uyuluyor mu?
8. **Routing:** View `@RoutePage` annotasyonlu mu, `AppRouter.routes` listesine eklenmiş mi, `app_router.gr.dart` regenere edilmiş mi?
9. **Tasarım token / asset:**
   - SVG `SvgEnum` üzerinden mi?
   - `pubspec.yaml` asset entry eklendi mi?
   - Hardcoded renk veya bare `TextStyle()` var mı?
10. **Import stili:** Yeni dosyalarda relative path yok mu?
11. **Cross-layer ihlalleri:** `core/` → `app/` import, `feature/<x>` → `feature/<y>` import, presentation → datasource doğrudan erişim — varsa kırmızı bayrak.

## 8. Yeni feature eklerken referans şablon (örnek: `contact`)

```
lib/app/feature/
├── data/
│   ├── model/contact/
│   │   ├── contact_request_model.dart      # toMap()
│   │   └── contact_response_model.dart     # Equatable + fromMap()
│   ├── datasource/remote/contact/
│   │   └── contact_remote_datasource.dart  # abstract + Impl, DioApiManager kullanır
│   └── repository/contact/
│       └── contact_repository.dart         # abstract + Impl, ApiResponseModel → DataResult
└── presentation/kisiler/
    ├── cubit/
    │   ├── kisiler_cubit.dart              # repository inject, copyWith state
    │   └── kisiler_state.dart              # Equatable
    ├── view/
    ├── widget/
  
```

DI tarafında:

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

## 9. Skill çalıştırma protokolü

Skill çağrıldığında:

1. `git status` + `git diff main...HEAD` ile değişen dosyaları al.
2. Her değişen dosyayı bölüm 7'deki checklist üzerinden değerlendir.
3. Çıktı formatı:
   - **Uyumlu:** kısa onay cümlesi.
   - **Sapma:** `path:line — ihlal açıklaması — önerilen düzeltme.`
4. Sonuçları bölüm başlıklarına göre grupla (Katman ihlali / DI / State / Routing / Tasarım token / Import). Boş bölümleri yazma.
5. Tek cümlelik genel özet ile bitir; "şuna takıldım, kullanıcı karar versin" tarzı belirsizlikleri açıkça işaretle, kendi başına code edit yapma.
