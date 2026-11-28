# حالة المشروع وخطة المهام الثمانية

**آخر تحديث لهذا السجل:** 2026-11-25  
**المستودع:** `osamahosny2/iruma_manga_v2`  
**آخر فرع/HEAD تحققنا منه حيًا:** `fix/hijala-parser` — `ffe3e9c5dee75249a4f2a6dc29a1b6d9d9625a2f`  
**الإصدار الحالي للاختبار:** `1.5.1` (`versionCode=10501`) — Private Test Pre-release منشور؛ ينتظر اختبار المستخدم على الهاتف.

> هذه لقطة حالة مبنية على آخر معلومات موثقة متاحة عند إنشاء هذا الدليل. يجب على أي جلسة عمل لاحقة فحص أحدث commit وGitHub Actions قبل التنفيذ أو تحديث الحالة. لا تفترض أن SHA المذكور أدناه ما زال HEAD الحالي دون فحص.

## ملخص الحالة

| رقم | المهمة | الحالة |
|---|---|---|
| 1 | إعادة بناء نظام تخزين البيانات (Room) | منجزة بحسب سجل التنفيذ |
| 2 | إعادة بناء معمارية التطبيق | منجزة بحسب سجل التنفيذ؛ اجتازت CI |
| 3 | إعادة بناء محرك المصادر | قيد التنفيذ/غير مكتملة؛ تحتاج حلًا واختبارًا عمليًا |
| 4 | إعادة بناء قارئ المنجا | لم تبدأ |
| 5 | إعادة بناء نظام القراءة وسجل القراءة | لم تبدأ |
| 6 | إعادة بناء نظام التنزيلات | لم تبدأ كمهمة مستقلة |
| 7 | إعادة بناء نظام الشبكة والتخزين المؤقت | لم تبدأ كمهمة مستقلة؛ توجد أجزاء شبكية عولجت ضمن المهمة 3 |
| 8 | بناء نظام اختبارات حقيقي | لم تبدأ كمهمة مستقلة؛ توجد اختبارات وحدات وCI سابقة لكنها لا تمثل نظام الاختبارات الكامل المطلوب |

## تفاصيل كل مهمة

### المهمة 1 — إعادة بناء نظام تخزين البيانات (Room)
**الحالة: منجزة بحسب سجل المشروع.**
تم التعامل مع إعادة بناء/تأسيس نظام التخزين باستخدام Room ضمن مرحلة إعادة بناء التطبيق. لا يعني هذا أن كل حالات التخزين المستقبلية أو اختبارات التكامل قد اكتملت؛ يجب الحفاظ على التوافق مع المهام التالية والتحقق عند تعديل التخزين.

### المهمة 2 — إعادة بناء معمارية التطبيق
**الحالة: منجزة بحسب سجل التنفيذ، واجتازت GitHub Actions.**
تضمنت الأعمال المسجلة فصلًا معماريًا باستخدام طبقات Domain/Repositories، وتجميع الاعتمادات في `AppContainer`، ووجود `DownloadManager` و`NetworkClientProvider` ضمن الهيكل. نجاح GitHub Actions لا يثبت اختبار التطبيق كاملًا على الهاتف، ولم يُسجل دمج الفرع في `main` ضمن الحالة المعروفة.

### المهمة 3 — إعادة بناء محرك المصادر
**الحالة: قيد التنفيذ وغير مكتملة.**
هذه المهمة أولوية أساسية لأنها توفر بيانات حقيقية تعتمد عليها بقية المهام. لا تنقل إلى آخر الخطة ولا تتجاوزها إلى بناء القارئ على بيانات وهمية.

الأعمال/الملفات التي ظهرت في سجل التعديلات السابقة تشمل:
- `KotatsuSourceEngine.kt`: إنشاء Parsers للمصادر المدمجة، واستخدام تفاصيل العمل والفصول.
- `ConnectorEngine.kt` و`ChapterPageExtractor.kt`: استخراج الصفحات والتحقق من الاستجابات.
- `AndroidKotatsuWebViewExecutor.kt` و`KotatsuBrowserChallengeCoordinator.kt`: مسار WebView والتعامل مع تحديات المتصفح وCloudflare.
- `AndroidCookieJar.kt` و`NetworkClientProvider.kt`: إدارة Cookies وطلبات الشبكة.
- تعديلات في `MangaViewModel.kt` و`WebviewScreen.kt` و`ReaderScreen.kt` لتوضيح حالة الفشل وتمرير سياق المصدر.
- أضيفت اختبارات وحدات لتنسيق جلسة Cloudflare والإلغاء وإعادة استخدام نجاح التحقق، واختبارات أخرى لرفض HTML بدل صورة واستبعاد الغلاف من صفحات الفصل.

**نتيجة الاختبار الآلي الأحدث — 2026-11-25:**
- الفرع `fix/hijala-parser` عند HEAD `ffe3e9c5dee75249a4f2a6dc29a1b6d9d9625a2f`.
- CI #126 (`45210080619`) و#127 (`45210409242`) اكتمل كلاهما بنجاح.
- شمل التشغيل الناجح Unit Tests (`gradle testDebugUnitTest :source-parsers:test`)، بناء Debug APK، التحقق من بصمة شهادة APK بالمفتاح الثابت، حذف ملف keystore المؤقت، وحفظ Room schema snapshots.
- Room schema v4 موجود، مع جدول `http_cookies` وتشفير AES-GCM لقيم Cookies.
- تم نشر Private Test Pre-release `v1.5.1` (Release ID `455656286`) وإرفاق `app-debug.apk` بحجم 53,323,253 bytes. SHA-256: `fa484f1fd34302b50ae41a9795fbf691ac52bfce86430496bb805e6b9423c9b1`.
- Release: https://github.com/osamahosny2/iruma_manga_v2/releases/tag/v1.5.1
- APK: https://github.com/osamahosny2/iruma_manga_v2/releases/download/v1.5.1/app-debug.apk

**نتيجة الاختبار العملي:**
- إصدار 1.5.1 متاح للمستخدم للاختبار، لكن لا يوجد تأكيد موثق حتى 2026-11-25 بأن المستخدم أنهى اختباره على الهاتف.
- المطلوب: استمرار Cookies بعد إغلاق التطبيق وإعادة فتحه، مشاركة الجلسة بين WebView وOkHttp، عدم تسريب Cookies بين النطاقات، تجربة مصدر محمي بعد أن يتعامل المستخدم مع CAPTCHA بنفسه، والتحقق من البحث والتفاصيل والفصول والصفحات الحقيقية.

**المطلوب قبل إغلاق المهمة 3:**
- انتظار نتيجة اختبار المستخدم لإصدار 1.5.1 ومعالجة ما يظهر من مشاكل بعد فحص السبب الجذري والموافقة على نطاق الإصلاح.
- استكمال التحقق العملي من المصادر: البحث، تفاصيل العمل، الفصول، الصفحات والصور الحقيقية، مع عدم الاعتماد على نتائج وهمية أو تجاوز CAPTCHA.
- توثيق نتيجة الهاتف الفعلية. نجاح CI أو نشر Release لا يساوي اكتمال المهمة.

### المهمة 4 — إعادة بناء قارئ المنجا
**الحالة: لم تبدأ كمهمة مستقلة.**
تعتمد على أن يعيد محرك المصادر صفحات حقيقية. لا تبدأ بناء القارئ اعتمادًا على قوائم أو روابط وهمية. قد توجد تعديلات تشخيصية سابقة في `ReaderScreen.kt` لكنها لا تعني اكتمال إعادة بناء القارئ.

