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

## 2026-11-21 — Source parser resolution regression tests

- Added `SourceParserResolutionTest` with coverage for local override precedence over upstream `@Broken`, no silent fallback after local-constructor failure, rejection of broken upstream without override, normal upstream creation, stable-ID matching despite duplicate display names, and DivaScans adult classification.
- Commit: `dfdfde17935986e4075340909d9dd8e91f69b6b4`.
- CI run: `44849756102` (Build APK) was still running the unit-test step at the time of this record. Record the final result only after checking unit tests, APK build and signature verification.

## تحديث CI — 2026-11-21

- Run `44849756102`, commit `dfdfde17935986e4075340909d9dd8e91f69b6b4`: **Run unit tests = success**.
- **Build Debug APK = in progress** عند آخر فحص؛ **Verify APK signature = pending**.
- لا تسجل نجاح البناء أو التوقيع قبل أن تظهر خطواتهما `success` في GitHub Actions.


## 2026-11-21 — إصلاح أول فشل Compile في Hijala

- CI run `44853363425` على commit `aa8031d...` فشل في `:source-parsers:compileKotlin` لأن `PagedMangaParser` يطلب تنفيذ `getFilterOptions()`.
- أضيف التنفيذ `MangaListFilterOptions()` في `HijalaParser` بالـcommit `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`.
- CI run الجديد `44853813290` على commit `83e033d...` بدأ؛ كان في `Setup Gradle` عند آخر فحص. لم تُعلن الاختبارات أو APK أو التوقيع ناجحة بعد.


## 2026-11-21 — تدقيق تغطية CAPTCHA/Cloudflare

- CI run `44853813290`, commit `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`: Unit Tests, Debug APK build, stable signing-key verification كلها نجحت. Release publishing step skipped، ولا يوجد workflow artifact.
- يوجد اعتراض Cloudflare مشترك في OkHttp: `PROTECTION_CAPTCHA` يتحول إلى `KotatsuBrowserActionRequiredException`، ثم `withBrowserVerification` ينتظر تحقق WebView ويعيد تنفيذ الطلب.
- WebView لا يعلن النجاح لمجرد اختفاء شاشة التحدي؛ ينتظر حالة الصفحة السليمة ووجود `cf_clearance` في ثلاث قراءات متتالية. `AndroidCookieJar` يربط CookieManager مع OkHttp.
- Manga Pro لديه Turnstile flow مستقل، token cache لمدة 4 دقائق، ثم إعادة طلب بعد 403.
- القيود: `PROTECTION_BLOCKED` يُرجع IOException؛ لا يوجد حل عام مثبت لأنواع CAPTCHA الأخرى؛ لم يتم اختبار نجاح Hijala أو كل مصادر الجرد حيًا بعد التحقق.
- لا تعتبر الكابتشا محلولة على مستوى كل المصادر حتى إجراء source smoke/device tests واقعية.


## تحديث 2026-11-22 — حالة التحقق الحالية

- آخر GitHub Actions run تم التحقق منه: 44853813290 على commit 83e033d. Unit Tests وBuild Debug APK وVerify APK signature جميعها نجحت.
- خطوة نشر private test pre-release كانت Skipped، ولا توجد Artifacts لهذا التشغيل. لذلك لا يوجد APK قابل للتنزيل من هذا الـrun، ولا يجوز اعتبار نجاح CI إثباتًا لاختبار الهاتف.
- لم يتم تشغيل اختبارات جديدة في جلسة 2026-11-22، ولم يتم إنشاء APK جديد أو اختبار أي مصدر على هاتف المستخدم.
- الاختبارات الموجودة التي تمت مراجعتها: SourceParserResolutionTest، KotatsuBrowserChallengeCoordinatorTest، WebviewScreenTest، HijalaMarkupTest، ChapterPageExtractorTest، ConnectorEngineNoFallbackTest.
- فجوة اختبارات مؤكدة: لا توجد اختبارات CookieJar تغطي host-only/domain وPath وSecure وexpiry وMax-Age=0 والاستبدال ومنع التسريب والاستمرار بعد إعادة تشغيل التطبيق ومزامنة WebView.
- HijalaMarkupTest يعتمد HTML fixtures اصطناعية، ولا يثبت نجاح بحث Hijala الحي أو DOM الحالي للموقع.
- لا تعتبر المهمة الثالثة مكتملة حتى تنفيذ الاختبارات الناقصة، نجاح CI مع Artifact قابل للتنزيل، واختبار الهاتف الحي للبحث والتفاصيل والفصول والصور والتنزيلات بعد تحقق المستخدم.


