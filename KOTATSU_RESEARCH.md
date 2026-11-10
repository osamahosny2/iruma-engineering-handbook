# دراسة Kotatsu-Redo وkotatsu-parsers-redo

## الهدف
فهم كيفية تنظيم مصادر القراءة واستخراج الأعمال والفصول والصفحات، ثم الاستفادة من المبادئ المناسبة في ايروما منجا دون نسخ غير مدروس.

## المستودعات محل الدراسة
- التطبيق: https://github.com/Kotatsu-Redo/Kotatsu-Redo
- المحللات: https://github.com/Kotatsu-Redo/kotatsu-parsers-redo

## الحالة الحالية
**الدراسة العميقة لم تُوثق على أنها مكتملة بعد.** توجد فرضية تصميمية عامة بأن فصل Parser كل موقع خلف واجهة موحدة طريقة مناسبة، لكن يجب تأكيد تفاصيل التنفيذ من ملفات المصدر والاختبارات الفعلية قبل اعتمادها كحقيقة عن Kotatsu-Redo.

## خطة الفحص
1. README وملفات البناء والإصدارات والتبعيات.
2. واجهة Parser الأساسية والعقود.
3. Parser حقيقي لموقع واحد: البحث ثم التفاصيل ثم الفصول ثم الصفحات.
4. نماذج الأعمال والفصول والصفحات والتحويلات.
5. HTTP client وHeaders وCookies وReferer وUser-Agent.
6. Pagination وإدارة الروابط.
7. Cloudflare أو WebView أو أي challenge coordinator إن وجد.
8. الاختبارات والعينات والـ fixtures.
9. كيفية تسجيل المصادر واكتشافها وتحديثها.
10. مقارنة التصميم بكود ايروما منجا وكتابة توصيات قابلة للتنفيذ.

## قالب توثيق كل نتيجة
- السؤال:
- المستودع والفرع/الـ commit:
- مسار الملف:
- الرمز/الواجهة ذات الصلة:
- ما يفعله الكود فعليًا:
- الاختبار الذي يؤكد ذلك:
- ما يمكن الاستفادة منه في ايروما منجا:
- ما لا ينبغي نقله:
- مستوى الثقة والقيود:

## ملفات في تطبيق ايروما منجا يجب مقارنتها
- `KotatsuSourceEngine.kt`
- `ConnectorEngine.kt`
- `ChapterPageExtractor.kt`
- `AndroidKotatsuWebViewExecutor.kt`
- `NetworkClientProvider.kt`
- `MangaViewModel.kt`
- `MangaModels.kt`
- `BrowseScreen.kt`
- `ReaderScreen.kt`
- `WebviewScreen.kt`

هذه قائمة ملفات من سجل فحص سابق؛ يجب التحقق من وجودها ومحتواها الحالي في الفرع قبل الاستناد إليها.

## سجل التحقق
لم يتم في إنشاء هذا الملف فتح وتحليل ملفات Kotatsu-Redo الفعلية. لا تكتب نتائج تفصيلية هنا إلا بعد تنفيذ خطة الفحص وتسجيل الأدلة.


## نتائج مؤكدة من ملفات المصدر — 2026-11-10

### مرجع النسخة
تم فحص مكتبة `kotatsu-parsers-redo` على commit `b3c000024563920d3eecde5816f2e9d3734fa467`، وهو نفس commit المثبت في `app/build.gradle.kts` بالتطبيق.

### واجهة التسجيل
- `MangaParser` يوفر عقدًا موحدًا للقائمة والتفاصيل والفصول والصفحات وروابط الصور ورؤوس الطلبات واعتراض HTTP.
- `@MangaSourceParser` Annotation ذات SOURCE retention.
- `ParserProcessor.kt` يستخدم KSP لتوليد `MangaParserSource` وسجل إنشاء Parser. التسجيل compile-time وليس runtime reflection.

### أمثلة محللات تمت قراءتها
- `site/ar/Azoramoon.kt`: API للبحث والتفاصيل والفصول وصور الصفحات؛ يدعم Pagination وحالات القفل.
- `site/ar/TeamXNovel.kt`: HTML selectors مع Pagination للفصول وتحميل صفحات متعددة بالتوازي.
- `site/madara/ar/MangaStarz.kt`: يرث `MadaraParser` ويحتفظ بإعدادات الموقع الخاصة؛ يوضح إمكانية مشاركة BaseParser بين مواقع CMS المتشابهة.

### الترخيص
ملف `LICENSE` عند النسخة المثبتة هو GPL-3.0. النقل المباشر يتطلب الالتزام بشروط الترخيص وتحديث إشعارات المصدر والنسخة.

### حدود الدراسة
تمت مراجعة ملفات العقد والـ KSP وبعض المحللات والاختبارات والبناء، لكن لم تكتمل بعد مراجعة كل Parser أو اختبار المواقع الحية. تفاصيل البناء والمقارنة مع التطبيق في [التقرير الكامل](docs/SOURCE_ENGINE_RESEARCH_2026-11-10.md).


## تطبيق نتائج البحث — 2026-11-10
- تم اعتماد Gradle module محلي `:source-parsers` بدل إبقاء الاعتماد الخارجي مباشرة داخل `:app`.
- المكتبة الخارجية تبقى حاليًا طبقة مشتركة للعقود وBaseParser؛ المحللات التي نعيد بنائها تنتقل إلى ملفات محلية مسجلة عبر `LocalSourceParserRegistry`. هذا نقل تدريجي وليس نسخًا كاملًا للمكتبة بعد.
- أول ملف محلي `IrumaAzoramoonParser.kt` مقتبس/مكيّف من `site/ar/Azoramoon.kt` على commit `b3c000024563920d3eecde5816f2e9d3734fa467`، مع إشعار GPL-3.0.
- اكتُشف في upstream Azoramoon أن فقدان محدد `astro-island` كان يتحول إلى قائمة فصول فارغة، وأن استجابة API غير الصالحة كانت تتحول إلى نتائج فارغة؛ النسخة المحلية لا تخفي هذين الفشلين.
- `Manga Pro` ليس محللًا من مكتبة Kotatsu في هذه الدراسة؛ مرجع قارئه الأحدث من `keiyoushi/extensions-source` يستخدم API وقارئ WebView slots، وسيتم بناء تكامل محلي مناسب بدل افتراض أن HTML الثابت كافٍ.

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
