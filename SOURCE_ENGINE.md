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


## 2026-11-10 — ProComic API and reader implementation commit `0220e115`

- `ProComicConnector.kt` now uses the live JSON endpoints for search, content details, and paginated chapters instead of depending on the old static HTML extraction path.
- Content identity retains numeric ID, content type, and slug. Chapter URLs use `/ar/chapter/{slug}-{number}-{chapterId}`; only Arabic, unlocked, numeric chapter entries are accepted.
- Reader page count and descriptors come from the live lazy reader slots. A persistent WebView session intercepts the site's `*.procomic.im` image requests through the shared OkHttp client and retains their bytes until the app consumes them.
- A slot may be a single image or multiple positioned pieces; the app stitches pieces into one page. No placeholder chapters or cover-as-page fallback is introduced.
- Listing requests are spaced by five seconds. Reader host fallback between `procomic.pro` and `procomic.net` is bounded to one alternate host.
- Four Robolectric tests cover ContentDto mapping, stable content identity, Arabic/unlocked chapter DTOs and chapter URL shape, plus the reader slot script contract.
- Current CI verification: run `42896328650` for commit `0220e115d34d2e8c6758414be510a25a7a911d3b` was still running its unit-test step at the time of this update.
- Remaining ProComic work: commit and verify the `safe_browsing=off` cookie integration, implement short-lived Turnstile token handling for later listing pages if required, then perform live site and phone tests.
- No phone-level validation has happened; task 3 remains incomplete.


### Follow-up commit `a718ee66` — compile and cookie correction

- CI run `42896328650` exposed two Kotlin errors where `RectF.width` and `RectF.height` were used as properties. They were corrected to `width()` and `height()` in commit `a718ee66cf0a27243891c8766c546c6aad6add2c`.
- Added the ProComic `safe_browsing=off` cookie for both reader hosts through `CookieManager`, and included the browser cookie jar in API requests.
- The follow-up Actions run is `42897100762` (run 86); it was still in setup at the time of this note.


## 2026-11-11 — ProComic Turnstile and reader-check follow-up (`f2d86395`)

- Added an app-owned Turnstile token provider to `AndroidKotatsuWebViewExecutor`: a dedicated WebView renders the upstream site's widget; it displays a foreground dialog only when Cloudflare requires human interaction, and cleans up on success/cancel/timeout.
- Added a four-minute in-memory token cache, `x-turnstile-token` header propagation, a rate-limited retry after HTTP 403, and fallback to the existing browser-verification UI if the token retry is still forbidden.
- Search now advances past API pages containing only unreadable items, using `meta.page` / `meta.pages`; text-only novels and login-blocked series remain excluded.
- Reader slot extraction now follows upstream's `check` / status-row protocol, scrolling the security row into view and routing the user through the app's existing in-app verification UI.
- Added six Robolectric tests for content mapping/identity, Arabic and locked chapter filtering, novel/blocked-series exclusion, API pagination, and reader slot/check behavior.
- Commit: `f2d863959bd5a3f2126d86a56a10936fde02c7c1` on `fix/room-storage`. CI run `43056952825` (run 90) is the verification run for this patch; its final result is pending.
- Previous run 89 attempt 2 confirmed unit tests and Debug APK build succeeded, but the Upload APK step failed because GitHub Actions artifact storage quota was reached. The workflow found one APK file; this is an account/repository storage quota blocker, not a missing APK path. GitHub says usage is recalculated every 6–12 hours.
- No live-site or phone test has been performed. Task 3 remains in progress.
