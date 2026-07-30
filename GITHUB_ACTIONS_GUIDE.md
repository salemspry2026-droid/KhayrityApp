# ☁️ دليل GitHub Actions — بناء APK سحابي

> **الأسهل والأسرع لمن عنده تابلت فقط.** يبني الـ APK في سيرفرات GitHub وينزّله على تابلتك مباشرة.

---

## 📌 الفكرة باختصار

```
أنت (تابلت)  →  ترفع المشروع على GitHub  →  GitHub يبني الـ APK تلقائياً
                                                    ↓
                                            تنزّل الـ APK على تابلتك
                                                    ↓
                                            تثبّته وتشغّله
```

**ما تحتاج تثبّت أي شي** على التابلت. كل البناء يصير في سيرفرات GitHub (Linux + 16GB RAM).

---

## الخطوة 1: إنشاء حساب GitHub (إذا ما عندك)

افتح https://github.com من **متصفح التابلت** وأنشئ حساب مجاني.

---

## الخطوة 2: إنشاء Repository جديد

من متصفح التابلت:

1. اضغط على `+` (أعلى يمين) → `New repository`
2. اسم الـ repo: `KhayrityApp` (أو أي اسم)
3. اختر `Public` (مجاني بالكامل) أو `Private` (مجاني أيضاً)
4. ❌ **لا تضف** README, .gitignore, license (موجودة بالمشروع)
5. اضغط `Create repository`

---

## الخطوة 3: نقل المشروع إلى GitHub

### الخيار A: من متصفح التابلت (الأسهل بدون Termux)

1. في صفحة الـ repo الجديدة، اضغط `uploading an existing file`
2. اسحب وأفلت **محتويات** مجلد `KhayrityApp/` (لا تضغط المجلد نفسه، اضغط محتواه)
3. اكتب commit message: "Initial commit"
4. اضغط `Commit changes`

> ⚠️ المشكلة: `gradle-wrapper.jar` و `gradlew` يجب أن تُرفع. إذا GitHub رفض `gradlew` (لأنه executable)، ارفعه من Termux (الخطوة 3B).

### الخيار B: من Termux (موصى به)

ثبّت Termux كما في `TERMINUX_GUIDE.md`، ثم:

```bash
pkg install -y git
cd ~/KhayrityApp  # أو حيث فكيت الضغط
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourname/KhayrityApp.git
git push -u origin main
```

> عند `git push`، سيطلب منك Username + Personal Access Token (كلمة سر).
> للحصول على Token: https://github.com/settings/tokens → Generate new token (classic) → اختر `repo` → Generate. استخدم الـ Token ككلمة سر.

---

## الخطوة 4: تشغيل البناء

بمجرد ما يكون المشروع على GitHub:

### تلقائياً:
> ⚡ البناء يبدأ تلقائياً عند كل push بسبب ملف `.github/workflows/build-apk.yml` المضمّن في المشروع.

### يدوياً:
1. افتح صفحة الـ repo على GitHub
2. اضغط على تبويب `Actions`
3. اضغط `Build Debug APK` من القائمة اليسرى
4. اضغط `Run workflow` → `Run workflow`
5. انتظر 5-10 دقائق

---

## الخطوة 5: تنزيل الـ APK

1. في تبويب `Actions`، اضغط على الـ run المكتمل (علامة ✅ خضراء)
2. انزل لأسفل قسم `Artifacts`
3. اضغط على `khayrity-debug-apk` لتنزيله
4. افتح الملف المحمّل (سيُطلب منك فتحه بـ Files)

---

## الخطوة 6: تثبيت APK على التابلت

1. افتح تطبيق **Files** (مدير الملفات) على التابلت
2. اذهب إلى `Downloads` (أو `تنزيلات`)
3. اضغط على `khayrity-debug-apk.zip` (سيكون مضغوط)
4. اضغط `Extract` أو `فك الضغط` → يعطيك `app-debug.apk`
5. اضغط على `app-debug.apk`
6. إذا طلب إذن "مصادر غير معروفة":
   - اضغط `Settings`
   - فعّل `Allow from this source`
   - ارجع واضغط Install
7. اضغط `Install` ثم `Open`
8. 🎉 التطبيق شغّال!

---

## 🔄 تحديث التطبيق (لو عدّلت الكود)

1. عدّل الملفات في GitHub (من المتصفح أو Termux)
2. الـ Actions يبني تلقائياً
3. نزّل الـ APK الجديد
4. ثبّته (Android يسألك: "تثبيت تحديث؟" → اضغط Update)

---

## 📱 بديل: رفع APK مباشرة للمتصفح

بعض تطبيقات المتصفح في Android تسمح بتنزيل APK وتشغيله:

- **Chrome** → بعد التنزيل، افتح الـ APK من notification bar
- **Kiwi Browser** → ممتاز لتنزيلات APK
- **Firefox** → اضغط على الملف المحمّل من القائمة

---

## 🆓 الحدود المجانية

- **Public repos:** 2000 دقيقة Actions/شهر (كافية جداً)
- **Private repos:** 500 دقيقة Actions/شهر
- بناء واحد = حوالي 5-7 دقائق → تقدر تبني 300+ مرة شهرياً مجاناً

---

## 🐛 إذا البناء فشل

1. افتح الـ run الفاشل في Actions
2. اضغط على `build` job
3. انزل وشوف الخطأ الأحمر
4. شائع:
   - **Out of memory:** أضف `-Xmx4g` (لديهم 16GB فما المفروض يحصل)
   - **Compile error:** غالباً خطأ في الكود — راسلني

---

## 💎 Bonus: APK مُوقّع (Release)

للنشر على Google Play أو المشاركة العامة:

1. أنشئ keystore (مرة واحدة):
   ```bash
   keytool -genkey -v -keystore release.keystore -alias khayrity -keyalg RSA -keysize 2048 -validity 10000
   ```

2. ارفعه كـ GitHub Secret:
   - Settings → Secrets and variables → Actions
   - أضف: `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_PASSWORD`, `KEY_ALIAS`

3. شغّل workflow "Release APK" من Actions

---

استمتع! 🚀
