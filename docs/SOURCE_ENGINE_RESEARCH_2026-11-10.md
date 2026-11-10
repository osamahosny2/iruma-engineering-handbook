# تقرير دراسة محرك المصادر — 2026-11-10

## الحالة والنطاق
- نوع العمل: دراسة وقراءة فقط؛ لم يتم تعديل كود التطبيق أو مستودعات Kotatsu.
- مستودع التطبيق: `osamahosny2/iruma_manga_v2`، الفرع `fix/room-storage`.
- HEAD الذي فُحص: `82baa349bfb75381f27fee8254045c11df357710`.
- مستودع المعرفة: `osamahosny2/iruma-engineering-handbook`، الفرع `main`.
- مكتبة المحللات: `Kotatsu-Redo/kotatsu-parsers-redo`، الإصدار المثبت في Gradle هو commit `b3c000024563920d3eecde5816f2e9d3734fa467`.
- تطبيق Kotatsu-Redo: `Kotatsu-Redo/Kotatsu-Redo`، الفرع `devel` وقت الفحص.
- لا Fork، ولا تعاون، ولا Pull Request إلى Kotatsu. تمت قراءة الملفات العامة فقط.

## قرار المستخدم بشأن شكل المحللات
- كل موقع له Parser مستقل في ملف Kotlin مستقل.
- توجد مجموعة/مجلد واحد يجمع كل المحللات مع سجل أو مدير مركزي يتيحها للتطبيق.
- ملفات المحللات داخل مستودع التطبيق نفسه، وتُدمج في APK؛ لا يعتمد التطبيق على تنزيل ملفات المحللات من GitHub وقت التشغيل.
- مستودع المعرفة يوثق البحث والتصميم والقرارات، ولا يحتوي على كود تشغيل المحللات.

## ما اكتُشف في التطبيق الحالي

### الاعتماد الحالي
`app/build.gradle.kts` يحتوي على اعتماد خارجي:
`com.github.Kotatsu-Redo:kotatsu-parsers-redo:b3c000024563920d3eecde5816f2e9d3734fa467`.
هذا يعني أن ملفات المحللات ليست حاليًا منقولة إلى مجلد محلي داخل مستودع التطبيق؛ التطبيق يستهلك مكتبة منشورة كاعتماد Gradle.

### مسار التوجيه
- `ConnectorEngine.getCatalogSources()` يجمع المصادر القادمة من `KotatsuSourceEngine` مع مصدر `PROCOMIC` المخصص.
- `PROCOMIC` يذهب إلى `ProComicConnector` في البحث والتفاصيل والفصول والصفحات.
- كل مصدر آخر يذهب إلى `KotatsuSourceEngine`.
- `StarzConnector.kt` موجود، لكن لا يوجد استدعاء له في مسارات التوجيه العامة داخل `ConnectorEngine` الذي فُحص. وجود الملف وحده لا يعني أن التطبيق يستخدمه.
- كتالوج Kotatsu يُبنى من `MangaParserSource.values()`، ويقتصر في الكود الحالي على اللغتين `ar` و`en`. فشل إنشاء Parser أو وسمه `isBroken` يجعل المصدر غير متاح، كما أن المصادر تبدأ `isEnabled = false`.

### حل هوية المحلل
- `resolveParser()` يحاول أولًا `MangaParserSource.valueOf(source.id)`، ثم يقارن العنوان بعد تطبيعه.
- لا يختار Parser بناءً على الدومين، وهذا يمنع اختيار محلل خاطئ لمجرد تشابه عنوان الموقع.
- فشل إنشاء Parser يُخزّن في `missingParserCache`؛ وهذا يخفي بعض تفاصيل سبب الفشل ويمنع إعادة المحاولة في عمر المحرك الحالي.

