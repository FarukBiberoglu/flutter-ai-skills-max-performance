---
name: firebase-flutter-helper
description: Reviews Firebase integration in Flutter projects (Auth phone/OTP + email link, Cloud Messaging/FCM) against the architecture. Checks that the Firebase SDK stays in the service layer only, the FirebaseAuthException → domain exception mapping, the ID token → Dio interceptor flow, the DI registration order, and Firebase-backed cubit/state/Equatable usage.
---

# Firebase Flutter Helper

> While running, this skill follows the behavioral rules in [../../CLAUDE.md](../../CLAUDE.md) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). For architectural layer rules see [../clean-architecture-reviewer/SKILL.md](../clean-architecture-reviewer/SKILL.md); for Cubit/State/`Equatable` performance rules see [../flutter-performance-reviewer/SKILL.md](../flutter-performance-reviewer/SKILL.md). This skill does not repeat them; it adds Firebase-specific rules.

## 0. Scope — where is Firebase used in this project?

In this architecture Firebase is used in **two** places:

- **Auth** (`firebase_auth`) — phone/OTP verification, email/password linking (`linkWithCredential`), email login, password reset, account deletion, ID token generation.
- **Messaging** (`firebase_messaging`) — FCM push token and notification permissions.

**There is NO Firestore / Realtime Database.** App data goes to the NestJS backend via `DioApiManager`. Firebase provides **identity** and **notifications** only. If you see a `cloud_firestore` import in a review, that's unexpected for this architecture; report it with the `[FIREBASE_SCOPE]` tag and ask for the rationale.

## 1. Layer rule — Firebase SDK only in the `service` layer — MANDATORY

Firebase SDK classes (`FirebaseAuth`, `User`, `UserCredential`, `FirebaseAuthException`, `PhoneAuthProvider`, `FirebaseMessaging`, `Firebase`) may be imported **only** in services under `lib/app/common/service/`.

**The only permitted exception:** the `Firebase.initializeApp(...)` call in `main.dart` and `firebase_options.dart`.

The following layers **do not** see the Firebase SDK:

- `presentation/*/cubit` → calls only `AuthService` / `NotificationService`.
- `presentation/*/view` & `widget` → don't know about Firebase at all.
- `data/datasource` & `data/repository` → this is the backend (Dio) side; the Firebase token comes from the interceptor, the datasource doesn't fetch the token manually.

**Why:** if Firebase types (especially `FirebaseAuthException`) leak into the UI, the app becomes coupled to Firebase's raw error codes (`invalid-verification-code`, `too-many-requests`) and untestable. The service closes the boundary by converting these types into a **domain exception** (see section 3).

If a `package:firebase_*` import is seen in any cubit/view/repository file, report it with the `[FIREBASE_LAYER]` tag.

## 2. Service contract — `AuthService`

The service that wraps Firebase follows these rules:

- `final class AuthService` — `FirebaseAuth` is **injected via the constructor**, so a fake can be supplied in tests:

  ```dart
  final class AuthService {
    AuthService({FirebaseAuth? firebaseAuth})
        : _firebaseAuth = firebaseAuth ?? FirebaseAuth.instance;

    final FirebaseAuth _firebaseAuth;
  }
  ```

  Calling `FirebaseAuth.instance` directly inside a method (`FirebaseAuth.instance.signOut()`) is **forbidden** — use the injected field. Otherwise the service can't be unit-tested. `[FIREBASE_TESTABILITY]`.

- The service exposes **pure async methods** (`sendOtp`, `verifyOtp`, `login`, `linkEmail`, `signOut`, `deleteAccount`, `getToken`, `refreshIdToken`). Each method either returns successfully or throws an `AuthServiceException` — there's no third state.
- Simple getters for UI state: `hasAuthenticatedUser`, `currentUid`, `currentEmail`. These are read via `_firebaseAuth.currentUser` and do not leak the `User` object outward.
- Each method logs entry/success/error via `AppLogger` (`[AuthService][<op>] ...`). Be careful when logging PII like phone/email; a raw token is never logged outside `debug`.

## 3. Exception mapping — raw Firebase codes don't leak to the UI — MANDATORY

Every Firebase call in the service is wrapped with `on FirebaseAuthException catch` + a generic `catch` and converted into a **domain exception**:

```dart
try {
  final credential = await _firebaseAuth.signInWithCredential(cred);
  return credential;
} on FirebaseAuthException catch (e) {
  _logger.error('[AuthService][verifyOtp] firebase error code=${e.code}', error: e);
  throw _mapFirebaseAuthException(e);   // → AuthServiceException
} catch (_) {
  _logger.error('[AuthService][verifyOtp] unknown error');
  throw const AuthServiceException(code: 'unknown', message: 'Unexpected error while verifying OTP.');
}
```