### المهمة 5 — إعادة بناء نظام القراءة وسجل القراءة
**الحالة: لم تبدأ كمهمة مستقلة.**
تشمل حفظ تقدم القراءة وسجل القراءة وحالات القراءة، مع سلوك صحيح عند فتح فصل بالخطأ وإمكانية إرجاع الفصل إلى غير مقروء يدويًا بحسب المتطلبات التي يعتمدها المستخدم.

### المهمة 6 — إعادة بناء نظام التنزيلات
**الحالة: لم تبدأ كمهمة مستقلة.**
توجد مكونات/أعمال تأسيسية مثل `DownloadManager` أو مسارات تنزيل صور، لكنها لا تثبت اكتمال نظام التنزيلات. يلزم تصميم واختبار تنزيل فصل كامل وتخزينه وإدارته دون الاعتماد على الشبكة أثناء القراءة.

### المهمة 7 — إعادة بناء نظام الشبكة والتخزين المؤقت
**الحالة: لم تبدأ كمهمة مستقلة.**
توجد تحسينات شبكية نُفذت ضمن المهمة 3، منها توحيد بعض طلبات HTTP/Cookies، مشاركة جلسة Cloudflare، إعادة المحاولة بعد التحقق، وحد تزامن تنزيل الصور. لكن هذا لا يساوي اكتمال نظام الشبكة والتخزين المؤقت بكامل متطلباته. يجب تحديد النطاق والاعتماديات قبل بدء المهمة رسميًا.

### المهمة 8 — بناء نظام اختبارات حقيقي
**الحالة: لم تبدأ كمهمة مستقلة.**
توجد Unit Tests وGitHub Actions سابقة. المطلوب نظام أوسع يثبت السلوك الحقيقي للمصادر والتخزين والقارئ والتنزيلات والشبكة، مع فصل الاختبارات الوحدوية عن التكاملية والاختبارات الواقعية. لا تستخدم بيانات وهمية كدليل على عمل Parser حقيقي.

## ترتيب الاعتماديات
المهمة 3 prerequisite عملي للمهام التي تحتاج بيانات فصول وصفحات حقيقية، خصوصًا المهمة 4. لا تؤجل محرك المصادر إلى ما بعد القارئ أو بقية الوظائف. يمكن التخطيط للمهمة 8 مبكرًا كممارسة اختبار مستمرة، لكن تنفيذ نظام اختبارات شامل يجب أن يغطي الأنظمة الفعلية بعد توفرها.

## قاعدة تحديث هذا الملف
عند كل تغيير:
- اذكر التاريخ والـ commit.
- اذكر ما تم تنفيذه تحديدًا.
- افصل نتائج CI عن اختبار الهاتف.
- حدّث الحالة فقط عندما تتوفر أدلة.
- لا ترفع المهمة إلى "مكتملة" لمجرد نجاح البناء.


## تحديث دراسة المهمة 3 — 2026-11-10
- الحالة: **قيد البحث والتحليل؛ لم يبدأ تعديل جديد على كود التطبيق في هذه الجولة**.
- فُحص HEAD للفرع `fix/room-storage`: `82baa349bfb75381f27fee8254045c11df357710`.
- تم تأكيد أن التطبيق يعتمد على مكتبة المحللات كاعتماد Gradle خارجي، وأن المستخدم يريد نقل/تنظيم المحللات كملفات مستقلة داخل مستودع التطبيق مع سجل مركزي.
- تمت دراسة عقد Parser وتوليد السجل عبر KSP وبعض محللات Azoramoon وTeamXNovel وMangaStarz.
- ظهرت فرضيات مهمة: إعادة إنشاء الفصل ببيانات ناقصة قبل جلب الصفحات، تحويل null chapters إلى قائمة فارغة ناجحة، احتمال stale response، وتكرار جلب التفاصيل. لم تثبت أي فرضية بعد كسبب وحيد للمشكلة.
- تم اكتشاف اختلاف commit في `THIRD_PARTY_NOTICES.md` عن commit المثبت في Gradle، ومخاطر دمج KSP/dependencies عند نقل المكتبة محليًا.
- لا تزال المهمة 3 غير مكتملة؛ لا يوجد إثبات عملي جديد لظهور فصول وصور حقيقية على الهاتف.
- التفاصيل والأدلة: [تقرير دراسة محرك المصادر](docs/SOURCE_ENGINE_RESEARCH_2026-11-10.md).


## تحديث تنفيذ المهمة 3 — 2026-11-10 (جلسة التنفيذ)
- HEAD الحالي وقت هذا التحديث: `31636df2317829a06254d10ab6117b1d6e40007c` على `fix/room-storage`.
- commit `ad629006`: إصلاح default source activation، ترحيل قديم لتعطيل المصادر المفعّلة تلقائيًا، حفظ كائن الفصل الأصلي، منع null chapter list من التحول لنجاح فارغ، حماية stale chapter loads، واختبارات جديدة.
- CI للـ commit `ad629006`: Unit tests وBuild Debug APK نجحا؛ Upload APK فشل بسبب GitHub Actions artifact storage quota.
- commit `31636df`: إضافة `:source-parsers` وLocalSourceParserRegistry ومحلل Azora محلي واختبارات markup، مع تصحيح THIRD_PARTY_NOTICES.
- CI للـ commit `31636df` كان قيد التنفيذ عند آخر تحقق؛ لا تسجل نجاحه حتى تظهر نتيجة نهائية.
- Manga Pro لم يُصلح بعد؛ المحلل الحالي HTML ثابت بينما قارئ الموقع الحالي يعتمد على API وWebView slots.
- لا اختبار عملي على الهاتف بعد؛ المهمة 3 غير مكتملة.
- التفاصيل: `docs/SOURCE_ENGINE_IMPLEMENTATION_2026-11-10.md`.

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


## تحديث المهمة 3 — تصحيح قارئ Manga Pro — 2026-11-10

- أحدث commit على osamahosny2/iruma_manga_v2 / fix/room-storage: d560fc54eff633f89d171c51c78dcbc7fbbc5ecd.
- commit سابق في نفس السلسلة 0220e115d34d2e8c6758414be510a25a7a911d3b أضاف مسار API الحقيقي للمحتوى والفصول وربط قارئ ProComic بجلسة WebView الدائمة.
- commit d560fc5 صحّح منطق slot script وفق ملف upstream assets/slot.js، وأضاف ترتيب البحث latest_chapter وحد 12 نتيجة للبحث النصي.
- تم التأكد من أن القارئ يعتمد على ProComicReaderSession الموجودة، وأن تنزيل الصور يمر عبر ProComicConnector بدل إرسال روابط procomic-reader:// مباشرة إلى OkHttp.
- CI run 42896444353 قيد التنفيذ وقت التحديث؛ إعداد Android وGradle نجحا، واختبارات الوحدة كانت تعمل. لا تسجل نجاح البناء قبل النتيجة النهائية.
- لم يحدث اختبار حي أو اختبار هاتف بعد. الفصول والصور الحقيقية لم تثبت بعد.
- الحالة: المهمة 3 ما زالت قيد التنفيذ. المهمة 4 وما بعدها لا تُعلن مكتملة بناءً على هذا التعديل.


## تحديث لاحق — 2026-11-10 — CI وتصحيح التجميع