### جلب الفصول والصفحات
- البحث يستدعي `parser.getList()` مع offset محسوب من الصفحة.
- التفاصيل والفصول تستخدم `parser.getDetails()`.
- `fetchChapters()` يحوّل `details.chapters == null` إلى قائمة فارغة ناجحة باستخدام `orEmpty()`؛ هذا يخلط بين «الموقع لم يرجع فصولًا» و«الموقع رجع قائمة فارغة فعلًا».
- `mapChapter()` يحفظ رابط الفصل المشفر، لكنه يضع `pageCount = 0` دائمًا.
- `fetchPages()` يعيد إنشاء `MangaChapter` اصطناعيًا من رابط الفصل فقط: `number = 0`, `title = null`, وبيانات أخرى فارغة، ثم يستدعي `parser.getPages()` و`parser.getPageUrl()`.
- قد يكون هذا كافيًا للمحللات التي تعتمد على `chapter.url` فقط، لكنه قد يفقد معلومات يحتاجها محلل موقع آخر مثل `chapter.id` الأصلي أو رقم الفصل أو بياناته. هذا احتمال يحتاج اختبارًا لكل Parser، وليس سببًا مثبتًا لكل المواقع.
- يتم استبعاد روابط الغلاف ثم رفض النتيجة إذا لم تبقَ صفحات. لا يوجد fallback مقصود إلى الغلاف في هذا المسار.

### إدارة الحالة في ViewModel
- `MangaViewModel.loadChaptersForManga()` يطلب تفاصيل العمل أولًا، ثم يطلب الفصول.
- `KotatsuSourceEngine.fetchChapters()` يطلب `getDetails()` مرة أخرى؛ بالتالي قد تُنفذ عملية التفاصيل مرتين عند فتح العمل.
- لا يظهر في `loadChaptersForManga()` token أو guard يضمن أن نتيجة الطلب تخص العمل الذي ما زال محددًا بعد عودة الشبكة. لذلك توجد فرضية سباق (stale response) قد تسمح لنتيجة طلب قديم بتحديث قائمة العمل الحالي. يلزم اختبار موجه لإثباتها أو نفيها.

### الشبكة والتحقق
- `NetworkClientProvider` يوفر OkHttp وCookieJar مبنيًا على Android `CookieManager` ومعترضات الشبكة.
- `KotatsuSourceEngine` يبني عميلًا مشتقًا ويضيف معترضات Kotatsu الخاصة بالطلبات وCloudflare.
- `IrumaMangaApplication.onCreate()` يستدعي `AndroidKotatsuWebViewExecutor.initialize(this)`، ثم يقرأ `KotatsuSourceEngine` المنفذ من `AndroidKotatsuWebViewExecutor.getOrNull()`؛ إذن يوجد ربط ظاهر بين تهيئة التطبيق ومنفذ WebView.
- وجود آلية Cloudflare لا يثبت أن كل فشل سببه Cloudflare. يجب التفريق بين DNS/TLS/timeout/HTTP 403/429/5xx وHTML challenge وبيانات parsing الخاطئة.

## ما تعلمناه من kotatsu-parsers-redo

### عقد Parser
`MangaParser` يحدد واجهة موحدة تشمل المصدر، الدومين، إمكانات البحث والفرز والفلاتر، `getList`, `getDetails`, `getPages`, `getPageUrl`, رؤوس الطلبات، واعتراض الطلبات. كل Parser يطبق ما يلزم لموقعه.

### التسجيل التلقائي
- `@MangaSourceParser` Annotation ذات `SOURCE` retention.
- `ParserProcessor.kt` يستخدم KSP للعثور على الكلاسات المعلّمة.
- KSP يولد `MangaParserSource` (سجل المصادر) و`MangaParserFactory` (إنشاء Parser المناسب) أثناء البناء.
- الاسم الفريد للمصدر يكون UPPER_CASE، والـ Annotation يحمل العنوان واللغة ونوع المحتوى.
- هذا يحقق فكرة «ملف مستقل لكل موقع + ملف/سجل مركزي يجمعهم» دون فحص ملفات عشوائي وقت تشغيل التطبيق.

### نماذج الفصل والصفحة
- `MangaChapter` يحتفظ بالمعرف والرابط ورقم الفصل والعنوان والمجلد والمترجم وتاريخ الرفع والمصدر.
- `MangaPage` يحتفظ بالمعرف والرابط والمعاينة والمصدر؛ الرابط قد يكون صورة مباشرة أو صفحة HTML وسيطة.
- `getPageUrl()` يحول الصفحة إلى رابط الصورة الفعلي.
- يجب ألا نختزل بيانات الفصل عند تخزينه ثم نعيد بناء كائن ناقص إذا كان Parser يحتاج الحقول الأصلية.

