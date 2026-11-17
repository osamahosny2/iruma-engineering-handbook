# إعداد مفتاح توقيع Android ثابت — 2026-11-17

## سبب التغيير
كان إعداد Gradle يربط توقيع release بمتغيرات بيئة غير متوفرة في CI، بينما `debug` كان يعتمد على مفتاح runner الافتراضي الذي قد يتغير بين تشغيلات GitHub Actions. لذلك لم يكن هناك ضمان لاستمرارية توقيع APK المنشور.

## الإعداد الجديد
- صيغة keystore: PKCS12.
- الخوارزمية: RSA 4096-bit.
- الشهادة: self-signed، صالحة لمدة 10 سنوات من تاريخ الإنشاء.
- Alias: `iruma-upload-2026`.
- ملف المفتاح وكلمات المرور لا تُحفظ في Git.
- GitHub Actions secrets: `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`.
- Workflow يستخرج keystore مؤقتًا إلى `app/iruma-upload-key.p12`، ويستخدمه لتوقيع `debug` و`release` عند توفره، ثم يحذف الملف.
- إذا لم تتوفر أسرار CI، يستمر البناء المحلي debug باستخدام توقيع Android debug الافتراضي.

## إدارة المفتاح
- لا تغيّر هذه الأسرار ولا تولّد مفتاحًا جديدًا للإصدارات اللاحقة.
- لا تضع keystore أو كلمات المرور في المستودع أو في سجل CI.
- احتفظ بنسخة احتياطية آمنة خارج GitHub؛ فقدان المفتاح يعني عدم إمكانية إصدار تحديث متوافق مع التثبيتات الموقعة به.
- لا يمكن استخراج المفتاح الخاص من APK أو من الشهادة العامة.

## إصدار التطبيق
- `versionName=1.5.0`
- `versionCode=10500`
- commit: `4d9079a220c88b27019f8fdcb4185e35b7739137`
- branch: `fix/room-storage`
- CI run: https://github.com/osamahosny2/iruma_manga_v2/actions/runs/44110140411

## التحقق المطلوب
1. نجاح اختبارات Gradle.
2. نجاح `assembleDebug` باستخدام PKCS12 الجديد.
3. استخراج شهادة APK الناتج والتحقق من بصمتها.
4. عدم نشر Release حتى اكتمال بقية نطاق العمل وموافقة التحقق النهائي.