- commit a718ee66cf0a27243891c8766c546c6aad6add2c أضاف اختبار DTO للفصول، وتجهيز Cookie safe_browsing=off، ومشاركة Cookies المتصفح في طلبات API والصور، وحدودًا آمنة لتجميع الصور المركبة.
- كشف CI run 42896328650 خطأ تجميع محددًا في RectF.width وRectF.height. تم تصحيحهما باستدعاء الدالتين width() وheight() في commit 8db92b23a10ec010a65543069a6cb80f4e0bc142.
- أحدث رأس الفرع الآن: 8db92b23a10ec010a65543069a6cb80f4e0bc142.
- CI run 42897158863 قيد التنفيذ؛ نجح إعداد Android وGradle وكانت اختبارات الوحدة تعمل عند آخر فحص.
- لا تزال المهمة 3 قيد التنفيذ حتى ينجح CI ثم يثبت البحث والفصول والصور الحقيقية على الهاتف. لا يوجد تأكيد تشغيل حي بعد.


## 2026-11-11 — ProComic Turnstile checkpoint

- Latest app commit: `f2d863959bd5a3f2126d86a56a10936fde02c7c1` on `fix/room-storage`.
- Implemented Turnstile token generation through a dedicated WebView, human-interaction dialog, four-minute in-memory token cache, 403 retry and fallback to the app browser-verification UI.
- Improved readable search pagination and reader security-check handling. Six Robolectric tests now cover the new parser/reader contracts.
- CI run `43056952825` (run 90) is pending.
- Run 89 attempt 2: all unit tests passed; Debug APK build passed; Upload APK failed because GitHub Actions artifact storage quota was reached despite finding the APK file.
- Task 3 remains in progress: wait for run 90; resolve artifact storage quota before an APK can be downloaded from Actions; perform live Manga Pro and phone tests; then build Azora to the same standard.


## تحديث المهمة 3 — 2026-11-12

- أحدث commit موثق على `fix/room-storage`: `8041716bcc3919f34234de33fd3ab7e398059501` (`fix(azora): filter novels and preserve chapter titles`).
- تم تحديث Azoramoon إلى API الحالي للبحث/التفاصيل/الفصول/الصفحات، وإضافة mapper واختبارات تمنع خلط الروايات وتضمن حفظ رقم الفصل وعنوانه.
- تم تثبيت اعتماد `kotatsu-parsers-redo` على commit `68f3fed3952f5c9cf1ad8cc00bdb1ff047688dd8` مع استمرار وجود المحللات المحلية في `:source-parsers`.
- تم تأكيد أن كل المصادر غير مفعلة عند التثبيت الجديد؛ Settings تتيح التفعيل اليدوي وBrowse يخفي غير المفعّل.
- تم إصلاح استعادة كائن الفصل الأصلي من تفاصيل العمل بعد إعادة تشغيل التطبيق، بدل إعادة إنشاء فصل ناقص من الرابط فقط.
- تمت مراجعة Manga Pro: API حقيقي للفصول والتفاصيل، وقارئ WebView دائم للصور وslots المركبة، مع Turnstile/Cloudflare handling. يظل `ProComicConnector` خارج وحدة `:source-parsers` حاليًا، ويحتاج قرار توحيد معماري.
- CI run `43242033999` (run 94) كان في خطوة Unit Tests عند آخر فحص؛ لم يتم تسجيل نتيجة نهائية.
- لا يوجد حتى الآن اختبار حي أو اختبار هاتف بعد هذه التعديلات؛ لم يثبت إصلاح مشكلة Placeholder/الغلاف فقط عمليًا. المهمة 3 ما زالت غير مكتملة.
- التفاصيل: `docs/SOURCE_ENGINE_IMPLEMENTATION_2026-11-12.md`.


## متابعة المهمة 3 — 2026-11-12 (بعد مراجعة أحدث commits)

**أحدث HEAD تم التحقق منه:** `c9347644cd1b145c53de10eaef3467206f92f8d7` على `fix/room-storage`.

- `:source-parsers` أصبح Module محليًا داخل مستودع التطبيق، لكنه يستخدم اعتماد Gradle خارجيًا `kotatsu-parsers-redo` عند commit `68f3fed3952f5c9cf1ad8cc00bdb1ff047688dd8` للعقود والـbase parsers وبقية المصادر التي لم تُنقل محليًا. محلل Azoramoon نفسه override محلي عبر `LocalSourceParserRegistry`؛ هذا تصميم هجين وليس اعتمادًا وقت التشغيل على تنزيل كود من GitHub.
- محلل Azoramoon المحلي يستخدم API الحالي للتفاصيل والفصول وصفحات الصور، ويحتفظ بهوية الفصل ورقمه وعنوانه، ويرفض الفصول المقفلة والروايات. أضيفت اختبارات mapping للترتيب، العناوين، الحالات، الروايات، الصفحات المقفلة وترتيب الصور.
- تم إصلاح تمرير `MangaChapter` الأصلي وسياق العمل عبر طبقات التطبيق؛ وعند استعادة فصل محفوظ بعد إعادة تشغيل التطبيق يُطابق الفصل الأصلي من قائمة Parser بالرابط بدل إنشاء كائن ناقص من الرابط وحده.
- تم إصلاح سياسة تفعيل المصادر: Browse يعرض المصادر المفعّلة فقط، بينما Settings يعرض الكتالوج للتفعيل اليدوي. التهيئة الأولى تكتب `enabled=false` لكل مصدر لتجاوز سجلات قديمة. تم رفع marker إلى v3 لأن بعض الأجهزة قد تكون طبّقت marker v2 المعيب بالفعل؛ هذا التحديث يعيد تعطيل المصادر مرة واحدة ثم يحترم تغييرات المستخدم بعد ذلك.
- تمت مراجعة Manga Pro: البحث والتفاصيل والفصول تأتي من JSON API، وهوية العمل تشمل النوع والمعرف والـslug، والفصول العربية المقفلة تُستبعد. صفحات القارئ تُكتشف عبر WebView والـlazy slots، وقد تكون صورة مباشرة أو أجزاء مركبة. هذا مسار صفحات حقيقي في الكود وليس قائمة فصول Placeholder، لكنه لم يُختبر حيًا على الهاتف بعد.
- نقطة لاحقة للمهمة 6: مسار Reader يرسل `procomic-reader://` إلى `ProComicConnector.downloadPageImageBytes`، لكن `DownloadManager` الحالي يستعمل مسار `downloadImageBytes(url)` العام؛ هذا المسار لا يحل الـvirtual URI الخاص بـProComic. يجب إصلاحه ضمن مهمة التنزيلات، ولا يُعد إثباتًا لفشل عرض الصور داخل Reader.
- تم تحديث `THIRD_PARTY_NOTICES.md` ليطابق commit `68f3fed...`. ترخيص المكتبة GPL-3.0 متوافق مع ترخيص التطبيق GPL-3.0 بشرط الالتزام بإشعارات المصدر وتوفير المصدر المقابل عند التوزيع وفق GPL.
- commits التطبيق ذات الصلة: `f74b4d960455436a32abbc5f8245ec334d507b47` (سياق هوية الفصل)، `8041716bcc3919f34234de33fd3ab7e398059501` (فلترة الروايات وحفظ عنوان الفصل)، `97e2992e8c9dd35b4c7b2bc804e61ab18bb66c27` (تعطيل الكتالوج عند التهيئة الأولى)، `c9347644cd1b145c53de10eaef3467206f92f8d7` (إعادة تشغيل الترحيل عبر marker v3).
- أحدث CI موثّق: run `43242711623` (run 96) للـHEAD `c9347644...` كان لا يزال قيد التنفيذ عند آخر فحص؛ لا تُسجل نتيجة نهائية قبل إعادة التحقق.
- لا يوجد اختبار حي أو اختبار هاتف مثبت حتى الآن. يجب تثبيت البحث، تفاصيل العمل، الفصول العربية الحقيقية، صور أول/آخر صفحة، ترتيب الصور، وعدم ظهور الغلاف كصفحة، على Azora وManga Pro قبل إغلاق المهمة 3.

