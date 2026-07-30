# خريطتي (Khayrity) — Mind Map Notes

> تطبيق أندرويد أصلي وبسيط لتنظيم أفكارك بصرياً عبر خرائط ذهنية. **أوفلاين بالكامل**، **خاص**، **يدعم العربية RTL أولاً**، ويستخدم **Jetpack Compose + Material 3**.

---

## ✨ المزايا

- 🎨 **لوحة Canvas لانهائية** — Pan/Zoom بإصبعين، شبكة بصرية، أداء 60fps
- 🔵 **عقد قابلة للتخصيص** — نص، لون، شكل (مستدير/كبسولة/مستطيل/سحابة)
- ➡️ **روابط منحنية** — Bézier curves مع أسهم اتجاه
- ↩️ **تراجع وإعادة** — 50 خطوة (Command Pattern)
- 💾 **حفظ تلقائي** — كل تغيير يُحفظ بعد 500ms
- 🌓 **وضع ليلي/نهاري** — يتبع النظام افتراضياً، قابل للتثبيت
- 🌐 **عربية + إنجليزية** — RTL-first
- 📤 **تصدير** — PNG (صورة)، JSON (نسخة احتياطية)، OPML (توافق مع تطبيقات أخرى)
- 🔒 **خصوصية كاملة** — لا جمع بيانات، لا إعلانات، لا حساب

---

## 🛠️ الحزمة التقنية

| الطبقة | الأدوات |
|---|---|
| اللغة | Kotlin 100% |
| UI | Jetpack Compose + Material 3 |
| Architecture | Clean Architecture (3 طبقات) + MVVM + UDF |
| DI | Hilt |
| Database | Room (مع TypeConverters) + DataStore |
| Async | Coroutines + Flow + StateFlow |
| Navigation | Navigation Compose (type-safe) |
| Serialization | Kotlinx Serialization |
| Build | Gradle KTS + Version Catalog |
| Testing | JUnit4, MockK, Turbine, Truth, Compose UI Test |

- **minSdk** 24 (Android 7.0) | **targetSdk** 35 (Android 15) | **JDK** 17

---

## 📁 بنية المشروع

```
KhayrityApp/
├── app/
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── java/com/mindmap/khayrity/
│       │   │   ├── KhayrityApplication.kt       # @HiltAndroidApp
│       │   │   ├── MainActivity.kt
│       │   │   ├── core/
│       │   │   │   ├── designsystem/            # Theme, Color, Type, Shape, Spacing
│       │   │   │   └── utils/                   # Constants, Time, IdGenerator, Debouncer
│       │   │   ├── domain/                      # Pure Kotlin (لا Android)
│       │   │   │   ├── model/                   # MindMap, Node, Connection
│       │   │   │   ├── repository/              # MindMapRepository (interface)
│       │   │   │   └── usecase/                 # 9 UseCases
│       │   │   ├── data/
│       │   │   │   ├── local/                   # Room: KhayrityDatabase, DAOs, Entities
│       │   │   │   ├── mapper/                  # Mappers Entity↔Domain
│       │   │   │   ├── repository/              # MindMapRepositoryImpl
│       │   │   │   ├── preferences/             # SettingsDataStore
│       │   │   │   └── export/                  # JsonExporter, PngExporter, OpmlExporter
│       │   │   ├── di/                          # Hilt modules
│       │   │   ├── navigation/                  # Routes + NavGraph
│       │   │   ├── undo/                        # Command Pattern + CommandHistory
│       │   │   └── ui/
│       │   │       ├── home/                    # HomeScreen, HomeViewModel, HomeState
│       │   │       ├── editor/                  # EditorScreen, ViewModel, State
│       │   │       │   └── canvas/              # MindMapCanvas
│       │   │       └── settings/                # SettingsScreen, ViewModel
│       │   └── res/
│       │       ├── values/strings.xml           # Arabic (default)
│       │       ├── values-en/strings.xml        # English
│       │       ├── values/colors.xml
│       │       ├── values/themes.xml
│       │       ├── drawable/                    # Launcher icons
│       │       ├── mipmap-anydpi-v26/           # Adaptive icon
│       │       └── xml/                         # Backup rules
│       └── test/                                # Unit tests
├── build.gradle.kts                             # Root build
├── settings.gradle.kts
├── gradle.properties
├── gradle/
│   ├── libs.versions.toml                       # Version catalog
│   └── wrapper/gradle-wrapper.properties
├── ARCHITECTURE.md                              # Architecture decisions
├── README.md                                    # This file
└── .gitignore
```