**Rules:**

- `_mapFirebaseAuthException` maps every known `e.code` to a user-presentable, translated message; the `default` branch falls back to `e.message` for unknown codes.
- The Cubit catches **only `AuthServiceException`** (`on AuthServiceException catch (e)`), never `FirebaseAuthException`. If a `FirebaseAuthException` is caught in a cubit, that's `[FIREBASE_EXCEPTION]` — move it into the service.
- For **callback-based** APIs like `verifyPhoneNumber`, the error callback (`verificationFailed`) and success callback (`codeSent`) are collapsed into a single `Future` via a `Completer`; use `completer.completeError(_mapFirebaseAuthException(e))`. An `if (!completer.isCompleted)` guard between callbacks is mandatory (double-completion is possible in Firebase).
- Operations that require recent login (`deleteAccount`, `changeEmail`) handle the `requires-recent-login` code separately and return a "please sign in again" message to the user.

## 4. ID token → Dio interceptor flow

The backend verifies every request with a Firebase ID token. Flow:

1. `AuthService.getToken()` → `currentUser.getIdToken()` (force-refresh with `getIdToken(true)` if needed).
2. The token is added to the request by **`AuthInterceptor`** — the cubit or datasource does not put the token in the header manually.
3. After an email/claim change (`linkEmail`, `changeEmail`), the token is **force-refreshed** with `getIdToken(true)` / `refreshIdToken()`; otherwise the backend sees stale claims (e.g. a missing `email`) and the first request fails.

**Review check:** if `getIdToken` / manual `Authorization` header setting is seen in a cubit or view, that's `[FIREBASE_TOKEN]` — token management belongs to the service + interceptor.

## 5. Messaging (FCM) → `NotificationService`

- The FCM token, permission request, and `onMessage` listeners are gathered inside `NotificationService`; `firebase_messaging` is not imported elsewhere.
- The FCM token is sent to the backend **without blocking the login flow**: `unawaited(_notificationService.registerToken())`. Even if token submission throws, the login flow must continue; therefore `registerToken` swallows/logs its own error internally and does not throw outward.
- **DI ordering trap:** `NotificationService` depends on `NotificationsCubit` but is registered in the `_setupService()` phase, while `NotificationsCubit` is registered in the `_setupCubit()` phase. Because `registerLazySingleton` is used, by the time the service is first **resolved**, all phases are done and the dependency is ready. If this dependency is registered with `registerSingleton` (eager), `NotificationsCubit` doesn't exist yet at setup time → resolution blows up. `[FIREBASE_DI_ORDER]`.

## 6. Firebase-backed Cubit + State + `Equatable`

Additional rules for cubits that make Firebase calls (the base Cubit/State rules are in [../flutter-performance-reviewer/SKILL.md](../flutter-performance-reviewer/SKILL.md)):

### 6.1 The cubit calls the service and updates state

```dart
Future<void> _runAction(String name, Future<void> Function() task) async {
  if (state.isSubmitting) return;            // double-trigger protection
  emit(state.copyWith(isSubmitting: true));
  try {
    await task();                            // _authService.verifyOtp(...) etc.
  } on AuthServiceException catch (e) {       // domain exception only
    _effects.add(AuthEffectMessage(e.message));
  } finally {
    if (!isClosed) emit(state.copyWith(isSubmitting: false)); // isClosed guard
  }
}
```

- The **`isSubmitting` guard** is checked at the start of every async action; if the user taps the button twice, a double OTP/login request won't go out.
- The `emit` inside `finally` must always be guarded with **`if (!isClosed)`**; if the view closes before the async job finishes, an "emit after close" error occurs. `[FIREBASE_EMIT_AFTER_CLOSE]`.
- The `TextEditingController`, `FocusNode`, `Timer` (OTP countdown), and `StreamController` (effects) held inside the cubit are **all** disposed/cancelled/closed in the `close()` override.

### 6.2 One-off side effects via an `effect` stream instead of `state`

Events that must happen **once** — navigation, a snackbar, "go to the OTP screen" — are not put in `state` (state is persistent → re-triggers on rebuild). Instead use a separate effect stream:

```dart
final _effects = StreamController<AuthEffect>.broadcast();
Stream<AuthEffect> get effects => _effects.stream;
// on the view side: StreamBuilder/listen → Navigator / SnackBar
```

