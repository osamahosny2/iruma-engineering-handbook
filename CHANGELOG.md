# سجل تغييرات مستودع المعرفة

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

## 2026-11-10 — ProComic/Manga Pro reader reconstruction

- The previous HTML-only path was not a reliable source of real chapter data. Manga Pro now uses its public JSON API for search, content details and paginated chapter lists; the app must reject placeholder chapters and report an empty/invalid API response instead of inventing data.
- The reader path is being rebuilt around the site's actual lazy reader slots. A chapter page is not assumed to be one image: the slot can expose one CDN image or several positioned pieces that must be composed in order.
- A persistent WebView session is required because page slots load lazily and the site may only serve image requests in browser context. The app proxies the reader's image requests through its shared OkHttp client and retains captured bytes until the reader consumes them.
- ProComic and ProChan may move between `procomic.pro` and `procomic.net`; the reader must preserve the current host and retry the alternate host only when the chapter route fails. CDN image hosts remain under `*.procomic.im`.
- The site documents a minimum five-second spacing between listing API calls and uses `safe_browsing=off`. Later listing pages may additionally require a short-lived Turnstile token; this remains an explicit follow-up until implemented and tested.
- Upstream research was read from `keiyoushi/extensions-source`, especially `Procomic.kt`, `Dto.kt`, and `assets/slot.js` at commit `4591fbc9bb571ad03aaea0f7428f7f403350e90e`, plus `TurnstileHelper.kt` at `d9420cd1c66ecd3385946b616aacfcf8f6314f70`.
- The app is GPL-3.0 and the upstream ProComic extension is Apache-2.0; these licenses are compatible. The existing `THIRD_PARTY_NOTICES.md` already attributes the adapted files and `LICENSES/Apache-2.0.txt` retains the upstream license text. This is not a fork.
- Code commit `8ff18f1b20e1213ac03b2ae62e869d3782651134` introduced the API/reader path. CI run `42895363904` exposed a missing `java.io.IOException` import; that compile fix is commit `5211f99d58d4d7c28a26f2b48cdff1e5b2f020b5`. CI run `42896185574` is the verification run for the fix and was still running when this note was written.
- No phone-level validation has happened yet. The parser is not considered complete until a real Manga Pro work returns real Arabic chapters, the first/last page of a chapter displays real images, composite pages stitch correctly, and a downloaded chapter opens offline.


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
