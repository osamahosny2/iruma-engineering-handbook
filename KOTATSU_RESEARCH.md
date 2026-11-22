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


## ProComic source distinction
The current Manga Pro/ProComic parser reference is the separate `keiyoushi/extensions-source` ProComic extension, not a Kotatsu-Redo parser. Its API DTOs, lazy reader slot script and Turnstile helper were studied as a separate upstream implementation. The app ports the needed behavior into its own connector/WebView engine; it does not fork Kotatsu-Redo or Keiyoushi, and it does not add an upstream runtime dependency for ProComic. Licensing and attribution are recorded in `PROCOMIC_READER_RESEARCH.md` and the app's `THIRD_PARTY_NOTICES.md`.


## Turnstile helper source
The ProComic Turnstile token flow is adapted from `keiyoushi/extensions-source` `core/src/main/kotlin/keiyoushi/utils/TurnstileHelper.kt` at commit `d0478bb62bbc485753d2830102aef4e1b87cb336` (Apache-2.0). This is not a Kotatsu-Redo component. The app contains its own adaptation and license attribution; no upstream runtime dependency or fork was added.

## دراسة فعلية مقارنة — 2026-11-21

### المراجع والإصدارات
- تمت مراجعة كود تطبيق Kotatsu الأصلي وكود Kotatsu-Redo، وليس README فقط.
- تمت مراجعة مكتبة `kotatsu-parsers-redo`، والعقد `MangaParser` و`MangaSourceParser` ونماذج `MangaChapter` و`MangaPage`، إضافة إلى `AbstractMangaParser` و`CloudFlareHelper` و`OkHttpWebClient`.
- التطبيق يثبت `kotatsu-parsers-redo` عند commit `68f3fed3952f5c9cf1ad8cc00bdb1ff047688dd8` في `source-parsers/build.gradle.kts`. لم يتم تغيير هذا الاعتماد أو ترقية نسخته خلال هذه الشريحة.

### طبقة التطبيق والشبكة
تمت مقارنة ملفات `MangaLoaderContextImpl` و`ParserMangaRepository` و`CloudFlareInterceptor` و`WebViewExecutor` في Kotatsu الأصلي وRedo. كما تمت مراجعة إضافات Redo ذات الصلة مثل `WebViewRequestInterceptorExecutor` و`CaptchaHandler`، ومزامنة CookieJar وCommonHeadersInterceptor. الاستفادة الأساسية هي الفصل بين سياق Parser، وHTTP client، وCookie/session state، وتنفيذ التحديات داخل WebView عند الحاجة؛ لا ينبغي نقل هذه المكونات كاملة أو تجاوز تحديات الحماية.

### عائلات المحللات المؤكدة من قائمة المستخدم
- Madara: عدة مواقع متشابهة يمكنها مشاركة BaseParser مع تخصيصات صغيرة لكل موقع.
- FoolSlide: له مسار مختلف لاستخراج القائمة والفصول وصفحات الصور من JSON مضمّن في الصفحة، وقد يتطلب قبول بوابة العمر.
- MangaReader: عائلة مستقلة لاستخراج القائمة والتفاصيل والفصول والصفحات، مع دعم صور/مصادر مشفرة في بعض المواقع.
- KeyoApp: بنية مختلفة للبحث والفصول واستخراج CDN URL.
- Manga18: مثال لاستخراج روابط الصفحات المشفرة من بيانات JavaScript.
- GalleryAdults: بعض المواقع تحتاج التنقل إلى صفحة كل صورة ثم استخراج `gimg` بدل افتراض أن رابط الفصل نفسه صورة.
- FuzzyDoodle: مثال آخر لقائمة وفصول متعددة الصفحات.
- DemonicScans وDivaScans وKDTScans ترث `PagedMangaParser` لتوحيد pagination، مع تنفيذ خاص لكل موقع؛ KDT يتعامل مع chunks مشفرة ورؤوس Referer، وDivaScans يستخدم API JSON.

