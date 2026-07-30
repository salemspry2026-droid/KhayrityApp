# Architecture — خريطتي (Khayrity)

> توثيق قرارات التصميم المعماري للتطبيق. هدفه: أن يكون أي مطور جديد قادراً على فهم لماذا كل قرار اتُخذ، وإضافة ميزات دون كسر البنية.

---

## 1. المبادئ التوجيهية

| المبدأ | التطبيق |
|---|---|
| **Offline-First** | كل البيانات محلية. لا توجد خدمات سحابية في الـ MVP. |
| **Privacy-First** | لا analytics، لا tracking، لا أذونات إنترنت. |
| **RTL-First** | العربية هي اللغة الأساسية، الإنجليزية مضافة لاحقاً. |
| **Gesture-First** | كل العمليات تتم عبر اللمس، لا قوائم معقدة. |
| **Modular** | كل feature في package مستقل قابل للإزالة/الاستبدال. |
| **Reactive** | كل تدفقات البيانات من Room/DataStore → StateFlow → Compose. |
| **Testable** | Domain Layer pure-Kotlin، قابل للاختبار بدون Android. |

---

## 2. Clean Architecture — 3 طبقات

```
┌─────────────────────────────────────────────────────────────┐
│                       UI Layer                              │
│  • Composables (Stateless)                                  │
│  • ViewModels (State holders)                               │
│  • State (immutable data classes)                           │
│  → يستهلك Domain Layer فقط                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓ uses
┌─────────────────────────────────────────────────────────────┐
│                     Domain Layer                            │
│  • Models (data classes خالصة)                              │
│  • UseCases (single-purpose)                                │
│  • Repository interfaces                                    │
│  → لا يعرف عن Android أو Room أو Compose                   │
└─────────────────────────────────────────────────────────────┘
                            ↑ implements
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                             │
│  • Room: Database, DAOs, Entities                           │
│  • RepositoryImpl (يطبق domain interface)                   │
│  • Mappers (Entity ↔ Domain)                                │
│  • DataStore (preferences)                                  │
│  • Exporters (JSON, PNG, OPML)                              │
└─────────────────────────────────────────────────────────────┘
```

### لماذا Clean Architecture؟
- **استبدال أي طبقة** دون تأثير على الباقي (مثلاً استبدال Room بـ SQLDelight)
- **اختبار Domain** بدون أي اعتماد على Android (سرعة + عزل)
- **فصل واضح للمسؤوليات** → فريق أكبر يقدر يشتغل بالتوازي

---

## 3. Unidirectional Data Flow (UDF)

```
┌──────────────┐  events   ┌──────────────┐
│   Composable │ ────────▶ │  ViewModel   │
│   (UI)       │           │              │
└──────────────┘           └──────────────┘
        ▲                        │
        │ state                  │ uses
        │                        ▼
        │                ┌──────────────┐
        │                │   UseCase    │
        │                └──────────────┘
        │                        │
        │                        ▼
        │                ┌──────────────┐
        └────  collect ─ │  Repository  │
                         └──────────────┘
                                │
                                ▼
                         ┌──────────────┐
                         │ Room Database│
                         └──────────────┘
```

- **State** = immutable data class (`HomeState`, `EditorState`, `SettingsState`)
- **Events** = دوال في الـ ViewModel (`onQueryChange`, `addNodeAt`)
- **StateFlow** = قناة الاتصال من VM إلى UI

---

## 4. نظام Undo/Redo (Command Pattern)

### المشكلة
كل عملية على الخريطة (إضافة/حذف/نقل/تعديل) يجب أن تكون قابلة للتراجع. حلول تقليدية مثل `Snapshot` لكل حالة تستهلك ذاكرة ضخمة.

### الحل: Command Pattern
```kotlin
interface Command {
    val label: String
    fun apply(current: MindMap): MindMap
    fun revert(current: MindMap): MindMap
}
```

