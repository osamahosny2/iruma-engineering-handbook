# سجل القرارات الهندسية

هذا الملف يسجل القرارات المؤكدة، وليس الأفكار التي لم يوافق عليها المستخدم بعد.

## قرارات معتمدة

### 1. مستودع التطبيق منفصل عن مستودع المعرفة
- كود التطبيق والمحللات التنفيذية داخل `iruma_manga_v2`.
- المعرفة والمنهج والقرارات والأبحاث داخل `iruma-engineering-handbook`.
- السبب: التطبيق يجب أن يعمل دون تنزيل محللاته من مستودع GitHub وقت التشغيل، مع بقاء المعرفة قابلة لإعادة الاستخدام من الذكاء الاصطناعي والمساهمين.

### 2. المستودع القديم ممنوع التعديل
- العمل على `osamahosny2/iruma_manga_v2` فقط.
- الفرع المسموح `fix/room-storage`.
- `iruma_manga` القديم خارج النطاق.

### 3. المحللات مستقلة لكل موقع وتستخدم عقدًا موحدًا
- كل موقع له Parser خاص به.
- المحرك ينسق واجهة موحدة للبحث والتفاصيل والفصول والصفحات.
- لا يسمح بفشل Parser واحد أن يوقف بقية المصادر.

### 4. لا بيانات وهمية لإخفاء فشل المصدر
- لا Placeholder chapters كأنها فصول حقيقية.
- لا fallback صامت إلى مصدر آخر.
- لا اعتبار الغلاف صفحة فصل.

### 5. المهمة 3 prerequisite
- محرك المصادر يجب أن ينتج فصولًا وصفحات حقيقية قبل الاعتماد عليه في إعادة بناء القارئ.
- لا تؤجل المهمة 3 إلى نهاية الخطة.

### 6. الاختبار الآلي ليس اختبارًا عمليًا
- نجاح GitHub Actions لا يثبت عمل المصدر على هاتف المستخدم.
- لا تعلن اكتمال المهمة قبل تحقق معاييرها.

### 7. الموافقة لكل مهمة
- يجب جمع المتطلبات وشرح النطاق وانتظار إذن صريح قبل كل مهمة برمجية جديدة.
- الموافقة السابقة لا تعتبر إذنًا لمهمة أخرى أو توسعة نطاق.

### 8. تنظيم المحللات داخل مستودع التطبيق
- كل موقع له Parser مستقل في ملف مستقل.
- جميع ملفات المحللات تكون مجمعة في مجلد/Module واحد داخل مستودع `iruma_manga_v2`، مع سجل أو مدير مركزي يجمعها ويوفر واجهة موحدة للتطبيق.
- لا يحتاج التطبيق إلى تنزيل المحللات من GitHub وقت التشغيل.
- مستودع المعرفة يوثق التصميم والبحث ولا يحتوي على كود التشغيل.
- لا Fork لمشاريع Kotatsu ولا تعاون معها؛ يمكن دراسة الكود العام، ويمكن نقل كود مناسب مباشرة عند الحاجة بشرط الالتزام بترخيصه وإشعاراته.
- هذا القرار لا يمنح إذنًا تلقائيًا بتعديل كود التطبيق؛ يجب عرض خطة المهمة وانتظار موافقة صريحة قبل التنفيذ.

### 9. الوحدة المحلية للمحللات (تنفيذ تدريجي)
- توجد وحدة Gradle محلية `:source-parsers` داخل مستودع التطبيق.
- كل موقع يعاد بناؤه يكون له Parser مستقل في ملف خاص، والسجل `LocalSourceParserRegistry` يجمع المحللات المحلية.
- خلال الانتقال، يسمح للوحدة المحلية باستخدام مكتبة Kotatsu كاعتماد `api` للعقود وBaseParser للمصادر التي لم تُنقل بعد؛ التطبيق يعتمد على الوحدة المحلية وليس على المكتبة مباشرة.
- لا تنزيل للمحللات من GitHub وقت التشغيل.
- لا يعتبر وجود Parser محلي لموقع واحد اكتمال نقل بقية المواقع.

### 10. المصادر غير مفعّلة افتراضيًا
- كل مصادر الكتالوج تبدأ disabled.
- صفحة الإعدادات تعرض الكتالوج حتى يختار المستخدم المصادر التي يريد تفعيلها؛ قائمة التصفح لا تعرض إلا المصادر المفعّلة.
- الترقية من النسخة التي كانت تفعّل خمسة مصادر تلقائيًا تعطل هذه القيم مرة واحدة، ثم تحفظ اختيار المستخدم لاحقًا.