### نتيجة مقارنة مهمة للتصميم
لا يوجد Parser عام واحد يصلح لكل المواقع. يجب أن يبقى العقد موحدًا (قائمة، تفاصيل، فصول، صفحات، وروابط صور)، بينما تختلف آلية استخراج البيانات وفق CMS/API الفعلي. المصادر التي لا تملك Parser لا يُنشأ لها دعم تخميني.

### تطبيق النتائج في إيروما منجا
أول إصلاح في 2026-11-21 جعل إنشاء المحلل المحلي يسبق رفض المصدر بسبب `@Broken`، وجعل هوية Parser تعتمد على ID ثابت بدل الاسم المعروض لتجنب التباس المصادر ذات الأسماء المتشابهة. لم يتم نسخ BaseParser أو WebView من upstream، ولم تتم ترقية المكتبة.

## تدقيق عائلات الوراثة والهدف العربي الأول — 2026-11-21

### عائلات إضافية تم التحقق منها عند commit المثبّت
- `PagedMangaParser` يوحّد pagination للقائمة والبحث ويترك `getListPage()` لكل موقع؛ تستخدمه DemonicScans وDivaScans وKDTScans.
- `IkenParser` (EzManga) يوفّر مسار API للقائمة `/api/query` والفصول `/api/chapters` مع fallback لاستخراج صور HTML عند الحاجة.
- `KeyoappParser` هو أساس KewnScans، وليس EzManga.

### Hijala (`HIJALA`)
- ملف `Hijala.kt` في commit `68f3fed...` يحمل `@Broken` صراحةً ويرث `ZeistMangaParser` مع النطاق `hijala.com`.
- `ZeistMangaParser` القديم يبني Blogger Feed من `/feeds/posts/default/-/...` ويقرأ `feed.entry`، بينما صفحات Hijala الحالية التي فُحصت سابقًا تبدو WordPress-like. لذلك لا يكفي إزالة `@Broken` أو إعادة استخدام selectors القديمة.
- جرى فحص REST paths العامة `/wp-json/` و`/wp-json/wp/v2/types` و`/wp-json/wp/v2/posts?per_page=1`؛ جميعها أعادت `bot_blocked`. لم تتم محاولة تجاوز Cloudflare أو استخدام stealth/proxy للتحايل.
- النتيجة: Hijala ما زال أول هدف عربي عام حسب ترتيب المهمة، لكن كتابة Parser محلي تتطلب DOM/HTML حاليًا من جلسة مستخدم يمكنها فتح الصفحة طبيعيًا.


## نتائج دراسة إضافية — 2026-11-22

### كود Kotatsu الأصلي وKotatsu-Redo
- تمت قراءة AndroidCookieJar وPreferencesCookieJar وNetworkModule وCloudFlareInterceptor وCaptchaHandler وCaptchaContinuationClient وWebViewExecutor وMangaLoaderContextImpl من Kotatsu الأصلي وKotatsu-Redo.
- AndroidCookieJar في المرجعين يستخدم CookieManager.getCookie(url) ويحوّل أزواج request-header إلى OkHttp Cookie. PreferencesCookieJar يخزن الكوكيز الدائمة في SharedPreferences، ويحتفظ بغير الدائمة في الذاكرة. NetworkModule يستخدم AndroidCookieJar عند توفر WebView وPreferencesCookieJar كبديل عند تعذر WebView؛ لا توجد مزامنة ثنائية الاتجاه كاملة بين الاثنين في المسار المعتاد.
- بعض مسارات Kotatsu تحاول حل CAPTCHA باستخدام WebView مخفي/آلي. هذا السلوك لا يُنقل إلى إيروما لأن المطلوب هو إتمام المستخدم للتحقق البشري داخل WebView، لا تجاوز التحدي.