---

## 🚀 كيف تبني وتشغّل

### 1. المتطلبات
- **Android Studio Ladybug 2024.2.1+** أو أحدث
- **JDK 17** (موصى به: Zulu أو Temurin)
- **Android SDK 35** (يُحمَّل تلقائياً من SDK Manager)
- **Gradle 8.10.2** (يُحمَّل تلقائياً عبر Wrapper)

### 2. افتح المشروع
1. افتح Android Studio
2. اختر `File → Open`
3. اختر مجلد `KhayrityApp`
4. انتظر Gradle Sync (أول مرة قد تأخذ 3-5 دقائق لتحميل التبعيات)

### 3. شغّل على جهاز
1. وصّل جهاز Android عبر USB (مفعّل Developer Options + USB Debugging)
2. اضغط ▶ Run
3. اختر `app` كـ Configuration

### 4. ابنِ APK من سطر الأوامر
```bash
# Debug APK
./gradlew assembleDebug
# → app/build/outputs/apk/debug/app-debug.apk

# Release APK (مع R8/ProGuard)
./gradlew assembleRelease
# → app/build/outputs/apk/release/app-release.apk
```

> **ملاحظة:** Release build يحتاج keystore. للتجربة السريعة، الـ debug build يكفي.

### 5. شغّل الاختبارات
```bash
./gradlew test              # Unit tests
./gradlew connectedAndroidTest  # Instrumented tests (تحتاج جهاز/محاكي)
```

---

## 🎨 نظام التصميم

| Token | القيمة |
|---|---|
| Primary | `#3B3B8F` (Indigo) |
| Secondary | `#4ECDC4` (Mint) |
| Tertiary | `#FF6B6B` (Coral) |
| خط عربي | Cairo (مُفعّل عبر Compose — حالياً النظام الافتراضي) |
| خط إنجليزي | Inter |
| الشبكة | 4dp |
| Animation | 300ms Spring |

كل الألوان والـ Spacing معرّفة في `core/designsystem/`. لتغييرها، عدّل ملف واحد فقط.

---

## 🌐 إضافة لغة جديدة

1. أنشئ `app/src/main/res/values-<locale>/strings.xml`
2. انسخ كل مفاتيح `strings.xml` وترجمها
3. أضف الـ locale إلى `resourceConfigurations` في `app/build.gradle.kts`

```kotlin
resourceConfigurations += listOf("en", "ar", "fr", "es")
```

---

## 📦 إضافة ميزة جديدة

مثال: إضافة ميزة "إعجاب/تمييز النجمة" على العقد

### الخطوة 1: Domain
```kotlin
// domain/model/Node.kt
data class Node(
    ...existing fields,
    val isStarred: Boolean = false,
)

// domain/usecase/ToggleNodeStarUseCase.kt
class ToggleNodeStarUseCase @Inject constructor(
    private val repository: MindMapRepository,
) {
    suspend operator fun invoke(nodeId: Long) { ... }
}
```

### الخطوة 2: Data
- أضف العمود `isStarred: Boolean` إلى `NodeEntity`
- زِد `DATABASE_VERSION` واكتب Migration
- حدّث `Mappers.kt`

### الخطوة 3: UI
- أضف action في `NodeEditorDialog`
- استدعِ الـ UseCase من `EditorViewModel`

كل ميزة جديدة تلمس **3 ملفات فقط** في كل طبقة — هذا قوة Clean Architecture.

---

## 🤝 المساهمة

1. Fork → Feature branch → PR
2. التزم بـ [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
3. أضف اختبارات لكل feature جديد
4. شغّل `./gradlew detekt` قبل الـ commit

---

## 📄 الرخصة

MIT License — استعمل، عدّل، وزّع كما تشاء.

---

## 💬 الدعم

- افتح Issue في GitHub
- أو راسلنا عبر البريد

استمتع بخريطة أفكارك! 🧠✨