### 11. Apache-2.0 ProComic source adaptation
- The upstream ProComic extension in `keiyoushi/extensions-source` is Apache License 2.0; the app is GPL-3.0, which is compatible with Apache-2.0.
- Any adapted code must retain upstream attribution and a copy of the Apache-2.0 license in the app repository.
- Direct copying/adaptation is allowed where useful; no fork, shared development, or write access to the upstream repository is authorized.
- The adapted parser and reader remain owned/maintained in `iruma_manga_v2`.

### 12. ProComic Turnstile token and reader check handling
- Use the upstream ProComic Turnstile site key and an app-owned WebView token flow; do not bypass or automate away human interaction.
- Show a dialog only when the Turnstile widget requests interaction; if the app is not foregrounded, fail clearly instead of pretending verification succeeded.
- Cache tokens in memory for four minutes and send them only as the site's `x-turnstile-token` request header. Retry a 403 once after a rate-limited token refresh, then use the existing browser verification UI.
- Reader security-check rows are surfaced to the user through the existing in-app browser verification screen.

### 13. تحديث مستودع التعليم مع كل تعديل في التطبيق
- كل تعديل في تطبيق ايروما منجا، حتى لو كان صغيرًا جدًا أو اقتصر على سطر واحد، يجب أن يقابله تحديث في مستودع `iruma-engineering-handbook` في نفس جولة العمل.
- لا يؤجل التوثيق إلى نهاية المهمة أو المحادثة.
- حدّث الملفات المرتبطة بالتغيير: السياق، حالة المشروع، القرارات، الاختبارات، وثائق التنفيذ أو البحث، وسجل التغييرات حسب الحاجة.
- سجّل commit التطبيق والفرع والاختبارات ونتائجها وما لم يُختبر متى توفرت.
- الإذن الدائم بتحديث مستودع التعليم لا يمنح إذنًا بتعديل كود التطبيق خارج المهمة التي وافق عليها المستخدم.

### 14. ترتيب المهام المعتمد في 2026-11-16
- تُراجع وتُصلح جميع المسائل التابعة للمهمة 1 أولًا، ثم المهمة 2، ثم المهمة 3.
- لا يبدأ العمل المستقل على المهمة 4 قبل التحقق من المهام 1–3.
- بعد ذلك يكون الترتيب: 4 ثم 5 ثم 6 ثم 7 ثم 8.
- تظل الاختبارات المناسبة مطلوبة مع كل تعديل؛ ولا يعني ذلك بدء المهمة 8 مبكرًا.
- التفاصيل والنتائج العملية في `docs/PROJECT_STATUS_2026-11-16.md` و`docs/USER_APPROVED_WORK_ORDER_2026-11-16.md`.

## أفكار غير محسومة
أي فكرة جديدة لم يوافق عليها المستخدم تُكتب هنا تحت هذا العنوان مع وسم **مقترح فقط**، ولا تعامل كقرار معتمد.


### 15. مراجع نظام القراءة والحسابات
- Tachiyomi مرجع برمجي مفتوح المصدر لدراسة حالة الفصل وسجل القراءة وآليات التخزين؛ يجب الاستناد إلى الكود الفعلي لا إلى وصف عام.
- قارئ مانجا عربي (Hamako)، الحزمة `org.hamak.mangareader`، مرجع للميزات والسلوك وتجربة الاستخدام، خصوصًا سجل القراءة وحالة مقروء/غير مقروء. لا يوجد حتى الآن مستودع مصدر عام موثوق عُثر عليه للتطبيق، لذلك لا تُفترض تفاصيل تنفيذه الداخلية ولا يُنسخ منه كود.
- Hikari Manga مستبعد من مراجع المشروع، ولا يُستخدم مرجعًا برمجيًا أو مرجع ميزات.
- لا يعتمد تصميم سجل القراءة قبل مقارنة سلوك Tachiyomi الفعلي مع سلوك قارئ مانجا عربي عمليًا، ثم عرض التصميم الناتج على المستخدم للموافقة.
- نظام الحسابات والمزامنة في قارئ مانجا عربي مؤجل إلى ما بعد إكمال المهام الثماني؛ لا بحث تصميمي تفصيلي ولا تنفيذ له قبل ذلك.


