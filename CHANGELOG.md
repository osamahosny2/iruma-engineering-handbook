# سجل تغييرات مستودع المعرفة

## 2026-11-16 — تحديث قاعدة التوثيق وترتيب المهام ونتائج الهاتف
- اعتماد قاعدة إلزامية: تحديث مستودع التعليم مع كل تعديل في التطبيق مهما كان صغيرًا، وفي نفس جولة العمل.
- توضيح أن الإذن الدائم بتحديث مستودع التعليم لا يوسّع صلاحية تعديل كود التطبيق.
- توثيق ترتيب العمل: مراجعة وإصلاح المهام 1 ثم 2 ثم 3 والتحقق منها، ثم 4 ثم 5 ثم 6 ثم 7 ثم 8.
- تصحيح وصف الاختبار العملي: المستخدم أكد ظهور أعمال وفصول وصفحات حقيقية في المصادر التي جربها؛ نتائج البحث ظهرت لكن مطابقة العمل المقصود لم تُحسم.
- تسجيل MANGA Plus كمصدر يعمل، وAsuraScansGg وAsuraScans.us كمصدرين موجودين لكن يتعذر الاتصال بهما، وMangaDex وWebtoon وExHentai كمصادر غير موجودة في كتالوج التطبيق حسب اختبار المستخدم.
- توثيق أن مصادر 18+ مخفية افتراضيًا مع زر إظهار، ولا يلزم تعديل هذا السلوك.
- توثيق أن تداخل المصادر غير مؤكد في الإصدار الحالي ويظل نقطة مراقبة.
- توثيق قرار جعل اختبار CAPTCHA يدويًا بزر داخل صفحة كل مصدر محمي، وإلغاء ظهوره التلقائي عند فتح التطبيق.
- توضيح حدود المسؤولية: البحث وParser ضمن المهمة 3؛ إدارة التنزيلات وقائمة التنزيلات والعمل دون اتصال ضمن المهمة 6.

## 2026-11-10 — إنشاء المستودع
- إنشاء الهيكل الأولي لدليل هندسة ايروما منجا.
- إضافة تعليمات الذكاء الاصطناعي.
- فصل منهج `/برمجة` عن قواعد التعامل مع المستخدم.
- توثيق دورة تحليل وتعلم المعرفة البرمجية دون ادعاء تجربة بشرية.
- توثيق سياق المشروع ومعماريته ومحرك المصادر.
- إضافة خطة المهام الثمانية وحالة الإنجاز المعروفة.
- توثيق معايير الاختبار وسجل القرارات وخطة دراسة Kotatsu-Redo.
- توضيح أن الدراسة العميقة لـ Kotatsu-Redo لا تزال بحاجة إلى فحص ملفات المصدر الفعلية.

## 2026-11-10 — دراسة محرك المصادر
- فحص قراءة فقط لفرع `fix/room-storage` في التطبيق ومكتبة `kotatsu-parsers-redo` على commit المثبت في Gradle.
- توثيق أن المحللات حاليًا اعتماد Gradle خارجي وليست ملفات محلية داخل مستودع التطبيق.
- دراسة واجهة Parser وآلية KSP لتوليد سجل المصادر ومصنع المحللات.
- مراجعة محللات Azoramoon وTeamXNovel وMangaStarz، ومسار البحث والتفاصيل والفصول والصفحات في التطبيق.
- توثيق فرضيات الأسباب الجذرية ومخاطر نقل المكتبة إلى Module محلي، بما فيها اختلاف إصدارات KSP وبعض التبعيات.
- توثيق اختلاف commit بين اعتماد Gradle و`THIRD_PARTY_NOTICES.md` ومراجعة GPL-3.0.
- لم يتم تعديل كود التطبيق أو مستودعات Kotatsu؛ المهمة 3 ما زالت قيد البحث وغير مكتملة.
- إضافة التقرير التفصيلي `docs/SOURCE_ENGINE_RESEARCH_2026-11-10.md`.

## 2026-11-10 — تنفيذ أولي للمهمة 3
- commit `ad629006`: تعطيل المصادر افتراضيًا وترحيل التفعيل التلقائي القديم، الحفاظ على بيانات الفصل الأصلية، عدم إخفاء null chapter lists، وحماية من نتائج طلبات قديمة؛ إضافة اختبارات.
- CI: اختبارات الوحدة وبناء APK نجحا، لكن رفع artifact فشل بسبب امتلاء حصة GitHub Actions.
- commit `31636df`: إنشاء Gradle module محلي `:source-parsers`، سجل مركزي، محلل Azora محلي واختبارات HTML fixtures، وتحديث إشعار نسخة GPL.
- CI للـ commit الثاني قيد التنفيذ عند آخر تحقق.
- توثيق مشكلات Manga Pro الحالية وخطته في `docs/SOURCE_ENGINE_IMPLEMENTATION_2026-11-10.md`.