### Mihon وCloudflare
- تمت مراجعة Mihon كمرجع Tachiyomi الحالي، بما في ذلك AndroidCookieJar وCloudflareInterceptor ودليل Troubleshooting.
- دليل Mihon يطلب فتح المصدر داخل WebView وإكمال CAPTCHA يدويًا، ويذكر أن بعض مستويات Cloudflare قد تستمر في إعادة التحميل. توثيق Cloudflare Turnstile يذكر ثبات User-Agent، وتمكين JavaScript وDOM storage والكوكيز، والسماح باتصالات challenges.cloudflare.com.
- توثيق Cloudflare يحدد cf-mitigated: challenge كإشارة موثوقة لاكتشاف استجابة Challenge Page.

### Android وOkHttp
- CookieManager.getCookie(url) يعيد أزواج key=value بتنسيق Cookie request-header فقط؛ لا يمكن استعادة كل خصائص Set-Cookie الأصلية منه.
- setCookie(url, value, callback) غير متزامنة ويعيد callback نتيجة النجاح، وflush() يكتب الكوكيز المتاحة عبر getCookie إلى التخزين الدائم.
- OkHttp CookieJar مسؤول عن سياسة قبول الكوكيز وتخزينها وإعادتها للطلب المناسب. يجب احترام host-only/domain وpath وSecure وانتهاء الصلاحية والاستبدال والحذف.
- لا يجوز ادعاء أن getCookie يسمح باسترجاع HttpOnly أو SameSite أو expiry أو Domain/Path الأصلية كاملة.

### GitHub Issues وReddit وX
- Mihon issue #2455 يوثق أن حذف Cookie يتطلب تطابق domain وpath الأصليين.
- Mihon issue #1722 يوثق أن استيراد Cookies من متصفح خارجي لم يكن كافيًا وحده لحل Cloudflare في حالة صاحب البلاغ.
- Keiyoushi issue #20975 يوثق حالة كان فيها WebView يفتح بينما يفشل القارئ الأصلي مع Cloudflare.
- Reddit يقدم تجارب فردية حول WebView وUser-Agent وإعادة تشغيل التطبيق؛ لا تُعامل هذه التجارب كضمان تقني.
- تمت محاولة البحث العام في X وفتح صفحة البحث، لكن لم تُسترجع منشورات X قابلة للتحقق. يظل هذا جزءًا غير مكتمل من البحث.

### أثر النتائج على التصميم
- المطلوب مخزن OkHttp داخلي مستقل ودائم، مع حفظ كامل خصائص Cookies القادمة من Set-Cookie متى وفرها OkHttp.
- بعد التحقق اليدوي في WebView، يمكن قراءة request-scoped cookie pairs من CookieManager، لكن يجب توثيق فقدان خصائصها وعدم الادعاء باستعادة البيانات الأصلية كاملة.
- يجب انتظار نتيجة callback عند نقل Cookies إلى WebView، ثم flush، ومنع التكرار والتسريب بين النطاقات.
- لا يعتمد التصميم النهائي على Room أو DataStore قبل فحص اعتماديات المشروع واختيار الأنسب لها.
- لا توجد تعديلات برمجية منفذة حتى الآن في جلسة 2026-11-22.


## 2026-11-22 — تدقيق التنفيذ الفعلي للكوكيز على fix/hijala-parser

### حالة Git والفروع
- تمت مقارنة `fix/room-storage...fix/hijala-parser`: النتيجة `ahead_by=3`, `behind_by=0`، أي أن الفرع الأحدث يحتوي تاريخ الفرع القديم كاملًا، ويضيف ثلاثة commits: `aa8031d` (إضافة Hijala parser)، `bd8f381` (اختبار التسجيل المحلي)، `83e033d` (خيارات فلترة Hijala). لا حاجة إلى merge من room-storage إلى hijala-parser.
- تمت مقارنة `main...fix/hijala-parser`: `ahead_by=101`, `behind_by=0`. لا تدمج إلى main أثناء المهمة؛ يبقى main خارج نطاق الكتابة.
- الفروع الحالية في المستودع: `fix/hijala-parser`, `fix/room-storage`, `main`.