**الحالة لا تتغير:** المهمة 3 ما زالت قيد التنفيذ؛ المهام 4–8 لم تبدأ كمراحل مستقلة.


## النتيجة النهائية لـ CI run 96 — 2026-11-12

- workflow run: `43242711623` للـHEAD `c9347644cd1b145c53de10eaef3467206f92f8d7`.
- خطوة `Run unit tests`: **نجحت**.
- خطوة `Build Debug APK`: **نجحت**؛ Gradle أكد `BUILD SUCCESSFUL` ووجد workflow ملف APK واحدًا في المسار المتوقع.
- خطوة `Upload APK`: **فشلت فقط بسبب GitHub Actions artifact storage quota** (`Failed to CreateArtifact: Artifact storage quota has been hit`). هذا ليس خطأ Kotlin أو Gradle، ولا يعني أن APK لم يُبنَ؛ لكنه يعني أن الملف لم يُحفظ كـArtifact قابل للتنزيل.
- GitHub يذكر أن إعادة احتساب المساحة قد تستغرق 6–12 ساعة. يلزم حذف Artifacts قديمة من إعدادات Actions أو انتظار إعادة احتساب الحصة ثم إعادة تشغيل workflow للحصول على APK قابل للتنزيل.
- لا يوجد اختبار حي على Azora/Manga Pro ولا اختبار يدوي على الهاتف حتى الآن؛ لذلك المهمة 3 تظل قيد التنفيذ.


## تحديث إضافي للمهمة 3 — commit 8ce7065
- تم إصلاح تنزيل فصول Manga Pro: DownloadManager يستخدم معالج الصفحات المعتمد على المصدر والرؤوس بدل downloader العام، مع اختبار Robolectric لعنوان `procomic-reader://`.
- توحيد تصنيف الروايات في Azoramoon بين البحث والتفاصيل.
- أحدث commit: `8ce70650772a4ad6a50a522ce5a813fda58074b0`. run 97 قيد الاختبار.
- run 96 أثبت نجاح Unit Tests وبناء Debug APK، لكن رفع Artifact فشل بسبب امتلاء حصة تخزين GitHub Actions؛ لذلك لا يوجد APK قابل للتنزيل حتى الآن.
- المهمة 3 غير مكتملة حتى نجاح CI الحالي والاختبار الحي على الهاتف.


## تحديث 2026-11-18 — جرد المحللات وخطة الإصلاح حسب الشهرة

- المهمة 3 ما زالت قيد التنفيذ.
- المستخدم طلب إصلاح جميع المحللات المعطلة، بما فيها المحللات التي لا يذكر تعليقها سبب التعطل.
- ترتيب العمل المعتمد: المجموعة العربية العامة أولًا، ثم الإنجليزية العامة، ثم مصادر 18+ (مع فصل العربية والإنجليزية داخل المجموعة). داخل كل مجموعة يبدأ الإصلاح بالأكثر شهرة/استخدامًا ثم الأقل.
- يجب إعادة تدقيق إحصاءات الرد السابق قبل اعتمادها؛ لم تثبت بعد سلامة اكتمال جلب كل ملفات المحللات، ولذلك لا تعتمد أرقام 468/153/105 كأرقام نهائية.
- لا يُعتبر Parser صالحًا لمجرد أن الموقع يفتح. يلزم اختبار البحث والتفاصيل والفصول والصفحات/الصور، بحسب عقد المصدر.
- لا تغييرات على تفاصيل الأعمال والفصول والصفحات، ولا تعديل لقارئها. اختبارات عدم الانحدار فقط.
- لا خلط بين Manga Pro (`PROCOMIC`) وProChan (`PROCHAN`).
- لم يُعدّل كود التطبيق في هذه الجولة. الخطوة التالية إعادة جرد موثوق ثم إصلاح تدريجي واختبار وتوثيق كل دفعة.
- إصدار التحقق المستهدف بعد إكمال المهمة الثالثة يظل 1.5.1؛ لا تغيير للإصدار أثناء الإصلاح والاختبارات.


## 2026-11-18 — تقدم إصلاح المحللات حسب الشهرة

- أعيد التحقق من جرد الإنتاج: 470 ملف Kotlin (403 en، 67 ar)، 468 Parser definitions (401 en، 67 ar)، 153 `@Broken`، 105 `HENTAI`. لا تخلط file count مع parser definition count.
- الإصلاحات ستُنفّذ بمجموعات منفصلة: العربية العامة، الإنجليزية العامة، ثم 18+ (عربي/إنجليزي). داخل كل مجموعة الأكثر شهرة أولًا.
- أول هدف عربي عام: Hijala (`HIJALA`)؛ الموقع نشط وانتشاره مرتفع (تقدير 1.56M زيارة في سبتمبر 2026). المحلل القديم يستخدم Blogger feed بينما الموقع الحالي صفحات أعمال وفصول WordPress-like؛ يلزم Parser محلي محدث.
- اكتُشف أن `KotatsuSourceEngine` يمنع local override لمصدر upstream معلّم Broken. هذا عائق بنيوي يجب إصلاحه قبل تفعيل محللات محلية بديلة.
- لا كود تطبيق عُدّل بعد. لا selectors تخمينية ولا رفع لعلامة Broken قبل اختبارات parser الفعلية. لا تعديل لمسارات تفاصيل الأعمال والفصول والصفحات المشتركة أو القارئ.
- الإصدار النهائي المستهدف بعد إكمال المهمة الثالثة: 1.5.1؛ لا تغيير رقم الإصدار أثناء الإصلاح والاختبارات.


## 2026-11-20 — استكمال تشخيص المهمة 3

- استؤنف العمل بعد انقطاع الإنترنت، وأُعيد فحص HEAD في `osamahosny2/iruma_manga_v2` على `fix/room-storage`: `b4360e73faba53e74e29d3ad3aa38466701091e7`.
- تأكد من الملفات الفعلية وجود عائق local override: `getCatalogSources()` و`resolveParser()` يرفضان محلل upstream المعلّم `isBroken` قبل السماح بتجاوز محلي؛ السجل المحلي يدعم `AZORAMOON` فقط.
- لا كود تطبيق جديد ولا اختبارات جديدة في هذه الجولة. لا يزال Hijala يحتاج فحص DOM حي قبل كتابة Parser، ولا تُستخدم selectors تخمينية.
- أُتيح للمستخدم ربط TinyFish كبديل للمتصفح السحابي بعد تعذر Browserless؛ الربط لم يكتمل بعد، لذلك فحص DOM الحي متوقف على إتمامه.
- المهمة 3 ما زالت قيد التنفيذ. ترتيب المجموعات كما اعتمده المستخدم: العربية العامة، الإنجليزية العامة، ثم 18+ (عربية/إنجليزية منفصلتان). لا تعديل لمسارات تفاصيل الأعمال والفصول والصفحات أو القارئ. الإصدار المستهدف بعد اكتمال المهمة 3: `1.5.1`، ولا تغيير للإصدار أثناء العمل.


## 2026-11-20 — استكمال فحص Hijala ومحرك local overrides

