# محرك المصادر ومحللات المواقع

## مكان التنفيذ
المحللات الفعلية تكون داخل مستودع التطبيق `osamahosny2/iruma_manga_v2`، وليست في مستودع معرفة مستقل. مستودع المعرفة يوثق طريقة التصميم والنتائج والقرارات.

## بنية مستهدفة
- Source registry: قائمة المصادر المدمجة المعروفة.
- Source engine: يختار Parser الصحيح ويقدم واجهة موحدة.
- Parser مستقل لكل موقع: مسؤول عن HTML/API الخاص بالموقع.
- Network/session services: HTTP وHeaders وCookies وWebView/Cloudflare عند الحاجة.
- Models/mappers: تحويل استجابات المصدر إلى نماذج التطبيق.
- Tests: اختبارات لكل Parser وعقود مشتركة.

## دورة البيانات المطلوبة
1. تحديد المصدر الصحيح للعمل.
2. البحث الحقيقي أو اكتشاف الأعمال.
3. تحويل النتائج إلى نموذج موحد مع الاحتفاظ بهوية المصدر.
4. جلب تفاصيل العمل.
5. جلب الفصول الحقيقية وترتيبها وترقيمها.
6. جلب صفحات فصل محدد.
7. التحقق من أن روابط الصفحات تشير إلى صور فعلية وليست HTML أو الغلاف.
8. عرض الصفحات بالترتيب الصحيح.
9. تنزيل الصور والتحقق من الملف المخزن عند اختبار التنزيل.

## التعامل مع اختلاف المواقع
كل موقع قد يختلف في:
- HTML أو API.
- بنية صفحات البحث والتفاصيل.
- ترقيم الصفحات وPagination.
- أسماء الحقول وترقيم الفصول.
- Headers وReferer وUser-Agent.
- Cookies والجلسات.
- الحماية وCloudflare.
- روابط الصور المباشرة أو المؤقتة.
- ترتيب الفصول والصفحات.

لا تفترض أن حل موقع واحد يعمل لكل المواقع. استخرج المشترك في العقد والخدمات، واترك التفاصيل الخاصة داخل Parser الموقع.

## Cloudflare والحماية
- لا تفترض أن كل فشل سببه Cloudflare.
- ميّز بين timeout وDNS وTLS و403 و429 و5xx وHTML challenge وAccess Denied.
- إذا تطلب الأمر WebView، يجب توثيق كيف تُنقل Cookies والجلسة إلى HTTP client.
- لا تعتبر الضغط على زر أو إغلاق WebView نجاحًا؛ يجب رصد نجاح التحقق فعليًا.
- لا تحتجز Mutex أثناء انتظار تدخل المستخدم.
- احترم Cancellation وأعد المحاولة بحدود واضحة.
- لا تتجاوز آليات الحماية بطرق غير مصرح بها؛ تعامل فقط ضمن الاستخدام المسموح وشروط المواقع.

## منع البيانات الوهمية
- لا تنشئ فصول Placeholder لإخفاء فشل Parser.
- لا تستخدم MangaDex أو مصدرًا آخر كـ fallback صامت لمصدر اختاره المستخدم.
- لا تحول الغلاف إلى صفحة أولى.
- إذا لم تتوفر الفصول أو الصفحات، أرجع خطأ مفهومًا مع تفاصيل تشخيصية آمنة.
- افحص نوع المحتوى وحالة HTTP قبل قبول صورة.
- افحص أن الصورة قابلة للفك وليست صفحة HTML محفوظة بامتداد صورة.

## معايير Parser
- Unit tests للـ HTML/API fixtures المسموح استخدامها.
- اختبارات mapping وPagination وترتيب الفصول.
- اختبارات عدم خلط هوية المصدر.
- اختبارات رفض HTML بدل الصورة.
- Integration tests لعمليات الشبكة عند الإمكان.
- Smoke test واقعي على مصدر معروف.
- توثيق القيود والتغييرات التي قد يكسرها تحديث الموقع.

## مصدر مرجعي
Kotatsu-Redo:
- https://github.com/Kotatsu-Redo/Kotatsu-Redo
- https://github.com/Kotatsu-Redo/kotatsu-parsers-redo

التفاصيل التي تم التحقق منها من الكود يجب تسجيلها في `KOTATSU_RESEARCH.md` مع مسار الملف ومرجع commit. لا تنسب تفاصيل إلى Kotatsu دون دليل مباشر.


## نتائج فحص الكود الحالي — 2026-11-10
- كل موقع يجب أن يظل في Parser مستقل، ويُجمع عبر سجل مركزي. هذا هو قرار المستخدم.
- التطبيق الحالي يستخدم اعتماد Gradle خارجيًا `kotatsu-parsers-redo` مثبتًا على commit `b3c000024563920d3eecde5816f2e9d3734fa467`؛ المحللات ليست بعد ملفات محلية في مستودع التطبيق.
- `ConnectorEngine` يوجه `PROCOMIC` إلى `ProComicConnector` وكل المصادر الأخرى إلى `KotatsuSourceEngine`. ملف `StarzConnector.kt` غير موصول بمسارات التوجيه التي تمت مراجعتها.
- `KotatsuSourceEngine.fetchPages()` يعيد بناء فصل ناقصًا من URL فقط قبل استدعاء Parser؛ يجب التحقق مما إذا كانت بعض المحللات تحتاج حقول الفصل الأصلية.
- `fetchChapters()` يحول `chapters == null` إلى نجاح بقائمة فارغة؛ يجب التمييز بين عدم وجود فصول وفشل الاستخراج.
- `MangaViewModel.loadChaptersForManga()` يجلب التفاصيل ثم الفصول، والفصول تجلب التفاصيل مرة أخرى؛ كما لا يظهر guard لمنع نتيجة طلب قديم من تحديث العمل الحالي.
- راجع [تقرير الدراسة الكامل](docs/SOURCE_ENGINE_RESEARCH_2026-11-10.md) قبل اقتراح التنفيذ. هذه نتائج قراءة وتحليل فقط، وليست تغييرات منفذة.


## تنفيذ فعلي — 2026-11-10
- أضيفت الوحدة المحلية `:source-parsers` داخل مستودع التطبيق.
- الوحدة تعرض اعتماد `kotatsu-parsers-redo` كعقود وطبقات مشتركة مؤقتة، وتحتوي المحللات المحلية في `source-parsers/src/main/kotlin/.../parsers/`.
- `LocalSourceParserRegistry` هو السجل المركزي، ويعطي أولوية للمحلل المحلي قبل المصنع الخارجي.
- `IrumaAzoramoonParser` هو أول محلل محلي مستقل. يستخدم Astro props لكل عناصر `astro-island`، وDOM fallback لروابط الفصول، وخصائص الصور الكسولة.
- لم ينقل Manga Pro بعد؛ `ProComicConnector` الحالي لا يزال يستخدم HTML ثابتًا، بينما قارئ الموقع الحالي يعتمد على API وWebView reader slots.
- تم تعطيل المصادر افتراضيًا وإضافة ترحيل مرة واحدة لإلغاء التفعيل التلقائي القديم.
- تم حفظ كائن الفصل الأصلي عند الانتقال من قائمة الفصول إلى استخراج الصفحات، وإيقاف تحويل null chapters إلى نجاح فارغ، وإضافة حماية stale request.
- التنفيذ والـ CI والقيود في [سجل تنفيذ محرك المصادر](docs/SOURCE_ENGINE_IMPLEMENTATION_2026-11-10.md).

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
