# 🚀 دليل سريع: تجربة التطبيق من التابلت فقط

> 4 حلول مرتّبة من الأسهل للأقوى. اختر اللي يناسبك.

---

## 🏆 الحل 1: GitHub Actions — **الأسهل والأسرع** (10 دقائق)

### الفكرة
ترفع المشروع على GitHub → سيرفرات GitHub تبني الـ APK → تنزّله على تابلتك.

### الخطوات من التابلت

#### 1. افتح متصفح التابلت واذهب لـ:
```
https://github.com/signup
```

#### 2. أنشئ حساب مجاني (إذا ما عندك)
- Email
- Password
- Username

#### 3. أنشئ Repository جديد
- اضغط `+` (أعلى يمين) → `New repository`
- اسم: `KhayrityApp`
- اختر Public (مجاني بالكامل)
- ❌ **لا تضف** README/.gitignore/license
- اضغط `Create repository`

#### 4. ارفع ملفات المشروع

**الطريقة الأسهل: رفع ZIP**

في صفحة الـ repo الجديدة، سترى:
```
or upload an existing file
```
اضغط عليها واسحب **محتويات** مجلد `KhayrityApp/` (لا تضغط المجلد نفسه بل الملفات داخله).

⚠️ **مشكلة شائعة:** GitHub قد يرفض رفع ملف `gradlew` (لأنه script). إذا حصل:
1. ارفع كل الملفات **ما عدا** `gradlew`
2. استخدم Termux (الحل 2) لإضافة `gradlew` فقط

#### 5. شغّل البناء

1. اضغط تبويب `Actions` (أعلى الصفحة)
2. ستلاقي workflow اسمه `Build Debug APK` — اضغط عليه
3. اضغط `Run workflow` → `Run workflow` (زر أخضر)
4. انتظر 5-7 دقائق (شاي/قهوة ☕)

#### 6. نزّل الـ APK

1. اضغط على الـ run اللي انتهى (علامة ✅)
2. انزل لأسفل لقسم `Artifacts`
3. اضغط `khayrity-debug-apk` لتنزيل ملف ZIP
4. افتح الملف من `Downloads` على تابلتك
5. **فك الضغط** (Extract) → يعطيك `app-debug.apk`

#### 7. ثبّت التطبيق

1. افتح `app-debug.apk`
2. اضغط `INSTALL`
3. افتح التطبيق واستمتع! 🎉

📖 **التفاصيل الكاملة في:** `GITHUB_ACTIONS_GUIDE.md`

---

## 🛠️ الحل 2: Termux (على التابلت مباشرة)

> يحتاج تابلت **قوي** (4GB+ RAM، 3GB+ مساحة فارغة). تابلتك 2GB RAM؟ استخدم الحل 1.

### الخطوات

#### 1. ثبّت Termux

- ⚠️ **مهم:** ثبّته من **F-Droid** (https://f-droid.org) وليس Google Play
- في F-Droid، ابحث "Termux" → Install

#### 2. افتح Termux ونفّذ الأوامر التالية (انسخ-الصق):

```bash
# تحديث وتثبيت الأساسيات
pkg update && pkg upgrade -y
pkg install -y git wget unzip openjdk-17

# تحميل المشروع (إذا رفعته على GitHub)
git clone https://github.com/yourname/KhayrityApp.git
cd KhayrityApp

# تحميل وتثبيت Android SDK
cd ~
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-11076708_latest.zip
mkdir -p android-sdk/cmdline-tools
mv cmdline-tools android-sdk/cmdline-tools/latest
rm commandlinetools-linux-11076708_latest.zip

echo 'export ANDROID_HOME=$HOME/android-sdk' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin' >> ~/.bashrc
source ~/.bashrc

# قبول التراخيص + تثبيت SDK
yes | sdkmanager --licenses
sdkmanager "platforms;android-35" "build-tools;35.0.0" "platform-tools"

# إعداد المشروع
cd ~/KhayrityApp
echo "sdk.dir=$HOME/android-sdk" > local.properties
wget -q https://raw.githubusercontent.com/gradle/gradle/v8.10.2/gradle/wrapper/gradle-wrapper.jar -O gradle/wrapper/gradle-wrapper.jar
chmod +x gradlew

# بناء APK (10-20 دقيقة في أول مرة)
./gradlew assembleDebug --no-daemon
```

#### 3. الـ APK في:
```
~/KhayrityApp/app/build/outputs/apk/debug/app-debug.apk
```

#### 4. ثبّته:
```bash
termux-open ~/KhayrityApp/app/build/outputs/apk/debug/app-debug.apk
```

📖 **التفاصيل الكاملة في:** `TERMINUX_GUIDE.md`

---

## ☁️ الحل 3: خدمات بناء سحابية بديلة

نفس فكرة GitHub Actions لكن بمنصات أخرى:

| الخدمة | مجاني؟ | سهولة |
|---|---|---|
| **GitLab CI** | ✅ 400 دقيقة/شهر | ⭐⭐⭐⭐ |
| **Bitrise** | ✅ 2000 دقيقة/شهر | ⭐⭐⭐⭐⭐ |
| **Codemagic** | ✅ 500 دقيقة/شهر | ⭐⭐⭐⭐⭐ |
| **CircleCI** | ✅ 6000 دقيقة/شهر | ⭐⭐⭐ |

كلها تشتغل بنفس المبدأ: ارفع الكود → يبني → نزّل APK.

---

## 🌐 الحل 4: معاينة سريعة على المتصفح (بدون APK)

> **تنبيه:** هذا يعطيك محاكي على المتصفح، **ما يثبّت APK على تابلتك**.
> مفيد فقط لمعاينة التصميم بدون بناء فعلي.

1. أبني الـ APK من أي حل فوق
2. ارفعه على **Appetize.io** (https://appetize.io)
3. يفتح التطبيق في متصفحك

---

## 📊 مقارنة سريعة

| | GitHub Actions | Termux | Cloud CI | Appetize |
|---|---|---|---|---|
| **سهولة** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **سرعة** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **APK حقيقي** | ✅ | ✅ | ✅ | ❌ |
| **تثبيت على تابلتك** | ✅ | ✅ | ✅ | ❌ |
| **بدون حساب** | ❌ | ✅ | ❌ | ❌ |

---

## 🎯 توصيتي

### "أبي أجرب بأسرع وقت"
→ **الحل 1: GitHub Actions** (10 دقائق، مجاني)

### "تابلتي 4GB+ وأبي كل شي على تابلتي"
→ **الحل 2: Termux** (30-60 دقيقة، بدون رفع على الإنترنت)

### "أبي جرّب التصميم بسرعة بدون بناء"
→ **الحل 4: Appetize.io** (بعد ما يبني أحد APK)

---

## 🆘 مساعدة

| المشكلة | الحل |
|---|---|
| "ما عندي حساب GitHub" | سجّل من المتصفح في 30 ثانية |
| "تابلتي بطيئة" | استخدم GitHub Actions بدل Termux |
| "ما عندي نت كثير" | GitHub Actions يستهلك ~500MB فقط |
| "ما أعرف git/GitHub" | استخدم رفع الملفات من المتصفح |
| "APK ما يتثبت" | فعّل "مصادر غير معروفة" من الإعدادات |

---

جاهز؟ ابدأ بـ **الحل 1** وستجرب التطبيق خلال 10 دقائق! 🚀