## 2026-11-10 — توثيق بحث ProComic
- أُنشئ بحث تكامل ProComic؛ التفاصيل الفنية وسجل التنفيذ في `PROCOMIC_READER_RESEARCH.md` (المرجع الأساسي)، وليس في سجل التغييرات.

## 2026-11-10 — تصحيح بروتوكول قارئ Manga Pro
- مراجعة ملف upstream slot.js بدل الاعتماد على DOM selector تخميني.
- ضبط البحث النصي على latest_chapter وحجم صفحة 12 نتيجة، مع الإبقاء على rate limit.
- توثيق استخدام جلسة WebView الدائمة الحالية لتتبع lazy slots وحفظ bytes الصور، بما في ذلك الصفحات المركبة من قطع متعددة.
- commit التطبيق: d560fc54eff633f89d171c51c78dcbc7fbbc5ecd.
- CI run 42896444353 قيد التنفيذ عند تسجيل هذا التحديث؛ لا يوجد تأكيد بعد على نجاح البناء أو اختبار الهاتف.

## 2026-11-10 — ProComic compile and session hardening
- Added focused tests for ProComic chapter DTO parsing and identity/slot behavior.
- Added safe_browsing=off cookie preparation and reused browser cookies for API and image requests.
- Corrected RectF width()/height() calls after CI identified the compile failure.
- Latest app commit: 8db92b23a10ec010a65543069a6cb80f4e0bc142.
- Latest CI run: 42897158863, in progress at the time of this entry.

## 2026-11-11 — ProComic Turnstile/reader follow-up `f2d86395`
- Added Turnstile token acquisition, human-interaction UI, four-minute token cache, rate-limited 403 retry and browser-verification fallback.
- Added readable-result pagination and reader security-check handling.
- Expanded ProComic Robolectric coverage to six tests and documented Apache-2.0 TurnstileHelper attribution.
- CI run 90 pending; previous run 89 passed tests and APK assembly but artifact upload hit the GitHub Actions storage quota.

## 2026-11-12 — Azoramoon API and Manga Pro review
- Updated local Azoramoon details/chapter/page extraction to the current JSON API.
- Added a shared novel classifier and chapter-title/number mapping regression tests.
- Verified default-disabled source activation and settings-only manual enablement.
- Recorded durable restoration of the original parser chapter object after app restart.
- Reviewed Manga Pro's JSON API and persistent WebView lazy-reader pipeline; documented that its custom connector is still outside the local parser module.
- App commit `8041716bcc3919f34234de33fd3ab7e398059501`; CI run 94 `43242033999` was in progress. No live/phone verification; task 3 remains open.

## 2026-11-12 — تصحيح التفعيل واستكمال مراجعة المصدرين الأولين
- commit `8041716bcc3919f34234de33fd3ab7e398059501`: تحسين تصنيف الروايات في Azora وحفظ عنوان الفصل مع رقمه، مع اختبارات إضافية.
- commit `97e2992e8c9dd35b4c7b2bc804e61ab18bb66c27`: عند تهيئة الكتالوج لأول مرة، كتابة حالة تعطيل صريحة لكل مصدر حتى لا تبقى سجلات `enabled=true` قديمة.
- commit `c9347644cd1b145c53de10eaef3467206f92f8d7`: رفع marker إلى v3 لإجبار الأجهزة التي سبق أن طبقت التهيئة v2 المعيبة على تنفيذ التصحيح مرة واحدة.
- تم تأكيد أن Settings تعرض الكتالوج للتفعيل اليدوي، وأن Browse يعرض المصادر المفعّلة فقط.
- مراجعة Manga Pro أثبتت من الكود أن صفحات Reader تعتمد على lazy slots داخل WebView وتُحل إلى صور مباشرة أو صفحات مركبة. تم تسجيل فجوة تنزيل الـvirtual URI ضمن المهمة 6.
- CI run `43242711623` (run 96) للـHEAD الحالي كان قيد التنفيذ عند آخر تحقق؛ لم يُسجل نجاح نهائي أو اختبار هاتف.

## النتيجة النهائية لـ CI run 96 — 2026-11-12
- workflow run: `43242711623` للـHEAD `c9347644cd1b145c53de10eaef3467206f92f8d7`.
- خطوة `Run unit tests`: **نجحت**.
- خطوة `Build Debug APK`: **نجحت**؛ Gradle أكد `BUILD SUCCESSFUL` ووجد workflow ملف APK واحدًا في المسار المتوقع.
- خطوة `Upload APK`: **فشلت فقط بسبب GitHub Actions artifact storage quota** (`Failed to CreateArtifact: Artifact storage quota has been hit`).
- لا يوجد اختبار حي على Azora/Manga Pro ولا اختبار يدوي على الهاتف في ذلك التاريخ؛ لذلك المهمة 3 ظلت قيد التنفيذ.