### أمثلة فعلية تمت قراءتها
- `site/ar/Azoramoon.kt`: يعتمد على API؛ يستخدم `/api/query` للبحث، `/api/post` للتفاصيل، `/api/chapters` للفصول، و`/api/chapter` لصور الفصل. يفرز الصور وفق حقل `order` ويتعامل مع الفصول المقفلة والروايات.
- `site/ar/TeamXNovel.kt`: يعتمد على HTML واستخراج عناصر الصفحة، ويدعم Pagination للفصول مع تحميل صفحات متعددة بالتوازي.
- `site/madara/ar/MangaStarz.kt`: ملف خاص بالموقع يرث من `MadaraParser` لأن الموقع يستخدم بنية Madara مشتركة؛ يحدد الدومين وبعض إعدادات الموقع. ليس مطلوبًا تكرار كل كود Madara داخل كل موقع.
- جرى حصر 10 ملفات في `site/ar` و27 في `site/madara/ar` و56 في `site/en` عند الـ commit المثبت؛ هذا جرد لهذه المسارات فقط وليس العدد الكامل لكل Parser بلغتي ar/en في كل الحزم.

### الاختبارات upstream
`MangaParserTest.kt` يغطي عمليات مثل القائمة والبحث والفلاتر والتفاصيل والصفحات والدومين والروابط. الاختبارات المعروضة تعتمد على مواقع حية في عدد من الحالات؛ لذلك تتأثر بتوفر الموقع وCloudflare والكوكيز وتغير HTML. سنستفيد من تغطية السلوك، لكن نحتاج أيضًا اختبارات fixtures محلية ثابتة واختبارات تكامل حقيقية مضبوطة.

## مخاطر وملاحظات اكتُشفت

### اختلاف مرجع الترخيص
- اعتماد Gradle مثبت على `b3c000024563920d3eecde5816f2e9d3734fa467`.
- `THIRD_PARTY_NOTICES.md` في التطبيق يذكر commit مختلفًا: `434030d4811c83dff6191437d8170721379b436d`.
- ملف `LICENSE` في التطبيق وملف `LICENSE` في مكتبة المحللات يحتويان نص GPL-3.0.
- يجب تصحيح توثيق commit والاحتفاظ بإشعارات المصدر والترخيص عند أي نقل أو تعديل.
- GPLv3 يسمح بالنسخ والتعديل وفق شروطه؛ توزيع APK يتطلب الالتزام بالشروط المناسبة، بما فيها توفير Corresponding Source والإشعارات، وقد تنطبق متطلبات Installation Information بحسب طريقة التوزيع والمنتج. هذا ملخص تقني وليس استشارة قانونية.

### قابلية نقل المكتبة إلى Module محلي
المكتبة upstream مشروع Gradle مستقل مكوّن من:
- وحدة Kotlin/JVM رئيسية للمحللات.
- وحدة `kotatsu-parsers-ksp` لتوليد سجل المحللات.

النقل إلى داخل مستودع التطبيق ممكن من حيث المبدأ، لكنه ليس مجرد نسخ ملفات Parser وحدها؛ يجب نقل طبقة الـ core المشتركة والـ models والـ network interfaces ومولد السجل واعتماداته.

مخاطر البناء التي ظهرت من مقارنة Gradle:
- Kotlin في المشروعين `2.2.10`.
- KSP في التطبيق `2.3.5`، بينما مكتبة المحللات تطلب `2.2.10-2.0.2`؛ يجب توحيد/اختبار إصدار KSP قبل اعتماد Module محلي.
- المكتبة تستخدم Kotlin JVM toolchain 17 بينما التطبيق يحدد Java compatibility 11.
- المكتبة تطلب OkHttp `5.1.0` بينما التطبيق يعلن `4.10.0`، ونسخة Jsoup مختلفة أيضًا (`1.21.2` مقابل `1.23.2`). يجب فحص resolved dependency graph بدل افتراض التوافق.
- بناء المكتبة يستورد `tasks.ReportGenerateTask`؛ يجب تحديد مصدر هذا الـ build logic قبل دمجها.