### ملفات التطبيق التي فُحصت
- `app/src/main/java/com/example/network/AndroidCookieJar.kt`
- `app/src/main/java/com/example/network/NetworkClientProvider.kt`
- `app/src/main/java/com/example/connector/kotatsu/KotatsuSourceEngine.kt`
- `app/src/main/java/com/example/connector/kotatsu/AndroidKotatsuWebViewExecutor.kt`
- `app/src/main/java/com/example/ui/screens/WebviewScreen.kt`
- `app/src/main/java/com/example/connector/ProComicConnector.kt`
- `app/src/main/java/com/example/connector/ConnectorEngine.kt`
- `app/src/main/java/com/example/IrumaMangaApplication.kt` و`app/src/main/java/com/example/di/AppContainer.kt`

### عيوب التنفيذ الحالية المثبتة من الكود
1. `AndroidCookieJar.loadForRequest()` يقرأ `CookieManager.getCookie(url)` ثم يقسم النص إلى `name=value` ويستعمل `Cookie.parse(url, pair)`. ناتج WebView لا يحمل Domain/Path/expiry/HttpOnly/Secure/SameSite الأصلية؛ لذلك يعيد OkHttp بناء Cookies بنطاق المضيف ومسار افتراضي من URL، ولا يمكن اعتبار هذا استيرادًا كاملًا لخصائص WebView الأصلية.
2. `saveFromResponse()` يستدعي `CookieManager.setCookie()` دون callback أو انتظار تأكيد الكتابة، ثم ينفذ `flush()` مباشرة. يجب تنسيق الإكمال قبل الاعتماد على انتقال الجلسة إلى WebView.
3. `NetworkClientProvider` ينشئ `AndroidCookieJar()`، كما أن `KotatsuSourceEngine` ينشئ Jar آخر عبر `.newBuilder().cookieJar(AndroidCookieJar())`. يجب توحيد مصدر الحقيقة ومشاركة CookieJar واحد؛ اختلاف الكائنات الحالية لا يضمن مشاركة مخزن داخلي مستقبلي.
4. `ProComicConnector.kt` يضع Cookies يدويًا في Request headers في عدة مواضع، و`ConnectorEngine.kt` يفعل ذلك أيضًا. هذه المسارات تتجاوز إدارة CookieJar أو تخلطها بترويسات Cookie يدوية؛ يجب توحيدها مع الحفاظ على السلوك الخاص `safe_browsing=off` وعدم إسقاطه.
5. `IrumaMangaApplication.onCreate()` يهيئ WebView executor، بينما `AppContainer` ينشئ Room عند الطلب. يجب تهيئة CookieJar/قاعدة البيانات قبل أول طلب شبكة، لا بعده.

### بنية التخزين الموجودة
- `AppDatabase` حاليًا version 3 ويحتوي جداول المكتبة والتصنيفات والمصادر وحالة القراءة والتقدم.
- `Room` runtime/ktx/compiler مثبتة على 2.7.0، وOkHttp مثبت على 4.10.0 في `gradle/libs.versions.toml`.
- لا يوجد حاليًا جدول Cookie أو Cookie DAO أو اختبار مستقل لـCookieJar.
- يمكن إضافة Cookie table بترحيل `MIGRATION_3_4` دون إنشاء قاعدة ثانية، بشرط الحفاظ على بيانات المستخدم واختبار migration من schema 3.