If a flag like `navigateToOtp: true` is seen in state, that's `[FIREBASE_EFFECT_IN_STATE]` — a one-off event must move to the effect stream (or a `BlocListener` + a marker that's reset once consumed).

### 6.3 Firebase response models must be **immutable** `Equatable`

Models carrying the Firebase-auth response from the backend (`FirebaseAuthResponseModel` etc.) extend `Equatable`, but their fields must be **`final` + a `const` constructor**. The mutable-field + `// ignore_for_file: must_be_immutable` pattern is a code smell:

**Smelly (current):**

```dart
// ignore_for_file: must_be_immutable
class FirebaseAuthResponseModel extends Equatable {
  String? id;        // mutable — conflicts with Equatable
  String? firstName;
  FirebaseAuthResponseModel({this.id, this.firstName});
  // ...
}
```

**Desired:**

```dart
final class FirebaseAuthResponseModel extends Equatable {
  const FirebaseAuthResponseModel({this.id, this.firstName});
  final String? id;
  final String? firstName;
  // fromMap / props stay the same
}
```

`Equatable`'s contract assumes immutability: `props` must stay stable so hashCode/`==` are consistent. If a field can change later, equality silently breaks. When writing a new model, don't suggest mutable + a `must_be_immutable` ignore; flag it with `[EQUATABLE_MUTABLE]`. (Don't do a bulk rename of existing models **without a request** — see Surgical Changes.)

## 7. Init — `main.dart`

- Keep the order `WidgetsFlutterBinding.ensureInitialized()` → `await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform)` → `ServiceLocator().setup()` → `runApp(...)`. Firebase must be initialized **before** the DI setup (services need `FirebaseAuth.instance`).
- `firebase_options.dart` is a generated file; it isn't edited by hand and isn't kept out of `.gitignore`, but the API keys inside it are considered public (Firebase security is provided by App Check + rules, not by secrecy).

## 8. Review tags

```
[FIREBASE_SCOPE]           — an unexpected Firebase service (e.g. Firestore) entered the architecture
[FIREBASE_LAYER]           — Firebase SDK leaked into cubit/view/repository/datasource
[FIREBASE_TESTABILITY]     — FirebaseAuth not injected, .instance called inside a method
[FIREBASE_EXCEPTION]       — FirebaseAuthException caught outside the service / not mapped to a domain type
[FIREBASE_TOKEN]           — ID token managed manually in cubit/view/datasource
[FIREBASE_DI_ORDER]        — firebase service in the wrong phase / can't resolve with an eager registration
[FIREBASE_EMIT_AFTER_CLOSE]— emit without an isClosed guard after an async firebase job
[FIREBASE_EFFECT_IN_STATE] — a one-off event (nav/snackbar) placed in state
[EQUATABLE_MUTABLE]        — Equatable model with mutable fields + a must_be_immutable ignore
[FIREBASE_INIT]            — wrong init order in main.dart
```

## 9. Review checklist (when the skill runs)

1. **Layer:** Are `package:firebase_*` imports only inside `common/service/` (and the `main.dart` init)?
2. **Testability:** Is the SDK injected in the constructor of a Firebase-dependent service, and not `.instance` called inside a method?
3. **Exception:** Does the service wrap every Firebase call with `on FirebaseAuthException` + a generic `catch` and convert it to a domain exception? Does the cubit catch only the domain exception?
4. **Token:** Is ID token management in the service + interceptor? Is there a `getIdToken(true)` after a claim change?
5. **Messaging:** Does FCM `registerToken` avoid blocking login (`unawaited` + internal error swallowing)? Is the DI phase order correct?
6. **Cubit/State:** `isSubmitting` guard, `finally` + `isClosed` guard, and are all controllers/timers/streams cleaned up in `close()`?
7. **Effect vs state:** Is navigation/snackbar in the effect stream, not a flag in state?
8. **Model:** Are Firebase response models immutable `Equatable` (for new code)?
9. **Init:** Is the `ensureInitialized → initializeApp → DI setup → runApp` order preserved in `main.dart`?

## 10. Skill execution protocol

1. Get the changed files via `git status` + `git diff main...HEAD`.
2. Evaluate each changed file against the section 9 checklist; skip unrelated files.
3. Output format:
   - **Conforms:** a short confirmation sentence.
   - **Deviation:** `path:line — [TAG] — description of the violation — suggested fix.`
4. Group findings by section heading (Layer / Exception / Token / Messaging / Cubit-State / Model / Init). Don't print empty groups.
5. Don't make code edits on your own; if there's ambiguity, explicitly flag "I'm stuck on this, let the user decide" (Think Before Coding).