## فرضيات السبب الجذري التي يجب اختبارها
1. فقدان حقول `MangaChapter` الأصلية عند إعادة إنشائه داخل `fetchPages()` قد يكسر محللات تحتاج معرف الفصل أو بياناته، ويؤدي إلى عدم استخراج الصور.
2. تحويل `chapters == null` إلى نجاح بقائمة فارغة يخفي فشل الاستخراج كأنه لا توجد فصول.
3. غياب حماية stale-response في `loadChaptersForManga()` قد يخلط نتائج طلبات أعمال مختلفة عند تداخل التحميل.
4. فشل إنشاء Parser المخزن في `missingParserCache` قد يبقي المصدر غير متاح بعد فشل مؤقت أو إعداد غير جاهز.
5. وجود `StarzConnector` غير موصول بالـ dispatch يعني أن كوده لا يعالج مصدر Starz في المسار الحالي؛ المصدر غير ProComic يذهب إلى Kotatsu engine.
6. تكرار طلب التفاصيل مرتين قد يضاعف الحمل ويخلق اختلافًا بين بيانات التفاصيل والفصول.

هذه فرضيات مدعومة بمسارات الكود لكنها ليست إثباتًا أن واحدة منها تفسر كل نتائج الهاتف. يلزم التقاط source ID وParser identity وروابط العمل والفصل وروابط الصفحات الفعلية لمصدر متأثر.

## التوصية المعمارية الأولية — لم تُنفذ بعد
- إنشاء Module محلي داخل مستودع التطبيق، مثل `:source-parsers`، أو مجلد مصدر منظم ضمنه.
- الاحتفاظ بملف مستقل لكل موقع في `parsers/site/...`.
- وجود `SourceParser` بعقد موحد و`SourceRegistry`/`SourceManager` يجمع المحللات ويختارها بالـ ID الثابت.
- فصل خدمات HTTP/Cookies/Headers/Browser challenge عن منطق Parser، مع السماح لكل Parser بإضافة متطلبات موقعه.
- يمكن استخدام BaseParser مشترك للمواقع التي تعمل على CMS واحد (مثل Madara)، مع بقاء تعريف كل موقع في ملفه.
- نسخ/تضمين المصدر upstream داخل المستودع (vendoring) ليس Fork ولا تعاونًا، لكنه يخضع لترخيص GPL-3.0 وإشعاراته.
- لا يعتمد التطبيق وقت التشغيل على GitHub أو مستودع خارجي لتحميل Parser.

لم تتم الموافقة على تنفيذ هذا التصميم أو تعديل Gradle/كود التطبيق بعد. هذه توصية الدراسة فقط.

## الملفات التي تمت مراجعتها في التطبيق
- `KotatsuSourceEngine.kt`
- `ConnectorEngine.kt`
- `ProComicConnector.kt`
- `StarzConnector.kt`
- `ChapterPageExtractor.kt`
- `NetworkClientProvider.kt`
- `AndroidCookieJar.kt`
- `AndroidKotatsuWebViewExecutor.kt`
- `KotatsuLoaderContextAdapter.kt`
- `KotatsuBrowserChallengeCoordinator.kt`
- `MangaViewModel.kt`
- `MangaModels.kt`
- `ConnectorMangaSourceRepository.kt`
- `AppContainer.kt`
- `IrumaMangaApplication.kt`
- `WebviewScreen.kt`
- `BrowseScreen.kt`
- `ReaderScreen.kt`
- `app/build.gradle.kts`, `settings.gradle.kts`, `gradle/libs.versions.toml`
- `ChapterPageExtractorTest.kt` وملفات الاختبارات ذات الصلة التي ظهرت في شجرة المشروع.

## الخطوة التالية
استكمال الدراسة التفصيلية للمحللات التي تطابق مصادر المستخدم الفعلية، ثم تقديم خطة تنفيذ محددة مع الملفات المتأثرة ومخاطر الترخيص والبناء والاختبارات. لا يبدأ أي تعديل على التطبيق قبل موافقة المستخدم الصريحة.