## 2026-11-12 — Manga Pro offline download fix
- Route DownloadManager page bytes through the source-aware handler, preserving ProComic WebView virtual reader URLs and page headers.
- Add Robolectric regression coverage for the Manga Pro offline page downloader.
- Use the same Azoramoon novel classifier in list/search and details.
- Commit `8ce70650772a4ad6a50a522ce5a813fda58074b0`; run 97 pending. Previous run 96 passed unit tests and APK build but artifact upload failed due to GitHub Actions storage quota.


## 2026-11-16 — اعتماد مراجع نظام القراءة وتأجيل الحسابات
- تثبيت Tachiyomi كمرجع برمجي لدراسة سجل القراءة وحالة الفصل.
- اعتماد قارئ مانجا عربي (Hamako) مرجعًا للميزات والسلوك وتجربة الاستخدام، مع توضيح عدم توفر مصدر عام موثوق عُثر عليه وعدم افتراض تفاصيل تنفيذه الداخلية.
- استبعاد Hikari Manga من مراجع المشروع.
- تأجيل نظام الحسابات والمزامنة إلى ما بعد إكمال المهام الثماني.
- اشتراط مقارنة السلوك الفعلي للمرجعين قبل اقتراح تصميم سجل القراءة، ثم موافقة المستخدم قبل التنفيذ.

## 2026-11-21 — Source engine first slice
- إصلاح أولوية المحللات المحلية بحيث يمكنها العمل بدل Parser upstream معلّم `@Broken`.
- توحيد إنشاء Parser وتخزينه المؤقت، ومنع المطابقة حسب اسم العرض لتفادي الخلط بين مصدرين لهما الاسم نفسه.
- تصحيح تصنيف DivaScans إلى 18+ في كتالوج التطبيق وفق تأكيد المستخدم.
- إضافة اختبارات وحدات لسياسة الاختيار والهوية والتصنيف. CI run `44849756102` قيد التنفيذ عند كتابة هذا السجل.


## 2026-11-25 — توثيق حالة التطبيق وتصحيح قواعد المستودع التعليمي
- تمت مراجعة المستودع التعليمي كاملًا وجمع قرارات المستخدم بشأن التعارضات والمعلومات القديمة والتكرار والتصنيف.
- أُلغي شرط مقارنة Hamako؛ Tachiyomi هو المرجع الحالي الوحيد لسلوك سجل القراءة.
- تم توحيد فهم تقارير JUnit: تقارير تشخيصية اختيارية، ولا يجوز أن تمنع بناء APK أو نجاح CI الأساسي بسبب حصة التخزين.
- تم توضيح أن مرجعي TurnstileHelper عند `d9420cd` و`d0478bb` يشيران إلى نسختين متطابقتين من الملف نفسه، مع بصمة SHA-256 موثقة في `KOTATSU_RESEARCH.md`.
- تم تحديث حالة التطبيق من المستودع الحي: `fix/hijala-parser` / `ffe3e9c5...`، إصدار `1.5.1`، CI #126 و#127 ناجحان، وPrivate Test Pre-release منشور. المهمة 3 لا تزال تنتظر اختبار المستخدم على الهاتف.
- تم توثيق تصحيح المستخدم لاسم Elfoon: لا يُعامل كموقع مستقل أو مصدر مؤكد.
- لا تعديل على كود التطبيق في هذه الجولة؛ التغييرات هنا توثيقية فقط.


## 2026-11-25 — توثيق تحديثات التطبيق حتى 1.5.1
- Room v4/CookieJar: `0ddcec4a25439592fa62b61f588a49e14c0c347a`، ثم إصلاحات CI `8d71e9d0166d5b6350bfde86aa5386abd2849b40` و`66f6ef1b8e8ce5a78e9b65075f3866c0b61e802e`.
- Local parser override وHijala: `dfdfde17935986e4075340909d9dd8e91f69b6b4`، `aa8031d805`، `bd8f381fd4`، `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`.
- إعداد إصدار الاختبار 1.5.1: `ffe3e9c5dee75249a4f2a6dc29a1b6d9d9625a2f`؛ CI #126 و#127 نجحا، وRelease `v1.5.1` منشور. تفاصيل التحقق والـSHA في `PROJECT_STATUS.md` و`TESTING.md`.
- لا يعني هذا أن المهمة 3 اكتملت؛ اختبار الهاتف ما زال مطلوبًا.