- كل عملية هي `Command` (مثل `AddNodeCommand`, `MoveNodeCommand`)
- `CommandHistory` يحتفظ بمكدسين: `undoStack` و `redoStack`
- كل command ينفذ `apply` ثم يُدفع على `undoStack` ويمسح `redoStack`
- `undo(current)` ينفذ `revert` ويبدّل المكدسين
- مكدسات محدودة بـ 50 خطوة (`Constants.UNDO_HISTORY_LIMIT`)

### المميزات
- **ذاكرة محدودة**: 50 command × ~200 بايت = 10KB فقط
- **سهل الإضافة**: أمر جديد = class واحد يطبق `Command`
- **قابل للاختبار**: `CommandHistoryTest` يختبر السلوك بدون UI

---

## 5. Canvas Rendering

### القرار: Compose Canvas (وليس View/SurfaceView)
- **دافعي**: API موحد، gesture handling مدمج، RTL تلقائي
- **البدائل المرفوضة**:
  - `SurfaceView`: معقد، يحتاج lifecycle يدوي
  - `Custom View`: يكرّر الكثير من work Compose
  - **GraphView / third-party**: Locks us in, no RTL

### Pan/Zoom
- `detectTransformGestures` يلتقط pinch + pan
- التحويلات تُخزَّن في `state.canvasScale / Offset`
- جميع الإحداثيات على الـ canvas تستخدم نظام إحداثيات **عالمي** (لا شاشة)
- التحويل `screenToWorld` يستخدم `offset + scale`

### Rendering
- `Canvas` Composable يعرض:
  1. **الشبكة (grid)** — خطوط رفيعة على فواصل 48dp
  2. **الروابط (connections)** — `Path` بـ Bézier curves
  3. **العقد (nodes)** — `drawRoundRect` + نص
- النص يُرسم عبر `nativeCanvas.drawText` للحصول على خط نظيف
- الترتيب: grid → connections → nodes (z-order)

### الأداء
- لا نستخدم `recomposition` لكل frame — التحويلات تُحدّث state فقط
- `Canvas` نفسه يتولى `drawScope` بكفاءة
- في الإصدار المستقبلي: نقل الـ rendering إلى `GraphicsLayer` للتسريع

---

## 6. إدارة الحالة: لماذا StateFlow وليس LiveData؟

| الميزة | StateFlow | LiveData |
|---|---|---|
| Coroutines native | ✅ | ❌ (يحتاج `asLiveData()`) |
| Multiplatform-ready | ✅ | ❌ |
| Type-safe | ✅ | ❌ |
| مع Compose | ✅ عبر `collectAsStateWithLifecycle` | ⚠️ عبر `observeAsState` |
| الاختبار | `turbine` بسيط | أعقد |

→ اخترنا `StateFlow` للاتساق مع stack الحديثة.

---

## 7. DI: Hilt (وليس Koin أو Dagger يدوي)

### لماذا Hilt؟
- **تكامل رسمي مع Jetpack** (ViewModel, WorkManager, Navigation)
- **أقل boilerplate** من Dagger اليدوي
- **سهل التعلم** للمطورين الجدد
- **متوافق مع KSP** (أسرع من kapt)

### البنية
```
@HiltAndroidApp
class KhayrityApplication : Application()

@AndroidEntryPoint
class MainActivity : ComponentActivity()

@HiltViewModel
class HomeViewModel @Inject constructor(... use cases ...)
```

### Modules
- `DatabaseModule` (object): DAOs + Database
- `RepositoryModule` (abstract): Repository bindings
- `DispatchersModule`: Coroutine dispatchers via qualifiers

---

## 8. المزامنة والتزامن Concurrency

### Auto-save
- كل تغيير يستدعي `viewModel.applyLocalAndPersist(state)`
- `Debouncer` يجمع التغييرات خلال 500ms ثم يحفظ مرة واحدة
- هذا يخفف الضغط على Room عند الكتابة السريعة

