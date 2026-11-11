# حالة المشروع وخطة المهام الثمانية

**آخر تحديث لهذا السجل:** 2026-11-10  
**المستودع:** `osamahosny2/iruma_manga_v2`  
**فرع العمل:** `fix/room-storage`

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

**نتيجة الاختبار الآلي المعروفة:**
آخر تشغيل معروف سابقًا كان GitHub Actions run `42483154189` (run #78)، ونجح فيه البناء والاختبارات الآلية ورفع artifact باسم `app-debug-apk`. commit معروف في سجل سابق هو `82baa349bfb75381f27fee8254045c11df357710` برسالة `fix(connector): resolve page image download compile errors`. هذه بيانات تاريخية يجب التحقق من حداثتها قبل البناء عليها.

**نتيجة الاختبار العملي:**
المستخدم أبلغ أن كل المصادر لا تزال تعرض المشكلة: فصول Placeholder/وهمية، وعند فتح الفصل يظهر الغلاف فقط. لم يثبت عرض صفحات حقيقية بعد. اختبار التنزيل لم يؤكد اكتماله. لذلك المهمة 3 ليست مكتملة مهما كانت نتيجة CI.

**المطلوب قبل إغلاق المهمة 3:**
- دراسة فعلية موثقة لـ Kotatsu-Redo وkotatsu-parsers-redo.
- إثبات أن كل Parser يربط هوية المصدر الصحيحة ولا يعيد بيانات مصدر آخر.
- اختبار البحث وتفاصيل العمل والفصول الحقيقية.
- اختبار استخراج صفحات الفصل الحقيقية واستبعاد الغلاف.
- اختبار عرض الصور والتنقل داخل الفصل.
- اختبار التنزيل عند الوصول لهذه المرحلة/وفق نطاق المهمة.
- رسائل أخطاء واضحة وعدم وجود fallback وهمي.
- اختبار مستخدم عملي على الهاتف وتوثيق النتيجة.

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