- TinyFish أصبح متاحًا بعد إكمال الربط. فحص صفحة Hijala للقائمة وصفحة Lookism في المتصفح السحابي؛ الاثنتان محجوبتان بتحدي Cloudflare قبل المحتوى. لم تتم محاولة تجاوز التحدي، ولا توجد selectors DOM موثوقة بعد.
- عائق المحرك المؤكد: كتالوج المصادر وحل parser يرفضان `isBroken` قبل المحلل المحلي، والسجل المحلي يدعم `AZORAMOON` فقط.
- الإجراء البرمجي التالي هو تعديل أولوية local override مع اختبارات تغطي النجاح والفشل، دون تغيير مسارات التفاصيل والفصول والصفحات أو القارئ.
- لا تعديل تطبيق في هذه الجولة؛ لم يبدأ بعد الإصلاح البرمجي. المهمة 3 قيد التنفيذ، ترتيب المصدر كما سبق، والإصدار المستهدف `1.5.1` دون تغيير أثناء الاختبار.


## 2026-11-20 — تأكيد إضافي لحجب صفحة Hijala

- إعادة الفحص المباشر لصفحة `https://hijala.com/lookism/` أعادت `bot_blocked`، بما يتفق مع نتيجة المتصفح الحي التي أظهرت تحقق Cloudflare قبل محتوى الصفحة.
- لا توجد بيانات DOM/Selectors مؤكدة للفصول أو الصور؛ لا نكتب Parser تخمينيًا ولا نتجاوز التحقق.
- كود التطبيق لم يتغير. Local Parser Override ما زال العائق البرمجي الأول المؤكد؛ المهمة 3 قيد التنفيذ والإصدار المستهدف 1.5.1 دون تغيير أثناء الاختبارات.\n\n
## 2026-11-21 — مرجع جرد المصادر اليدوي
- قدّم المستخدم ملفًا من 7 صفحات باسم `DOC-20261121-WA0000.pdf` يتضمن صور جرد يدوي وملاحظات.
- قاعدة المستخدم: أي مصدر عليه علامة صح في الصور يُعد شغالًا 100% وفق اختباره، ويُستخدم مرجعًا لدراسة محللات المصادر الأخرى.
- المصادر التي لا تملك محللات تُترك حاليًا خارج نطاق الإصلاح؛ لأن أغلبها لمواقع متوقفة أو غير موجودة.
- لا يُعد فتح الموقع وحده دليلًا على عمل المحلل، ولا تُنسب علامة صح إلى مصدر قبل التأكد من موضعها في الصورة ومطابقتها مع source ID والرابط.
- أُنشئ `docs/SOURCE_AUDIT_2026-11-21.md` لتسجيل القواعد والأسماء المقروءة مبدئيًا وحدود دقة القراءة.
- الأسماء الأولية: AdultWebtoon, 1stKissManga.com, 1stKissManga.net, 4QScans, A95Comics (غير مؤكدة), 19PornComic (غير مؤكدة), ALLPornComic (غير مؤكدة), Arenascans, ArabToons, ArabsDoujin, BananaScan.com, BeeHentai, BeeToon.net, BestManga.com.
- لم يُرفع ملف PDF الثنائي نفسه بعد؛ التوثيق النصي لا يحل محل حفظ الملف الأصلي. لا تعتبر أرشفة المرفق مكتملة حتى رفعه والتحقق من فتحه من GitHub.
- لا تعديل لكود التطبيق في هذا التوثيق. المصادر بلا محللات ليست ضمن نطاق الإصلاح الحالي.

## 2026-11-21 — بدء التنفيذ الفعلي للمهمة 3

- المستودع المستخدم: `osamahosny2/iruma_manga_v2` فقط، الفرع `fix/room-storage`؛ لم يتم لمس المستودع الأصلي.
- بدأ التنفيذ من HEAD السابق `b4360e73faba53e74e29d3ad3aa38466701091e7`، وأصبح HEAD بعد أول شريحة `dfdfde17935986e4075340909d9dd8e91f69b6b4`.
- أول شريحة: إصلاح أولوية local parser override على upstream `@Broken`، مطابقة المصادر بالـID الثابت فقط، وتصحيح تصنيف DivaScans إلى 18+.
- أضيفت 8 اختبارات وحدات في `SourceParserResolutionTest`.
- CI run `44849756102` بدأ على commit الجديد؛ حالة الاختبارات والبناء والتوقيع لا تزال قيد الانتظار عند كتابة هذا التحديث.
- لا تغيير لرقم الإصدار، ولا إصدار APK/Release جديد في هذه المرحلة. لا تُعد المهمة 3 مكتملة قبل نجاح CI والاختبارات الحية للمصادر وإثبات البيانات الحقيقية.
- لم يُرفع PDF الجرد الثنائي إلى GitHub بسبب مشكلة مساحة التخزين؛ تمت إضافة الأسماء المؤكدة والتصنيفات إلى وثيقة الجرد النصية.

## 2026-11-21 — تحديث نتيجة الاختبارات

- في CI run `44849756102` على commit `dfdfde17935986e4075340909d9dd8e91f69b6b4`: خطوة **Run unit tests نجحت**.
- خطوة **Build Debug APK** بدأت؛ فحص بصمة التوقيع لم يبدأ بعد عند آخر فحص.
- لا يوجد Release جديد ولا تغيير لرقم الإصدار. لا تعتبر الشريحة متحققة بالكامل حتى نجاح البناء وفحص التوقيع.
- محاولات الوصول العادية إلى Hijala REST endpoints أعادت `bot_blocked`؛ لم يتم تجاوز Cloudflare.


## 2026-11-21 — إصلاح أول فشل Compile في Hijala

- CI run `44853363425` على commit `aa8031d...` فشل في `:source-parsers:compileKotlin` لأن `PagedMangaParser` يطلب تنفيذ `getFilterOptions()`.
- أضيف التنفيذ `MangaListFilterOptions()` في `HijalaParser` بالـcommit `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`.
- CI run الجديد `44853813290` على commit `83e033d...` بدأ؛ كان في `Setup Gradle` عند آخر فحص. لم تُعلن الاختبارات أو APK أو التوقيع ناجحة بعد.


## 2026-11-21 — نجاح CI ومراجعة الكابتشا على فرع Hijala