### Threads
- `@IoDispatcher` → Room operations + JSON parsing
- `@DefaultDispatcher` → PNG rendering + computation
- `@MainDispatcher` → UI work فقط
- الـ Qualifiers تتيح استبدال كل dispatcher في الاختبارات بـ `TestDispatcher`

### Concurrency safety
- كل DAO إما `suspend fun` (write) أو `Flow<...>` (read)
- Room يضمن thread-safety على عمليات الكتابة
- الـ Repository يدمج 3 flows عبر `combine` → يُرجع `MindMap` كامل

---

## 9. الأخطاء والاستثناءات

### الاستراتيجية
- **`Result<T>` / `Flow<Result<T>>`**: في UseCases الحرجة (مثل التصدير)
- **Exceptions مدروسة**: في طبقة Data (مثل `IOException` من Disk)
- **`Snackbar` events**: في طبقة UI عبر `SharedFlow<UiEvent>`
- **`fallbackToDestructiveMigration`**: في v1 فقط، استبدلها بـ Migrations من v2

### Logging
- لا نستخدم `Log.d` في production (Performance)
- `Timber` جاهز للإضافة عند الحاجة
- Crashes تُسجَّل في Crashlytics (مستقبلاً، ليس في MVP)

---

## 10. الاختبارات

### الهرم
```
        ┌──── UI tests (Compose UI Test)
        │     • Editor screen renders empty state
        │     • Clicking FAB navigates
        ├──── ViewModel tests
        │     • State updates on event
        │     • SavedStateHandle integration
        ├──── Repository tests (Room in-memory)
        │     • create → observe roundtrip
        │     • delete cascades correctly
        └──── UseCase / Pure unit tests
              • SearchMindMapsUseCaseTest
              • CommandHistoryTest
              • OpmlExporterTest
```

### الأدوات
- **JUnit 4** — baseline
- **MockK** — Kotlin-native mocking
- **Turbine** — اختبار Flow بسهولة
- **Truth** — assertions قابلة للقراءة
- **Robolectric** — اختبارات تحتاج `Context` (للـ DataStore)

### ما لا نختبره في الـ MVP
- Composables معقدة (مثل `MindMapCanvas`) — نختبرها يدوياً + عبر screenshot tests مستقبلاً
- Compose preview — اختبارات يدوية

---

## 11. القرارات المرفوضة (وليش)

| القرار | البديل المرفوض | السبب |
|---|---|---|
| Room | SQLDelight | Room أبسط + تكامل مع Hilt |
| Hilt | Koin | Hilt official + KSP support |
| Compose Canvas | Custom View | Compose API أحدث + gesture handling |
| Kotlinx Serialization | Moshi/Gson | Kotlinx pure-Kotlin + KSP-friendly |
| DataStore | SharedPreferences | DataStore آمن + Flow-based |
| StateFlow | LiveData | Compose + Coroutines |
| Navigation Compose | Fragments | Compose-native + type-safe |
| Dark mode via DataStore | Manual | تخصيص المستخدم متوقع |
| In-app web | Custom browser | لا يوجد web في الـ MVP |

---

## 12. خارطة الطريق (مستقبلاً)

### v1.1
- تصدير/استيراد JSON كامل
- استيراد OPML
- اختصارات لوحة المفاتيح
- Tablet layout (master-detail)

### v1.2
- AI assistant (3 شخصيات)
- تحويل نص → خريطة
- ربط ثنائي بين الخرائط

### v2.0
- Memory Palace 3D
- Time-Travel animation
- Mood-adaptive theming
- Spaced repetition learning
- Wear OS companion

---

## 13. مصادر مرجعية

- [Now in Android](https://github.com/android/nowinandroid) — مرجعنا المعماري
- [Android Architecture Guide](https://developer.android.com/topic/architecture)
- [Jetpack Compose Docs](https://developer.android.com/jetpack/compose)
- [Hilt Testing Guide](https://developer.android.com/training/dependency-injection/hilt-testing)

---

آخر تحديث: 2026-07-27