### التصميم المقترح — لم يُنفذ ولم يعتمد بعد
- CookieJar واحد singleton مشترك بين NetworkClientProvider وKotatsuSourceEngine وكل طلبات ProComic/Coil/ConnectorEngine.
- Room version 4 وجدول Cookie منفصل بمفتاح فريد مركب من `name + domain + path` وحقول domain/hostOnly/path/expiresAt/secure/httpOnly/persistent وأي خصائص يدعمها إصدار OkHttp المثبت فعليًا. لا تفترض SameSite قبل التحقق من API في 4.10.0.
- تشفير قيمة كل Cookie عند التخزين باستخدام AES-GCM ومفتاح Android Keystore؛ لا تسجل القيم في Log ولا ترفعها إلى GitHub. تعامل مع فقد/إبطال المفتاح بحذف السجلات غير القابلة للفك وطلب تحقق يدوي جديد، لا بتسريب القيمة.
- لأن `okhttp3.CookieJar` واجهته synchronous، يجب أن تكون عمليات DAO المتزامنة محصورة في خيوط OkHttp الخلفية مع منع الاستدعاء من Main Thread؛ لا تستخدم `runBlocking` على Main ولا تكتب إلى DB بطريقة fire-and-forget إذا كان المطلوب ضمان الاستمرار بعد إغلاق العملية.
- عند `loadForRequest`: اجلب Cookies الداخلية غير المنتهية وطابق Domain/hostOnly/Path/Secure عبر قواعد OkHttp، ثم أضف snapshot من `CookieManager.getCookie(requestUrl)` بوصفه بيانات URL-scoped فقط. لا توسّع Cookie من WebView إلى parent domain لأن API لا يكشف Domain/Path الأصليين.
- عند `saveFromResponse`: احفظ Cookies الواردة من OkHttp مع خصائصها التي يدعمها OkHttp، واحذف Cookies المنتهية/المحذوفة، ثم جدْول مزامنتها إلى WebView. انتظر callback الخاص بـ`setCookie` ثم `flush()` قبل أن يبدأ WebView تنقلًا يعتمد على تلك القيم.
- بعد تحقق المستخدم في WebView: أبقِ CookieManager مصدرًا صالحًا للكوكيز التي لم يمكن استعادة نطاقها، واجعل OkHttp يقرأ snapshot حسب URL المطلوب؛ يمكن تخزين نسخة host-only للمضيف الحالي فقط إذا كان ذلك مفيدًا، مع توثيق أنها ليست نسخة كاملة من خصائص Cookie الأصلية.
- أزل حقن Cookie headers اليدوي من المسارات العامة، وحافظ على Cookie الخاص `safe_browsing=off` بطريقة محددة النطاق/الطلب دون تكرار أو استبدال Cookies التي يضيفها Jar.
- اختبارات مطلوبة: name/domain/path key، host-only مقابل domain، path matching، HTTPS/Secure، expiry، Max-Age=0/deletion، replacement، منع تسريب Cookie بين المضيفين، فك/تشفير AES-GCM، الاستمرار بعد إعادة إنشاء Jar وإعادة تشغيل قاعدة البيانات، WebView setCookie callback/flush، الدمج دون تكرار، وإثبات أن طلبات ProComic/Coil/Kotatsu تستخدم Jar نفسه.

**قرار الحالة:** هذه نتائج تدقيق وتصميم مرشح فقط. لم يتم تعديل كود التطبيق أو تغيير رقم الإصدار أو تشغيل build في هذه الجولة. يجب أخذ موافقة المستخدم على تصميم التخزين/التشفير قبل تنفيذه.


## 2026-11-22 — اعتماد تصميم CookieJar v4 للتنفيذ

وافق المستخدم على التصميم المقترح بعد تدقيق الكود. التنفيذ المعتمد: Room version 4 مع migration 3→4، تخزين Cookies القادمة من OkHttp بخصائصها المدعومة، تشفير القيم بـAES-GCM ومفتاح Android Keystore، CookieJar singleton مشترك، ومزامنة WebView/OkHttp مع انتظار `setCookie` callback ثم `flush()`.

لا يجوز افتراض خصائص غير متاحة في OkHttp 4.10.0 أو WebView API. بيانات `CookieManager.getCookie(url)` تظل URL-scoped ولا تُحوّل إلى Domain Cookie أوسع. إزالة Cookie headers اليدوية يجب أن تحافظ على `safe_browsing=off` دون تكرار أو استبدال غير مقصود.

الحالة: التصميم معتمد وبدأ التنفيذ على `fix/hijala-parser` عند `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`. لم تُسجل نتائج اختبار حتى الآن.
