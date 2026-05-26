---
name: flutter-performance-reviewer
description: Flutter uygulamalarındaki performans darboğazlarını (rebuild, jank, memory leak) tespit eder ve optimizasyon önerir.
---

# Flutter Performance Reviewer

> Bu skill çalışırken [../../CLAUDE.md](../../CLAUDE.md) içindeki davranış kurallarına (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) uyar.

## Amaç

Flutter kodunu performans açısından inceler:

- Gereksiz `setState` ve rebuild zincirleri
- `const` constructor eksiklikleri
- `ListView.builder` yerine `ListView` kullanımı gibi liste performans hataları
- Image cache, asset boyutu ve `precacheImage` kullanımı
- `FutureBuilder` / `StreamBuilder` kötüye kullanımları
- State management (Provider, Riverpod, Bloc) ile gereksiz dinleme
- `dispose` edilmeyen `Controller`, `StreamSubscription`, `AnimationController`
- Build metodunda ağır işlem (JSON parse, sıralama, hesaplama)

## Ne zaman kullan

- Uygulama jank yaşadığında
- DevTools profiler çıktısı incelendiğinde
- Liste/scroll ekranları yazılırken

## Çıktı formatı

```
[KRİTİKLİK: Yüksek/Orta/Düşük]
[DOSYA:SATIR]
Sorun: ...
Etki: ... (FPS düşüşü, memory leak, vb.)
Çözüm: ...
```

## Kontrol listesi

- [ ] Tüm immutable widget'lar `const` mi?
- [ ] Liste widget'ları `builder` kullanıyor mu?
- [ ] Controller'lar `dispose` ediliyor mu?
- [ ] Build metodu saf (pure) mı?
- [ ] Resimler doğru boyutta mı yükleniyor?

## Referans lint kuralları

Bu skill, `examples/analysis_options.yaml` içindeki performans ve bellek kurallarını temel alır:

**Performans:**
- `avoid_print`, `avoid_slow_async_io`
- `prefer_const_constructors`, `prefer_const_constructors_in_immutables`
- `prefer_const_declarations`, `prefer_const_literals_to_create_immutables`
- `unnecessary_const`, `unnecessary_lambdas`
- `avoid_unnecessary_containers`, `sized_box_for_whitespace`
- `prefer_final_fields`, `prefer_final_in_for_each`, `prefer_final_locals`

**Bellek yönetimi:**
- `close_sinks` — StreamController kapatma
- `cancel_subscriptions` — Subscription iptal
- `avoid_returning_null_for_future`

**Flutter spesifik:**
- `use_build_context_synchronously`
- `no_logic_in_create_state`
- `avoid_web_libraries_in_flutter`

Tam config: [../../examples/analysis_options.yaml](../../examples/analysis_options.yaml)

## State management (Cubit + State) kuralları

Bu skill `flutter_bloc` `Cubit` kullanımını şu kurallara göre denetler. Riverpod / Provider kullanan projelerde de aynı prensipler (immutability, eşitlik, selective rebuild) geçerlidir.

### 1. State sınıfı `Equatable` olmak zorundadır

Cubit `emit(newState)` çağırdığında `Bloc`/`Cubit` yeni state'i eskisiyle `==` ile karşılaştırır. `Equatable` yoksa her `emit` referans eşitsizliğinden dolayı **yeni** sayılır ve `BlocBuilder` gereksiz rebuild yapar.

**Kötü:**

```dart
class CounterState {
  final int count;
  final bool isLoading;
  CounterState({required this.count, required this.isLoading});
}
```

**İyi:**

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

`props` listesine state'in **tüm** alanları eklenmelidir. Unutulan alan = sessiz "UI güncellenmiyor" bug'ı.

### 2. State immutable + `copyWith` ile güncellenmeli

State alanları `final`, sınıf `const` constructor'a sahip olmalı. Güncelleme yalnızca `copyWith` üzerinden:

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

**Kurallar:**

- Cubit içinde `state.items.add(x)` gibi **mutasyon yasak** — yeni liste oluştur: `[...state.items, x]`.
- Bir alanı `null`'a çekmek için `copyWith(errorMessage: null)` çalışmaz (`?? this.errorMessage` yutar). Bunun için `clearError: true` gibi açık bayrak kullanılmalı.
- Default değerler `const []`, `const {}` olmalı; her constructor çağrısında yeni boş liste yaratmak gereksiz allocation üretir.

### 3. `emit` öncesi state karşılaştırması

`Cubit` zaten aynı state'i tekrar emit ederse dinleyicilere yayın yapmaz; bu da `Equatable` doğru kurulduğunda **bedavaya gelen performans** demektir. Manuel `if (state == newState) return;` yazmaya gerek yok — ama `Equatable` yoksa bu mekanizma çalışmaz.

### 4. `BlocBuilder` yerine seçici dinleme

Tüm state'i dinleyen `BlocBuilder` yerine, yalnızca ilgilenilen alana göre rebuild yapılmalı:

- `BlocSelector<C, S, T>` — tek bir alan için.
- `BlocBuilder` + `buildWhen: (prev, curr) => prev.x != curr.x`.
- Sadece side-effect için `BlocListener` (snackbar, navigation); rebuild gerekmez.

### 5. Loading / Done sealed wrapper (opsiyonel ama önerilen)

Per-feature `bool isLoading` bayrakları yerine async veri için sealed bir sarmalayıcı tercih edilebilir (örn. `CubitDataModel<T>` → `Loading<T>` / `Done<T>`). Bu sayede UI tarafında `state.data.when(loading: ..., done: ...)` ile tek noktadan dallanma yapılır ve "loading true kaldı" hatası imkansızlaşır.

### 6. Cubit yaşam döngüsü

- Cubit içinde tutulan `TextEditingController`, `ScrollController`, `StreamSubscription` mutlaka `close()` override edilerek dispose edilmeli.
- `lint: close_sinks` ve `cancel_subscriptions` bunu yakalar ama review sırasında manuel de kontrol edilir.
- `BlocProvider` `create:` içinde cubit oluşturulmalı; `value:` ile geçirilen cubit'in dispose sorumluluğu sahibine aittir.

### Review etiketleri

State management ile ilgili bulgular şu etiketle raporlanır:

```
[STATE] - [Equatable eksik | copyWith yok | Mutasyon | Selective rebuild eksik | Dispose eksik]
```
