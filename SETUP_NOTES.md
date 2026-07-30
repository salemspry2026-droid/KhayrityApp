# 📋 ملاحظات الإعداد — مهم قبل أول تشغيل

عند فتح المشروع في Android Studio لأول مرة، اتبع الخطوات التالية:

## 1️⃣ توليد Gradle Wrapper JAR

الملف `gradle/wrapper/gradle-wrapper.jar` غير مضغوط في الـ ZIP لأنه ثنائي.
عند فتح المشروع في Android Studio، سيُولَّد تلقائياً عند أول Gradle Sync.

**إذا ما اشتغل تلقائياً:**

### الطريقة 1: من Android Studio
- `File → Settings → Build, Execution, Deployment → Build Tools → Gradle`
- اضغط على `Use Gradle from:` واختر `gradle-wrapper.properties`
- أو من Terminal داخل Android Studio: `./gradlew wrapper --gradle-version 8.10.2`

### الطريقة 2: من سطر الأوامر (إذا عندك Gradle مثبت)
```bash
cd KhayrityApp
gradle wrapper --gradle-version 8.10.2
```

### الطريقة 3: تحميل يدوي
حمّل `gradle-wrapper.jar` من:
https://github.com/gradle/gradle/raw/v8.10.2/gradle/wrapper/gradle-wrapper.jar

وضعه في: `KhayrityApp/gradle/wrapper/`

---

## 2️⃣ إنشاء local.properties

انسخ `local.properties.example` إلى `local.properties` وعدّل المسار:

**Windows:**
```
sdk.dir=C\:\\Users\\YourName\\AppData\\Local\\Android\\Sdk
```

**macOS:**
```
sdk.dir=/Users/YourName/Library/Android/sdk
```

**Linux:**
```
sdk.dir=/home/YourName/Android/Sdk
```

> ✅ **أو ببساطة:** افتح المشروع في Android Studio، الـ IDE سيكتشف SDK تلقائياً.

---

## 3️⃣ أول Gradle Sync

1. افتح Android Studio
2. `File → Open → KhayrityApp`
3. انتظر أول Gradle Sync (3-5 دقائق لتحميل dependencies)
4. إذا ظهرت أخطاء، اضغط على "Try Again" أو `File → Sync Project with Gradle Files`

---

## 4️⃣ تشغيل التطبيق

1. وصّل جهاز Android أو شغّل محاكي
2. اضغط ▶ Run
3. اختر `app` كـ configuration
4. جرّب!

---

## 5️⃣ بناء APK

```bash
# من سطر الأوامر داخل مجلد KhayrityApp:
./gradlew assembleDebug
```

الـ APK سيكون في: `app/build/outputs/apk/debug/app-debug.apk`

---

## ❓ مشاكل شائعة وحلولها

| المشكلة | الحل |
|---|---|
| `SDK location not found` | أنشئ `local.properties` كما في الخطوة 2 |
| `Gradle Wrapper not found` | شغّل `gradle wrapper` من سطر الأوامر |
| `Java 17 not found` | ثبّت JDK 17 من [Adoptium](https://adoptium.net) |
| `License for package ... not accepted` | `sdkmanager --licenses` ثم اقبل الكل |
| `KSP not found` | تأكد إن الـ sync اكتمل، KSP يولّد ملفات عند الطلب |
| Build بطيء | أول build دائماً بطيء، الـ incremental بعدها أسرع بكثير |

---

## 🆘 إذا واجهت مشكلة

1. تحقق من [Issues](https://github.com/yourname/Khayrity/issues) (بعد رفعه على GitHub)
2. تأكد من JDK 17 و Android Studio Ladybug+
3. نظّف وأعد البناء: `./gradlew clean && ./gradlew assembleDebug`
4. أوقف Anti-Virus مؤقتاً (أحياناً يحذف ملفات)

استمتع! 🚀