### 16. مرجع نظام القراءة المعتمد مؤقتًا للمهمة 1 — 2026-11-16
- قرر المستخدم اعتماد Tachiyomi وحده مرجعًا حاليًا لتصميم حالة الفصل وسجل القراءة؛ تعذر عليه إرسال صور/فيديوهات لشرح سلوك Hamako.
- مقارنة قارئ مانجا عربي (Hamako) مؤجلة، ولا تمنع تنفيذ التصميم المستند إلى Tachiyomi.
- يعتمد Room v3 جدولًا مستقلًا لحالة الفصل وجدولًا لسجل القراءة، وهوية مركبة `(mangaId, chapterId)`، ولا حذف تلقائي للسجل.
- يجب الحفاظ على جميع بيانات v2 في Migration، مع اختبارات ترحيل وقاعدة ملفية واحتفاظ يتجاوز 100 سجل.


### 17. الاحتفاظ بتقارير الاختبارات عند الفشل — 2026-11-16
- بعد ظهور `AppDatabaseTest > initializationError` في Workflow 103، أضيف رفع JUnit XML reports دائمًا حتى تتوفر رسالة الخطأ الكاملة قبل تعديل صنف الاختبار.
- لا تُعتبر نتائج CI فاشلة بسبب JitPack أو بسبب تهيئة JUnit دليلًا على خلل migration نفسه؛ يجب تشخيص سبب الفشل الفعلي ثم إعادة الاختبار.


### 18. توقيع اختبارات JUnit — 2026-11-16
- اختبارات JUnit 4 المكتوبة بـKotlin يجب أن تعيد `Unit`؛ يجب ألا يكون آخر تعبير في `runBlocking` قيمة مثل `Boolean` من `deleteDatabase`.


### 19. عدم جعل تقارير الاختبارات الاختيارية شرطًا لنجاح البناء — 2026-11-16
- لا يجوز لرفع JUnit artifacts أن يمنع APK build عند امتلاء GitHub Actions artifact storage quota؛ أزيلت خطوة الرفع المؤقتة بعد تشخيصها.
- النتيجة الفعلية لاختبارات Workflow 105: `BUILD SUCCESSFUL in 3m 5s`؛ لكن البناء وschema snapshot لم ينفذا بسبب فشل خطوة artifact اللاحقة.


### 20. إصدار التطبيق بعد إصلاح Room — 2026-11-16
- الإصدار المطلوب بعد المهمة 1 هو `1.4.1` مع `versionCode=10401`، ولا يتغير رقم الإصدار أثناء الاختبارات السابقة لتجهيز النسخة النهائية.
- لا يُحذف Release `v1.4.0` قبل التحقق من Release `v1.4.1` ووجود APK قابل للتنزيل.


### 21. اكتمال Room v3 وإصدار 1.4.1 — 2026-11-16
- Workflow `43920986069` نجح بالكامل، وتم نشر Release خاص `v1.4.1` بعد اختبارات الوحدة والبناء.
- Room schema v3 snapshot ملتزم في `app/schemas/com.example.data.AppDatabase/3.json` ضمن commit `38819eb4034059142340a18c4f9432def35ee600`.
- حُذف Release `v1.4.0` بعد التحقق من APK الخاص بـ`v1.4.1`.
- التنفيذ البرمجي للمهمة 1 مكتمل؛ اختبار ترقية قاعدة بيانات الهاتف الفعلية ما زال مطلوبًا قبل اعتبار التحقق الميداني كاملًا.

### 22. Stable Android APK signing key and version 1.5.0 — 2026-11-17
- The user confirmed that the old 1.4.0 installation had been removed and approved replacing the unavailable 1.4.1 signing key with a new permanent key.
- A new RSA-4096 PKCS12 upload keystore was generated; its private material and passwords are stored only as encrypted GitHub Actions repository secrets in `osamahosny2/iruma_manga_v2` (`ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`). Secret values must never be committed, printed, or copied into this handbook.
- CI materializes the keystore only in the runner workspace, signs the debug APK with the stable key, then removes the temporary keystore file. Local builds without CI secrets retain the normal Android debug signing behavior.
- All future distributed APKs must use this same key. Losing or replacing it would prevent Android from accepting future in-place updates; maintain a secure backup outside the repository.
- Version target is `versionName=1.5.0`, `versionCode=10500`. Application commit: `4d9079a220c88b27019f8fdcb4185e35b7739137` on `fix/room-storage`.
- CI run `44110140411` is the validation for signing and build configuration; record its final outcome and APK signature only after the run completes.
