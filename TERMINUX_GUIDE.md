# 🛠️ دليل Termux — بناء APK من التابلت مباشرة

> هذا الدليل لمن عنده **تابلت قوي** (يفضّل 4GB+ RAM) ويريد يبني الـ APK بدون كمبيوتر.
> الخيار الأسهل للمبتدئين: استخدم **GitHub Actions** بدلاً من هذا.

## 📋 المتطلبات

- تابلت Android 7.0+
- مساحة فارغة **3GB+** على التابلت
- اتصال إنترنت (لتنزيل الأدوات والمكتبات)

---

## الخطوة 1: تثبيت Termux

### من F-Droid (موصى به):
1. افتح F-Droid: https://f-droid.org
2. ابحث عن "Termux"
3. حمّل وثبّت

### من Google Play (لا يُنصح):
> ❌ Termux على Google Play قديم وغير محدّث. استخدم F-Droid.

---

## الخطوة 2: تحديث Termux وتثبيت الأساسيات

افتح Termux وانسخ الأوامر التالية واحد واحد:

```bash
# تحديث الحزم
pkg update && pkg upgrade -y

# تثبيت الأساسيات
pkg install -y git wget curl unzip openjdk-17 zip
```

> ⏱️ قد تأخذ 2-5 دقائق حسب سرعة الإنترنت

---

## الخطوة 3: تحميل المشروع

### الخيار A: إذا عندك GitHub repo
```bash
git clone https://github.com/yourname/KhayrityApp.git
cd KhayrityApp
```

### الخيار B: نقل ZIP من الكمبيوتر
1. ضع `KhayrityApp.zip` في مجلد Downloads على التابلت
2. في Termux:
```bash
cp ~/storage/downloads/KhayrityApp.zip ~
cd ~
unzip KhayrityApp.zip
cd KhayrityApp
```

> إذا `~/storage` لا يعمل، جرّب:
> ```bash
> termux-setup-storage
> ```

---

## الخطوة 4: تحميل Android SDK

```bash
# تحميل Command Line Tools (~150MB)
cd ~
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-11076708_latest.zip
mkdir -p android-sdk/cmdline-tools
mv cmdline-tools android-sdk/cmdline-tools/latest
rm commandlinetools-linux-11076708_latest.zip

# إعداد متغيرات البيئة
echo 'export ANDROID_HOME=$HOME/android-sdk' >> ~/.bashrc
echo 'export ANDROID_SDK_ROOT=$ANDROID_HOME' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools' >> ~/.bashrc
source ~/.bashrc
```

---

## الخطوة 5: قبول التراخيص وتثبيت SDK

```bash
yes | sdkmanager --licenses
sdkmanager "platforms;android-35" "build-tools;35.0.0" "platform-tools"
```

> ⏱️ قد تأخذ 5-10 دقائق (تنزّل ~500MB)

---

## الخطوة 6: إعداد المشروع

```bash
cd ~/KhayrityApp

# إنشاء local.properties
echo "sdk.dir=$HOME/android-sdk" > local.properties

# تحميل gradle-wrapper.jar
wget -q https://raw.githubusercontent.com/gradle/gradle/v8.10.2/gradle/wrapper/gradle-wrapper.jar -O gradle/wrapper/gradle-wrapper.jar

# ضبط Gradle للذاكرة المنخفضة
cat >> gradle.properties <<'EOF'

# Low-memory tuning
org.gradle.jvmargs=-Xmx1536m -XX:MaxMetaspaceSize=512m
org.gradle.daemon=false
org.gradle.parallel=false
EOF

# منح صلاحية التنفيذ
chmod +x gradlew
```

---

## الخطوة 7: بناء APK

```bash
./gradlew assembleDebug --no-daemon
```

> ⏱️ **أول مرة:** 10-20 دقيقة (ينزّل كل المكتبات ويبني)
> ⏱️ **بعدها:** 2-5 دقائق

---

## الخطوة 8: إيجاد الـ APK

```bash
ls -lh app/build/outputs/apk/debug/
```

ستجد:
```
app-debug.apk  (حوالي 15-20 MB)
```

---

## الخطوة 9: تثبيت APK على التابلت

### من Termux مباشرة:
```bash
# ثبّت ADB
pkg install -y android-tools

# وصّل التابلت بـ USB... لا، هذا للتابلت نفسه 😄
# الخيار: استخدم مدير ملفات يفتح الـ APK
termux-open app/build/outputs/apk/debug/app-debug.apk
```

### يدوياً:
1. انسخ الـ APK إلى `/storage/emulated/0/Download/`:
   ```bash
   cp app/build/outputs/apk/debug/app-debug.apk ~/storage/downloads/
   ```
2. افتح مدير الملفات (Files by Google مثلاً)
3. اذهب إلى Downloads
4. اضغط على `app-debug.apk`
5. اسمح بتثبيت من مصادر غير معروفة (Unknown sources)
6. اضغط Install

---

## ⚠️ مشاكل شائعة وحلولها

| المشكلة | الحل |
|---|---|
| `java not found` | تأكد من `pkg install openjdk-17` |
| `SDK location not found` | تحقق من `local.properties` |
| Out of Memory | أغلق كل التطبيقات الأخرى + خفّض `Xmx` إلى 1024m |
| Build بطيء جداً | عادي! أول build يأخذ 15-20 دقيقة |
| `Permission denied` على `gradlew` | `chmod +x gradlew` |
| لا مساحة كافية | تحتاج 3GB+ فارغة |

---

## 💡 نصيحة

> **إذا الـ build فشل أو التابلت ما يحمل، استخدم GitHub Actions** (الأسهل).
> أنشئ repo على GitHub من المتصفح، ارفع المشروع، وسيبني الـ APK تلقائياً.
> راجع `GITHUB_ACTIONS_GUIDE.md` للتفاصيل.

---

## 🎁 اختصارات مفيدة

```bash
# تنظيف وإعادة بناء
./gradlew clean assembleDebug --no-daemon

# بناء + اختبارات
./gradlew assembleDebug test --no-daemon

# حجم الـ APK
ls -lh app/build/outputs/apk/debug/app-debug.apk

# مشاركة APK عبر Termux
termux-share app/build/outputs/apk/debug/app-debug.apk
```

بالتوفيق! 🚀