- الفرع: `fix/hijala-parser`، HEAD: `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`.
- GitHub Actions run `44853813290` (run #121) اكتمل بـ`success`؛ Unit Tests وBuild Debug APK وVerify APK signature كلها `success`.
- خطوة نشر private test pre-release كانت `skipped`، وقائمة Artifacts لهذا التشغيل فارغة. لم يُنشر APK جديد ولم يتغير رقم الإصدار.
- قائمة Releases لا تزال تعرض `v1.5.0` فقط كأحدث إصدار، وهو مبني من commit أقدم `b4360e73faba53e74e29d3ad3aa38466701091e7`، وليس من Hijala commit الحالي.
- مراجعة الكابتشا: `KotatsuCloudFlareInterceptor` مشترك في OkHttp المستخدم بمحرك المصادر؛ عند `PROTECTION_CAPTCHA` يرفع `KotatsuBrowserActionRequiredException`. `KotatsuSourceEngine.withBrowserVerification` ينسق WebView التفاعلي ثم يعيد الطلب. `WebviewScreen` يتحقق من clearance cookie، و`AndroidCookieJar` يشارك كوكيز WebView مع OkHttp.
- Manga Pro يملك مسار Turnstile خاصًا به مع رمز مؤقت لأربع دقائق وإعادة محاولة بعد 403؛ هذا المسار ليس حلًا عامًا لكل المواقع.
- فجوات مؤكدة: `PROTECTION_BLOCKED` يتحول إلى IOException ولا يبدأ التحقق التفاعلي؛ لم يثبت نجاح التحقق حيًا على Hijala أو كل المصادر؛ لا يوجد إثبات أن جميع أنواع الكابتشا غير Cloudflare مدعومة.
- النتيجة: دعم Cloudflare مشترك موجود جزئيًا، لكن مشكلة الكابتشا ليست محلولة بالكامل ولا يجوز إعلانها كذلك. يلزم اختبار حي بعد التحقق لكل مصدر قبل اعتباره صالحًا.
- المهمة 3 مستمرة؛ لا دمج إلى `main` ولا Release ولا تغيير إصدار.


## متابعة جلسة 2026-11-22 — استئناف المهمة الثالثة

- تمت قراءة ملف التسليم المرفق وملف جرد المصادر وملخصه. الجرد يحصي 1350 ملف Kotlin و1385 annotation، منها 429 Broken و22 مصدرًا مرجعيًا بعلامة الصح. يوجد Local Overrides في الجرد لـAZORAMOON وHIJALA.
- تم العثور على Skill /برمجة في Composio واستخدامها. تمت مراجعة مستودع التطبيق والمستودع التعليمي والفروع وCI وReleases وArtifacts مباشرة.
- حالة التطبيق: الفرع fix/hijala-parser عند 83e033d، والفرع fix/room-storage عند dfdfde. آخر CI run 44853813290 نجح في Unit Tests وBuild Debug APK وVerify APK signature، لكنه لا يحتوي Artifacts. أحدث Release هو v1.5.0 من commit أقدم b4360e73.
- تمت مراجعة كود AndroidCookieJar وNetworkClientProvider وKotatsuSourceEngine وWebviewScreen وWebView executor وHijala parser والاختبارات. لا يوجد مخزن CookieJar داخلي مستقل؛ التنفيذ الحالي جسر إلى Android CookieManager. لا توجد اختبارات CookieJar للاستمرار بعد إعادة التشغيل أو المزامنة الثنائية.
- اختبارات Hijala الحالية تعتمد HTML fixtures اصطناعية. اختبار البحث الحي غير مكتمل لعدم توفر MHTML لصفحة البحث.
- تمت مراجعة كود Kotatsu الأصلي وKotatsu-Redo وkotatsu-parsers-redo وMihon، إضافة إلى توثيق Android وOkHttp وCloudflare وReddit وGitHub Issues. البحث في X لم ينتج منشورات قابلة للتحقق؛ يسجل كبند بحث غير مكتمل.
- لم يُعدل كود التطبيق في هذه الجولة، ولم يتغير رقم الإصدار، ولم يُنشأ APK أو Release جديد. تم إنشاء ملف Word محلي مرحلي باسم Iroma_Reader_Project_Handover_2026-11-22.docx.
- **تحديث تعليق التنفيذ السابق:** كان هناك تعارض مؤقت بين وثيقة التسليم وتعليمات الفرع القديمة. حُسم التعارض بتوجيه المستخدم في 2026-11-22، وأصبحت القاعدة هي العمل على أحدث فرع `fix/`؛ لذلك الفرع المعتمد حاليًا هو `fix/hijala-parser` عند `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`. لا يوجد انتظار لموافقة إضافية على اختيار هذا الفرع.
- تم تحديث `CONVERSATION_HISTORY.md` في commit `8172b3d615294448ce34c82d8b05f4469b2b098c` ثم أضيف قرار اختيار الفرع في التحديث التوثيقي الحالي.
- المتبقي: اعتماد تصميم CookieJar الدائم وحدود مزامنة WebView، تنفيذ واختبار الكوكيز، تدقيق وإصلاح المحللات المستهدفة، اختبار البحث والفصول والصفحات والتنزيلات على الهاتف، ثم إنشاء APK اختبار وتقرير إغلاق.


## 2026-11-22 — حسم المستخدم لقاعدة اختيار فرع الإصلاح

- **توجيه المستخدم المعتمد:** نصلح على فرع يبدأ اسمه بـ`fix/`؛ والأفضل العمل على أحدث فرع موجود. إذا احتجنا إنشاء فرع جديد، يكون أيضًا باسم `fix/...`، لكن لا ننشئ واحدًا ما دام أحدث فرع مناسبًا للاستكمال.
- تمت إعادة التحقق من قائمة الفروع الحية في `osamahosny2/iruma_manga_v2` ومن تفاصيل HEAD لكل فرع إصلاح: `fix/hijala-parser` عند `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe` بتاريخ `2026-11-21 02:04:21 UTC`، و`fix/room-storage` عند `dfdfde17935986e4075340909d9dd8e91f69b6b4` بتاريخ `2026-11-21 01:22:04 UTC`.
- بناءً على ترتيب التوقيت الفعلي، `fix/hijala-parser` هو الأحدث، وهو فرع الاستكمال المعتمد حاليًا. لم يُنشأ فرع جديد.
- هذا التوجيه **يستبدل** القيد القديم الذي كان يحصر العمل في `fix/room-storage` فقط. يجب تحديث أي تعليمات أو ملخص قديم يكرر ذلك القيد.
- يبقى ممنوعًا تعديل `main` أو الدمج أو إصدار Release دون إذن منفصل. إنشاء فرع جديد — إذا أصبح ضروريًا — يكون من أحدث فرع إصلاح مناسب بعد التحقق من HEAD، وليس من `main` تلقائيًا.
- لم يحدث أي تعديل على كود التطبيق في هذه الخطوة؛ التغيير توثيقي فقط في المستودع التعليمي.
- الحالة التالية: استكمال المهمة 3 على `fix/hijala-parser` وفق `/برمجة`، مع تحديث مستودع المعرفة في نفس جولة أي تعديل برمجي.


## 2026-11-22 — استكمال خطوة تصميم الكوكيز ومراجعة دمج الفروع

### قرار الفروع بعد المقارنة الفعلية
- مقارنة `fix/room-storage...fix/hijala-parser`: `ahead_by=3`, `behind_by=0`. الفرع الأحدث descendant من الفرع القديم ويحتوي جميع commits القديمة؛ لا حاجة إلى merge بينهما.
- `main...fix/hijala-parser`: `ahead_by=101`, `behind_by=0`. لا merge إلى `main` أثناء العمل. نحتفظ بالفروع القديمة كنسخ مرجعية/احتياطية ولا نحذفها في هذه المرحلة.
- فرع التنفيذ الحالي: `fix/hijala-parser`، HEAD `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe` وقت الفحص.

### نتائج تدقيق الكوكيز
- فُحصت ملفات AndroidCookieJar وNetworkClientProvider وKotatsuSourceEngine وAndroidKotatsuWebViewExecutor وWebviewScreen وProComicConnector وConnectorEngine وApplication وAppContainer.
- التنفيذ الحالي يعتمد CookieManager، لا يملك مخزن Cookie داخليًا، لا ينتظر callback لـsetCookie، ويعيد إنشاء Jar في عميل Kotatsu. كما توجد ترويسات Cookie يدوية في ProComicConnector وConnectorEngine.
- Room موجودة في الفرع الحالي بإصدار schema 3، وRoom 2.7.0 وOkHttp 4.10.0 مثبتان. لا يوجد جدول Cookie أو DAO أو اختبار CookieJar.
- الخطوة التالية المقترحة: تصميم CookieJar singleton مع تخزين Room version 4، تشفير قيم Cookies بـAES-GCM/Android Keystore، مطابقة صحيحة للخصائص، مزامنة WebView/OkHttp مع انتظار callbacks، وتوحيد جميع العملاء وإزالة الحقن اليدوي مع الحفاظ على `safe_browsing=off`.
- قيد مهم: `CookieManager.getCookie(url)` يعرض أزواج name/value فقط ولا يكشف جميع خصائص Domain/Path/expiry/HttpOnly/Secure/SameSite؛ لذلك WebView-to-OkHttp يجب أن يكون URL-scoped ولا يجوز توسيع نطاق Cookie المستوردة تخمينًا.
- هذا التصميم مرشح غير منفذ وغير معتمد بعد. لا يوجد تعديل على كود التطبيق، ولا build جديد، ولا تغيير إصدار. ننتظر موافقة المستخدم على التصميم قبل التنفيذ.


## 2026-11-22 — موافقة التنفيذ وبدء Room v4 CookieJar

- وافق المستخدم صراحةً على تنفيذ التصميم: Room schema v4 / migration 3→4، AES-GCM عبر Android Keystore، وCookieJar singleton مشترك.
- أعيد التحقق من الفروع مباشرة: أحدث فرع إصلاح `fix/hijala-parser` عند HEAD `83e033d028ee8404b63ca52da6c3a5dd6ea22ffe`؛ سيظل فرع التنفيذ.
- بدأ تنفيذ المهمة 3 على هذا الفرع. لا تعديل على `main`، لا merge، لا تغيير إصدار، ولا Release دون إذن مستقل.
- نطاق التنفيذ: مخزن Room دائم، تشفير القيم، مشاركة Jar بين العملاء، مزامنة WebView مع انتظار callback/flush، إزالة الترويسات اليدوية المكررة مع الحفاظ على `safe_browsing=off`، واختبارات شاملة.
- قيد قبول أساسي: WebView `getCookie(url)` لا يكشف خصائص Domain/Path/expiry الأصلية؛ لا يجوز توسيع نطاق Cookies المستوردة تخمينًا.
- الحالة الآن: التنفيذ بدأ؛ لم يُعلن نجاح أي اختبار أو build بعد.


## 2026-11-22 — تنفيذ Room v4 وبدء CI

- Commit التطبيق: `0ddcec4a25439592fa62b61f588a49e14c0c347a` على `fix/hijala-parser`.
- الملفات العشرة المحدثة: `AppDatabase.kt`, `CookieValueCipher.kt`, `AndroidCookieJar.kt`, `NetworkClientProvider.kt`, `KotatsuSourceEngine.kt`, `ProComicConnector.kt`, `AndroidKotatsuWebViewExecutor.kt`, `IrumaMangaApplication.kt`, `ConnectorEngine.kt`, `AppDatabaseTest.kt`.
- تم رفع Room schema إلى 4 وإضافة `http_cookies` بمفتاح مركب `(name, domain, path)` وفهرس expiry وترحيل 3→4. القيم مشفرة AES-GCM بمفتاح Android Keystore.
- `CookieJarProvider` يهيئ قاعدة البيانات والـJar قبل الشبكة؛ NetworkClientProvider وKotatsuSourceEngine يستخدمان نفس المثيل.
- أزيلت الترويسات اليدوية Cookie من مسارات ProComic وConnectorEngine وWebView image interception. يستمر إعداد `safe_browsing=off` من WebView CookieManager ويقرأه Jar حسب URL.
- أضيفت اختبارات DAO للمفتاح المركب والاستبدال، واختبار ترحيل مباشر من schema 3 إلى 4 يحافظ على المكتبة والتقدم، إضافة إلى ترحيل v2→v4.
- تم التحقق من AndroidManifest وأن `IrumaMangaApplication` هو Application class.
- CI run `45039776683` / #122 على commit `0ddcec4` بدأ؛ `Run unit tests` كان in_progress عند آخر فحص. build/signature/Artifact لم تُحسم بعد.
- لا تغيير في `versionName=1.5.0` أو `versionCode=10500`، ولا Release أو merge أو تعديل main.


## 2026-11-22 — إصلاح أول فشل CI

- CI run `45039776683` فشل عند compileDebugUnitTestKotlin. سجل البناء أكد نجاح compileDebugKotlin؛ السبب المحدد أن اختبار `versionThreeMigrationAddsCookieTableAndPreservesLibraryAndProgress` استدعى `suspend` DAO methods خارج Coroutine.
- تم تصحيح الاختبار فقط باستخدام `runBlocking`. Commit التصحيح: `8d71e9d0166d5b6350bfde86aa5386abd2849b40` على `fix/hijala-parser`.
- CI جديد تلقائي: run `45040379978` (#123) على commit التصحيح؛ ما زال بحاجة إلى التحقق من النتيجة. لم يُنتج APK من run السابق لأن خطوة الاختبارات فشلت.
- لا تعديل على كود الإنتاج بعد run الأول؛ لا تغيير إصدار أو Release أو merge.


## 2026-11-23 — إصلاح فشل CI وإعادة تشغيل التحقق

- فرع العمل الحالي `fix/hijala-parser`، HEAD `66f6ef1b8e8ce5a78e9b65075f3866c0b61e802e`.
- Workflow #123 (`45040379978`) فشل عند `:app:testDebugUnitTest`: 42 اختبارًا، 2 فشلا. الأول `ConnectorEngineNoFallbackTest` لأن CookieJarProvider لم يكن مهيأ في اختبار JVM؛ والثاني `AppDatabaseTest.initializationError` بسبب إرجاع Boolean من اختبار ترحيل 3→4 بدل Unit.
- إصلاحات Commit `66f6ef1`: أضيف `Unit` في اختبار Room؛ أضيف حقن CookieJar إلى ConnectorEngine وKotatsuSourceEngine مع إبقاء CookieJarProvider.jar هو الافتراضي؛ عُزل اختبار ConnectorEngine باستخدام CookieJar.NO_COOKIES.
- CI run #125 / `45197200521` بدأ يدويًا على HEAD الحالي، مع `publish_release=false` و`release_version=1.5.0`. آخر حالة: in_progress، وSetup Android SDK قيد التنفيذ.
- الإصدار لا يزال 1.5.0 / 10500؛ لا يتم رفعه قبل نجاح CI الكامل والتحقق اليدوي من CookieJar/Room على جهاز Android.
- لم يحدث merge إلى main أو نشر Release.


## 2026-11-23 — CI #125 ناجح؛ انتظار اختبار الهاتف

- تأكد نجاح Workflow run #125 / `45197200521` على commit `66f6ef1b8e8ce5a78e9b65075f3866c0b61e802e` في `fix/hijala-parser`.
- جميع خطوات CI نجحت: unit tests، `assembleDebug`، تحقق التوقيع الثابت، إزالة keystore من workspace، وRoom schema snapshot.
- `app/schemas/com.example.data.AppDatabase/4.json` أصبح موجودًا على الفرع ويعرّف Room v4 وجدول `http_cookies` مع القيم المشفرة وخصائص المطابقة.
- الـworkflow لم ينشر Release (`publish_release=false`) ولم ينتج أي GitHub artifact قابل للتنزيل (artifacts=0).
- HEAD الفرع بعد run: `212295bdfbf7ad59cffc7e8de92ee93c44ed3547`؛ يجب إعادة التحقق قبل أي تعديل.
- `versionName=1.5.0`, `versionCode=10500` دون تغيير. لا يثبت CI وحده سلامة مزامنة WebView/OkHttp على هاتف حقيقي.
- الخطوة التالية: توفير Debug APK كـworkflow artifact بعد موافقة المستخدم على تعديل workflow، ثم اختبار الاستمرار/مطابقة النطاقات/مشاركة الجلسة على جهاز Android. لا CAPTCHA automation؛ المستخدم يتعامل معها بنفسه. لا Release ولا رفع إصدار قبل نتيجة الهاتف.


## 2026-11-23 — تصحيح حالة المهام والتمييز بين المهمة والخطوة

- أعيدت قراءة ترتيب العمل المعتمد `docs/USER_APPROVED_WORK_ORDER_2026-11-16.md` وتعليمات `AI_INSTRUCTIONS.md` و`HOW_TO_WORK_WITH_THE_USER.md` من المستودع الحي. التسلسل الأصلي ثابت: (1) Room والتخزين، (2) المعمارية وتدفق البيانات، (3) محرك المصادر، (4) قارئ المنجا، (5) القراءة وسجل القراءة، (6) التنزيلات، (7) الشبكة والتخزين المؤقت، (8) نظام الاختبارات الشامل.
- تصحيح صريح: **المهمة 4 هي إعادة بناء قارئ المنجا**. تجهيز APK أو نشر Release أو اختبار الهاتف خطوة تحقق/توزيع مرتبطة بالعمل الجاري في المهمة 3، وليست مهمة رابعة ولا مهمة مستقلة من المهام الثماني.
- المهمة 3 ما زالت **قيد التنفيذ وغير مكتملة**. تنفيذ Room v4/CookieJar جزء حقيقي من المهمة 3، لكنه لا يحقق وحده كامل معايير محرك المصادر.
- تحقق CI #125 (`45197200521`) بنجاح على commit `66f6ef1b8e8ce5a78e9b65075f3866c0b61e802e`؛ وبات الفرع بعد حفظ Room schema عند HEAD `212295bdfbf7ad59cffc7e8de92ee93c44ed3547`. نجحت Unit Tests وDebug APK والتوقيع وحفظ schema v4. هذا نجاح آلي فقط وليس اختبار هاتف.
- لا يوجد Artifact في تشغيل #125. التفضيل المعتمد عند طلب نسخة اختبار هو استخدام Private Test Pre-release الموجود في Workflow وإرفاق APK به، لا اختراع مسار Artifact منفصل.
- المستخدم طلب صراحةً إصدار `1.5.1` بغرض اختبار التحديث والتحقق منه. هذا الإذن يخص إصدار نسخة اختبار ولا يعني أن المهمة 3 مكتملة. لا يُحذف Release `v1.5.0` قبل التحقق من النسخة الجديدة وطلب المستخدم/تأكيده.
- تم تحديث `app/build.gradle.kts` إلى `versionName=1.5.1` و`versionCode=10501` في commit التطبيق `ffe3e9c5dee75249a4f2a6dc29a1b6d9d9625a2f` على `fix/hijala-parser`.
- بدأ CI التلقائي #126 / `45210080619` على هذا commit؛ عند آخر فحص كان `in_progress` في خطوة Setup Gradle، ولم يُعلن نجاح الاختبارات أو البناء بعد. بعد نجاحه يُشغّل Workflow يدويًا بخيار `publish_release=true` و`release_version=1.5.1` لنشر Private Test Pre-release.
- نتائج الهاتف المطلوبة بعد تثبيت النسخة: استمرار Cookies بعد إغلاق التطبيق وإعادة فتحه، مشاركة الجلسة بين WebView وOkHttp، عزل النطاقات وخصائص Cookie، ثم تجربة مصدر محمي بعد أن يتولى المستخدم CAPTCHA بنفسه، والتحقق من البحث والتفاصيل والفصول والصفحات الحقيقية.
- لا تبدأ المهمة 4 قبل إغلاق المهام 1–3 وفق معاييرها وموافقة المستخدم.
- **تصحيح وثائقي:** بعض رؤوس الملفات كانت قديمة ومتناقضة: README وHOW_TO_WORK حصرا الفرع في `fix/room-storage`، وAI_INSTRUCTIONS احتفظ بـHEAD قديم، ورأس PROJECT_STATUS مؤرخًا 2026-11-12. تم تحديث المراجع الحالية أعلاه؛ السجلات القديمة داخل الملف تبقى تاريخًا ولا تُعامل كالحالة الحالية إذا خالفت هذا الملحق الأحدث.


## 2026-11-25 — الحالة الحية المؤكدة بعد إصدار 1.5.1
- تم التحقق مباشرة من المستودع `osamahosny2/iruma_manga_v2` عبر Composio. الفرع `fix/hijala-parser` HEAD هو `ffe3e9c5dee75249a4f2a6dc29a1b6d9d9625a2f`.
- `app/build.gradle.kts`: `versionName=1.5.1`, `versionCode=10501`.
- CI #126 / `45210080619`: `completed / success`. CI #127 / `45210409242`: `completed / success`.
- Release `v1.5.1` ID `455656286` منشور بتاريخ 2026-11-23، مع `app-debug.apk` (53,323,253 bytes)، SHA-256 `fa484f1fd34302b50ae41a9795fbf691ac52bfce86430496bb805e6b9423c9b1`.
- الإصدار `v1.5.0` و`v1.4.1` ما زالا موجودين؛ لم يُحذف أي Release في هذا التحديث.
- المهمة 1 (Room/storage) والمهمة 2 (architecture/data flow) موثقتان كمنجزتين وفق نطاقهما واختباراتهما السابقة. المهمة 3 (source engine) ما زالت قيد التنفيذ وغير مكتملة حتى اختبار المستخدم لـ1.5.1 وإغلاق المتطلبات العملية. المهام 4–8 لم تبدأ كمهمات مستقلة.
- لا يوجد تعديل على كود التطبيق في هذه الجولة؛ هذا تحديث توثيقي فقط. لا merge إلى `main` ولا تغيير في إصدار التطبيق.
- **تنبيه تاريخي:** أي فقرة أقدم في هذا الملف تقول إن 1.5.1 ينتظر CI أو Release، أو إن HEAD هو `212295b...`، تصف الحالة وقت كتابتها فقط، وقد استُبدلت بهذه الحالة المؤرخة أعلاه. لا تحذف سجل الأحداث التاريخي.


## 2026-11-28 — اختبار المستخدم لتحديث 1.5.1
- أكد المستخدم أنه ثبّت تحديث `1.5.1` فوق النسخة `1.5.0` على الهاتف.
- نجح التحديث، وفتح التطبيق وعمل بعد التثبيت.
- أكد المستخدم أن بياناته انتقلت كما هي من 1.5.0 إلى 1.5.1؛ لا توجد خسارة بيانات مُبلغ عنها في هذا الاختبار.
- أكد المستخدم أن مشكلة المصادر ما زالت قائمة ولم تُحل.
- نتيجة القبول: نجاح ترقية التطبيق واستمرارية البيانات ضمن هذا السيناريو؛ فشل/عدم اكتمال قبول محرك المصادر عمليًا. المهمة 3 تظل مفتوحة، ولا يجوز إعلان اكتمالها.
- لم يُحدد المستخدم بعد أي مصدر/وظيفة بعينها ما زالت تفشل في هذه الجولة؛ يلزم تحديد السلوك المتبقي قبل اقتراح تعديل برمجي.
- لا تغيير في كود التطبيق أو الإصدار أو الفروع في هذه الجولة؛ تحديث توثيقي لنتيجة اختبار المستخدم فقط.
