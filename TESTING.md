# استراتيجية الاختبارات ومعايير النجاح

## مستويات الاختبار
1. **Static/Build:** فحص التنسيق والأنواع وتجميع التطبيق.
2. **Unit:** اختبار الدوال والعقود والـ mappers وحالات الفشل.
3. **Integration:** اختبار تفاعل Repository وRoom وNetwork وParser.
4. **Source smoke tests:** اختبار مصدر حقيقي على عمليات البحث والتفاصيل والفصول والصفحات.
5. **Device tests:** اختبار Android فعليًا، خصوصًا WebView وCookies وعرض الصور والتنزيل والتخزين.
6. **Regression:** إعادة تشغيل الاختبارات بعد أي تغيير قد يؤثر على مصادر أو طبقات مشتركة.

## تعريف نجاح محرك المصدر
لا يعد Parser ناجحًا إلا إذا ثبتت سلسلة البيانات:
- نتائج بحث حقيقية من الموقع الصحيح.
- تفاصيل العمل الصحيحة مع source identity.
- قائمة فصول حقيقية وليست Placeholder.
- روابط صفحات حقيقية للفصل.
- استبعاد الغلاف من صفحات الفصل.
- استجابات صور فعلية قابلة للفك والعرض.
- ترتيب الصفحات صحيح.
- عدم fallback الصامت لمصدر آخر.
- أخطاء واضحة عند فشل المصدر.

## تعريف نجاح القارئ
- يعرض كل صفحة حقيقية من الفصل.
- التنقل بين الصفحات يعمل.
- حالات التحميل والخطأ والفراغ واضحة.
- لا يستبدل الصفحات بالغلاف.
- لا يفقد سياق العمل أو المصدر.

## تعريف نجاح التنزيل
- تنزيل كل صفحات الفصل المطلوبة.
- التحقق من HTTP/content type وصحة الصورة.
- حفظ الملفات بمسارات مستقرة.
- التعامل مع فشل جزئي وإعادة المحاولة والإلغاء.
- عرض التقدم والحالة.
- إمكانية القراءة دون شبكة بعد اكتمال التنزيل.

## CI مقابل اختبار الهاتف
نجاح GitHub Actions يثبت فقط أن الخطوات التي نفذها workflow نجحت. لا يثبت:
- أن الموقع متاح من شبكة الهاتف.
- أن Cloudflare تم تجاوزه/التحقق منه بنجاح.
- أن الموقع أعاد فصولًا حقيقية.
- أن الصور صالحة.
- أن القارئ والتنزيل يعملان على الجهاز.

يجب أن يذكر التقرير اسم workflow وrun ID والـ commit والـ jobs والـ artifacts، ثم يذكر اختبار الهاتف بشكل منفصل.

## بيانات الاختبار
استخدم fixtures موثقة أو بيانات اختبار مصطنعة لاختبار parser logic، لكن لا تستخدمها لتدّعي أن مصدرًا حقيقيًا يعمل. اختبارات التكامل الواقعية يجب أن تميز بوضوح بين الاختبار الوهمي والاختبار المتصل بموقع حقيقي.

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


## 2026-11-10 — ProComic parser tests and CI checkpoint

The app commit `0220e115d34d2e8c6758414be510a25a7a911d3b` adds four Robolectric tests:

1. Content API DTO → `MangaItem` mapping, cover CDN host, authors, genres, alternate titles, status, source identity and chapter count.
2. Arabic chapter DTO mapping and the exact slug-number-ID reader route.
3. Rejection of non-Arabic and locked chapter entries.
4. Reader slot script contract for lazy rows, deferred media, CDN image paths and scrolling.

CI run `42896328650` was in progress at the unit-test step when recorded. Even if it passes, it verifies only compilation and the tests above; it does not validate a live ProComic response or actual WebView image display. A phone smoke test remains mandatory.


### Follow-up compile failure and fix — 2026-11-10

- Run `42896328650` failed compilation at `ProComicConnector.kt:270–271`: `RectF.width` and `RectF.height` must be called as methods.
- Corrected in commit `a718ee66cf0a27243891c8766c546c6aad6add2c`; that commit also adds the `safe_browsing=off` cookie setup and browser-cookie forwarding.
- Run `42897100762` (run 86) is the follow-up build/test run; result pending.
