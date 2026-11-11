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


## 2026-11-10 — Manga Pro reader slot regression

- App commit under test: d560fc54eff633f89d171c51c78dcbc7fbbc5ecd.
- Workflow: https://github.com/osamahosny2/iruma_manga_v2/actions/runs/42896444353
- Last observed state: Android SDK and Gradle setup succeeded; unit tests were in progress; APK build had not started.
- Required live/device sequence: search for a known Manga Pro title; open details; confirm real Arabic chapters and stable chapter IDs; open a chapter; confirm nonzero page count; display page 1 and a later page; verify a composite page is stitched without gaps; download and reopen offline.
- Also verify a blocked/locked chapter is not shown as readable, an empty API response is reported as an error rather than a fake list, and Turnstile/Cloudflare does not silently become an empty success.
- A green CI run is only a build/unit-test result; it does not count as a live-source or phone test.


## 2026-11-10 — ProComic compile correction

- Previous run 42896328650 failed at app:compileDebugKotlin because RectF.width and RectF.height were accessed as properties.
- Fixed in app commit 8db92b23a10ec010a65543069a6cb80f4e0bc142 by calling width() and height().
- New run: https://github.com/osamahosny2/iruma_manga_v2/actions/runs/42897158863
- At last check: setup steps succeeded; Run unit tests was in progress; APK build and upload were pending.
- After a green run, verify the resulting APK artifact, then perform live Manga Pro search/details/chapters/pages/download tests on-device. CI success alone is not source validation.


## 2026-11-11 — ProComic Turnstile test set

Six Robolectric tests now cover: content mapping and stable identity; Arabic/unlocked chapter mapping and URL shape; locked chapter variants; rejection of text-only novels and login-blocked series; pagination past API pages with no readable results; and the reader slot/check protocol.

Commit `f2d863959bd5a3f2126d86a56a10936fde02c7c1` is being verified by CI run `43056952825` (run 90). Previous run 89 attempt 2 passed tests and Debug APK assembly but failed only at Upload APK because GitHub Actions storage quota was reached.
