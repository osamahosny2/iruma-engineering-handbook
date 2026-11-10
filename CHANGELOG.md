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


## 2026-11-10 — ProComic API/reader app commit `0220e115`

- Added live JSON API mapping for Manga Pro search, details and paginated Arabic chapter lists.
- Added persistent WebView reader sessions, real lazy slot enumeration, captured CDN image bytes, composite-page stitching and alternate reader-host fallback.
- Added five-second listing rate limiting and four Robolectric parser/reader-contract tests.
- Added Apache-2.0 license text and third-party attribution for the adapted ProComic implementation; no upstream fork or changes.
- CI run `42896328650` was in progress during this documentation update; live website and phone tests remain outstanding.