## 2026-11-22 — خطة اختبارات CookieJar بعد التدقيق

### الوضع الحالي
- لا يوجد اختبار مستقل لـ`AndroidCookieJar` أو مخزن Cookie دائم.
- `WebviewScreenTest` يختبر فقط التعرف على `cf_clearance` في نص Cookie، ولا يختبر استيراد جميع خصائص Cookie أو نجاح طلب OkHttp بعد التحقق.
- `AppDatabaseTest` يغطي ترحيلات Room حتى schema version 3؛ إضافة جدول Cookie تتطلب اختبار migration 3→4 مع الحفاظ على بيانات المكتبة والتقدم الحالية.

### مصفوفة القبول المقترحة (غير منفذة)
1. قبول Cookie host-only على المضيف المطابق ورفضه على subdomain.
2. قبول domain Cookie على النطاقات الفرعية المسموح بها فقط.
3. Path matching: `/foo` يطابق `/foo/bar` ولا يطابق `/foobar`.
4. Secure Cookie لا يرسل عبر HTTP ويرسل عبر HTTPS فقط.
5. حذف Cookie عند انتهاء الصلاحية أو `Max-Age=0`، واستبدال Cookie بنفس `(name, domain, path)`.
6. عدم تسريب Cookie بين مصدرين/مضيفين غير متطابقين.
7. حفظ CookieJar وإعادة إنشائه ثم التأكد من استمرار Cookies الدائمة، وعدم استمرار Cookies session-only بعد إعادة التشغيل إذا كانت سياسة المنتج كذلك.
8. تشفير القيمة وفكها باستخدام Android Keystore/AES-GCM؛ فشل فك التشفير لا يطبع القيمة ويؤدي إلى إبطال السجل وطلب تحقق جديد.
9. مزامنة OkHttp→WebView: لا يبدأ WebView تحميل صفحة تعتمد على Cookie قبل نجاح setCookie callback وflush.
10. مزامنة WebView→OkHttp: بعد التحقق اليدوي، يرسل OkHttp Cookies التي يعيدها CookieManager للـURL المطلوب، دون تخمين Domain/Path أوسع.
11. جميع العملاء (Kotatsu/ProComic/Coil/ConnectorEngine) يستخدمون CookieJar موحدًا ولا يضيفون Cookie header مكررًا.
12. اختبار خاص يحافظ على سلوك `safe_browsing=off` دون استبدال Cookie header كاملًا.
13. اختبار تزامن طلبات متعددة للمضيف نفسه، واستبدال/حذف Cookies المتزامنة دون فقد تحديث أحدث.

### شروط نجاح الجولة
- Unit/Robolectric tests + Room migration tests + CI build/signature checks.
- اختبار يدوي على الهاتف: افتح المصدر، أكمل CAPTCHA بنفسك، أعد محاولة البحث/الفصول/الصور، أغلق التطبيق وافتحه، ثم أعد الاختبار للتأكد من استمرار الجلسة.
- لا يُعلن النجاح من مجرد وجود `cf_clearance` أو نجاح WebView؛ يجب نجاح طلب OkHttp الفعلي.
- لم يبدأ التنفيذ أو البناء في 2026-11-22؛ هذه خطة قبول فقط.
