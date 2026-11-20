# المهمة 3 — محرك المصادر والتحقق والبحث — 2026-11-17

## نتائج المراجعة
- `ConnectorEngine` يوجّه المصادر المدعومة إلى `KotatsuSourceEngine` أو `ProComicConnector`، واختبار `ConnectorEngineNoFallbackTest` يمنع إنشاء بيانات عامة وهمية عند عدم وجود parser صالح.
- `KotatsuSourceEngine` يحوّل URL/title/number للفصول من parser الحقيقي، ويطلب الصفحات من `parser.getPages()` ثم `parser.getPageUrl()`.
- `ProComicConnector` يستخرج الفصول من بيانات API الفعلية، ويرفض السجلات غير الصالحة/المقفلة، ويستخرج عدد صفحات القارئ عبر جلسة WebView.
- لم يُعثر على إنشاء صريح لفصول وهمية في المسار الحالي؛ لذلك لم يتم تغيير parser mapping تخمينًا. استمرار ظهور بيانات وهمية بعد APK الجديد يحتاج تحديد اسم المصدر والعمل لاختبار parser المحدد.

## إصلاح Cloudflare CAPTCHA
- السبب: `WebviewScreen` كان يعد الصفحة ناجحة بعد 3 حالات DOM غير محمية دون التحقق من وجود كوكي `cf_clearance`، ما يسمح بإغلاق المتصفح قبل أن تصبح جلسة المصدر صالحة.
- الإصلاح: فحص كوكي غير فارغة على عنوان المصدر (مع عناوين WebView الحالية) قبل احتساب دورة نجاح؛ ثلاث دورات ناجحة مع وجود الكوكي مطلوبة قبل `onVerified`. عند غيابها يظهر تنبيه انتظار بدل إغلاق المتصفح.
- `KotatsuBrowserChallengeCoordinator` أصبح يربط الجلسة باسم المضيف؛ اسم العرض ليس هوية أمنية للنطاق.
- اختبارات: cookie parsing، ومشاركة النطاق نفسه عبر display names مختلفة، وفصل نطاقين لهما اسم عرض واحد.

## إصلاح البحث متعدد المصادر
- البحث الشامل كان ينشئ coroutine غير محفوظ في `searchJob`؛ لذلك قد يكتب استعلام قديم نتائجه بعد استعلام أحدث.
- أصبح البحث الشامل يستخدم `searchJob` و`searchRequestId` نفسهما مع البحث العادي، ويلغي الطلب السابق عند تغيير الاستعلام أو المصدر.
- إزالة التكرار أصبحت حسب `(sourceId, id)` حتى لا يحذف عملان من مصدرين مختلفين إذا تساوى معرّفهما.
- اختبار يغطي بقاء نفس ID من مصدرين مختلفين وإزالة التكرار داخل المصدر الواحد.

## فحص توقيع APK
- أضيفت خطوة CI تستخدم `keytool` و`apksigner` لمقارنة بصمة شهادة keystore مع بصمة APK الناتج، قبل حذف ملف PKCS12 المؤقت.
- لا تُطبع كلمات المرور أو بيانات المفتاح في السجلات.

## التحقق
- التطبيق commit: `43cd7a14ba7376f976f69d437674ab397fcb7eee`
- الفرع: `fix/room-storage`
- CI: قيد التنفيذ وقت كتابة هذا الملف.
- لم يتم نشر Release حتى الآن.
- اختبار حقيقي على الهاتف مع مصادر حية لم يتم بعد.


## محاولة CI الأولى والتصحيح
- Run `44114123797` فشل في اختبار إلغاء التحقق فقط.
- الاختبار كان يعيد استخدام hostname ثابتًا استُخدم في اختبار سابق؛ وبما أن cache أصبح keyed by host، انتقلت نتيجة التحقق بين الاختبارين خلال نافذة 30 ثانية.
- تم تغيير hostname اختبار الإلغاء إلى قيمة عشوائية مستقلة، مع الإبقاء على سلوك production كما هو.
- تم إصلاح shell fingerprint parsing في workflow لاستخدام `sed`/`tr` بشكل صحيح.
- corrective commit: `efca138c8406a0c7d2c35d2051dc1bbf27d11946`.
- نتيجة CI الجديدة: قيد التنفيذ وقت التوثيق.


## فصل جلسات النطاقات المتزامنة
- اكتُشف أن وجود `pending` واحد بلا hostname يسمح لطلب من نطاق آخر أن ينتظر نفس التحدي ويعتبره ناجحًا، رغم أن كوكي النطاق الأول لا تثبت صلاحية النطاق الثاني.
- أصبح لكل طلب `sourceKey` مبنيًا على hostname؛ الطلبات من نفس النطاق تنتظر نتيجة الجلسة نفسها، والطلبات من نطاقات مختلفة تنتظر انتهاء الجلسة النشطة ثم تبدأ جلسة تحقق مستقلة.
- أضيف اختبار `concurrentDifferentHostsAreVerifiedSequentially`.
- commit: `49405a420792728ab482cf88bd61b8577ec438a6`.
- CI run `44115444539` أُلغي بسبب تعليق Setup Android SDK قبل الاختبارات؛ لا توجد نتيجة اختبار لهذا التشغيل.
- CI الجديد `44116165269` قيد التنفيذ.


## CI signing verification diagnosis and correction — 2026-11-18
- Workflow run `44116165269` completed with failure after `Run unit tests` and `Build Debug APK` both succeeded.
- The failing step was `Verify APK signature matches stable key`; publishing and schema commit were skipped. No `v1.5.0` Release was created.
- Job log showed `apksigner` output as `V2 Signer: certificate SHA-256 digest: ...`, while the workflow parser searched for `Signer #1 certificate SHA-256 digest: ...`. The extracted actual fingerprint was therefore empty; the run does not establish that the configured keystore is wrong.
- Corrected only the output parser to match the stable `certificate SHA-256 digest:` portion, preserving normalization and comparison against the keystore certificate.
- Application workflow commit: `b4360e73faba53e74e29d3ad3aa38466701091e7` on `fix/room-storage` (`fix(ci): parse apksigner certificate fingerprint`).
- The corrected workflow must still pass a fresh CI run before the signing key or `1.5.0` APK can be considered verified.


## 2026-11-18 — استكمال الجرد وتصحيح موثوقية الإحصاءات وخطة الإصلاح

- طلب المستخدم استكمال جرد الإنجليزية ومصادر 18+، ثم إصلاح جميع المحللات المعطلة، بما فيها التي لا يذكر تعليق `@Broken` سبب تعطلها.
- ترتيب الإصلاح المطلوب: الأشهر والأكثر استخدامًا أولًا ثم الأقل شهرة، مع فصل كل مجموعة وعدم خلط مصادر اللغات والفئة العمرية:
  1. العربية العامة (غير 18+).
  2. الإنجليزية العامة (غير 18+).
  3. 18+، وتقسم داخليًا إلى عربية وإنجليزية.
- يجب تحديد سبب التعطل من أدلة فعلية: كود المحلل، النطاق والتحويلات، استجابة الموقع، بنية HTML/API، اختبارات upstream، وتقارير موثوقة. غياب سبب في التعليق لا يعني أن السبب مجهول أو أن المحلل غير قابل للإصلاح.
- لا تُرفع علامة `@Broken` ولا يُفعّل Parser إلا بعد اختبارات وظائف فعلية مناسبة: البحث، تفاصيل العمل، قائمة الفصول، واستخراج الصفحات/الصور بحسب عقد المصدر. فشل وظيفة واحدة يبقي المصدر غير جاهز.
- لا يجوز استخدام مجرد فتح الموقع أو HTTP 200 كدليل نجاح Parser.
- يجب فصل حالة كل مصدر إلى: موجود upstream، له Parser implementation، معلّم Broken، مدرج في كتالوج التطبيق، قابل للإنشاء/التوجيه، مفعّل/معطّل، آخر فحص حي، ونتائج اختبارات Parser.
- **تصحيح موثوقية الجرد السابق:** الأعداد والقوائم التي ظهرت في رد 2026-11-18 السابق لا تعتبر مرجعًا نهائيًا حتى تُعاد مطابقتها آليًا من ملفات المصدر المكتملة. كانت هناك محاولات جلب دفعات وعمليات حفظ بأسماء ملفات متكررة؛ لذلك لا يجوز الجزم بإحصاء 468/153/105 أو اكتمال قوائم الأسماء دون إعادة تدقيق شامل. يجب تسجيل عدد الملفات التي جرى جلب محتواها بنجاح وعدد الملفات التي تعذر جلبها، ثم حساب الإحصاءات من الملفات المكتملة فقط.
- لا تغيير في تفاصيل الأعمال والفصول والصفحات أو قارئها؛ يقتصر العمل عليها على اختبارات عدم الانحدار التي لا تغيّر السلوك.
- لا تخلط `PROCOMIC` / Manga Pro مع `PROCHAN` / ProChan. يلزم التحقق المستقل من النطاق والهوية والتحويلات قبل أي توحيد.
- لم يبدأ أي تعديل كود في التطبيق ضمن هذا التحديث. الخطوة التالية: إعادة بناء جرد موثوق، ثم معالجة المجموعة الأولى حسب الشهرة مع اختبارات لكل Parser، وتوثيق كل تعديل في مستودع المعرفة في نفس جولة التنفيذ.
- المستودع التعليمي: `osamahosny2/iruma-engineering-handbook`، الفرع `main`. سجل المحادثة الإلزامي هو `CONVERSATION_HISTORY.md`.


## 2026-11-18 — تشخيص Hijala وترتيب الإصلاح

- أُعيد فحص ملفات الإنتاج كاملة: 470 ملف Kotlin ضمن مسارات اللغة العربية والإنجليزية (403 en، 67 ar). تحتوي على 468 تعريف Parser (401 en، 67 ar)؛ ملفان بلا annotation. ضمن التعريفات: 153 `@Broken`، و105 `ContentType.HENTAI`. لا تخلط عدد ملفات Kotlin بعدد تعريفات Parser.
- المجموعة الأولى: العربية العامة غير 18+، مرتبة بحسب الانتشار/النشاط المثبت. تليها الإنجليزية العامة غير 18+، ثم 18+ مقسمة إلى عربية وإنجليزية.
- أول مصدر عربي بحسب الانتشار الذي أمكن توثيقه: Hijala (`HIJALA`). تقدير زيارات سبتمبر 2026 نحو 1.56M بحسب Semlogin ToolsDuniya، والموقع وصفحات العمل والفصل ما زالت مفهرسة وتعرض محتوى حديثًا. المصادر: https://semlogin.toolsduniya.com/website/hijala.com/overview/?source=trending-websites ، https://hijala.com/lookism/ ، https://hijala.com/lookism-617/.
- السبب الجذري: upstream `Hijala.kt` عند الالتزام المثبت يمدد `ZeistMangaParser`؛ هذا الأساس يطلب Blogger feed `/feeds/posts/default/-/...` ويعتمد JSON `feed.entry`. الموقع الحالي يعرض صفحات أعمال مثل `/lookism/` وفصولًا مثل `/lookism-617/` وصورًا تحت `/wp-content/uploads/manga/`. إذًا التنفيذ القديم لا يطابق بنية الموقع الحالية. لا تُعامل هذه الحالة ككابتشا.
- مرجع خارجي مساعد قديم: `MohammedAdel224/Manga-Favorate-List` ملف `manga/hijala.py` يستخدم `div#chapterlist`, `li[data-num]`, `span.chapterdate`. هذا مرجع قديم وغير كافٍ لإثبات selectors الحالية؛ يجب التحقق من DOM قبل اعتماده.
- عائق في التطبيق: `KotatsuSourceEngine.getCatalogSources()` لا يحاول local override عند `parserSource.isBroken`، و`resolveParser()` يرفض `candidate.isBroken` قبل محاولة `LocalSourceParserRegistry.create()`. هذا يمنع إصلاح أي Parser معطّل عبر override محلي حتى لو أُضيف له implementation. يجب إصلاح أولوية local override واختبارها ضمن تنفيذ أول مجموعة.
- لم يُعدّل كود التطبيق بعد في هذه الجولة. لا تكتب selectors تخمينية. قبل اعتماد Hijala يجب التحقق من DOM الحالي لصفحة العمل والفصل، ثم اختبار البحث والتفاصيل والفصول والصفحات والصور. لا تُزل `@Broken` حتى تنجح الاختبارات.
- لا تعديل لمسارات تفاصيل الأعمال والفصول والصفحات المشتركة أو قارئ التطبيق؛ الإصلاح المحلي للمصدر يجب ألا يغير تلك المسارات. لا تخلط `PROCOMIC` و`PROCHAN`.


## 2026-11-20 — إعادة فحص عائق Local Parser Override

- أُعيد الوصول إلى مشروع التطبيق الخاص عبر اتصال GitHub Composio النشط، الفرع `fix/room-storage`، HEAD `b4360e73faba53e74e29d3ad3aa38466701091e7`.
- تمت قراءة الملفات الحالية مباشرة: `app/src/main/java/com/example/connector/kotatsu/KotatsuSourceEngine.kt`، `source-parsers/src/main/kotlin/com/example/connector/kotatsu/parsers/LocalSourceParserRegistry.kt`، `app/src/main/java/com/example/connector/ConnectorEngine.kt`، و`app/src/main/java/com/example/data/SourceCatalogActivation.kt`.
- تأكد من الكود أن `getCatalogSources()` يستخدم `if (parserSource.isBroken) null else context.newParserInstance(...)`، ثم يحسب `unavailable` باستخدام `parserSource.isBroken` دون اعتبار وجود local override. هذا يمنع ظهور المصدر المعطّل على أنه قابل للتشغيل حتى لو أُضيف له Parser محلي.
- تأكد أن `resolveParser()` يرفض `candidate == null || candidate.isBroken` ويضيف المصدر إلى `missingParserCache` قبل استدعاء `LocalSourceParserRegistry.create(...)`. لذلك لا يمكن حاليًا لأي local implementation تجاوز `@Broken` upstream.
- `LocalSourceParserRegistry` الحالي يعيد `IrumaAzoramoonParser` للمعرّف `AZORAMOON` فقط، ولا يحتوي على `HIJALA`.
- الاستنتاج: يجب جعل وجود local override أسبق من فحص `isBroken` في الكتالوج والحل، وإضافة API واضح مثل `supports(sourceId)`، ثم اختبارات تثبت أن المصدر upstream المعلّم Broken يصبح قابلًا للإنشاء فقط عند وجود override محلي فعلي. لا يكفي أن تكون `supports` صحيحة إذا فشل إنشاء Parser.
- لا كود تطبيق عُدّل ولا اختبارات شُغّلت في هذه الجولة. بقيت الخطوة التالية تنفيذ إصلاح الأولوية واختباراته ضمن المهمة 3، ثم بناء Parser Hijala بعد فحص DOM الحالي.
- تعذر استخدام Browserless في هذه الجلسة؛ عُرض TinyFish كبديل متاح للمتصفح السحابي، لكن لم يتم ربطه بعد. لا يجوز الادعاء بأنه متصل أو تنفيذ فحص حي به قبل اكتمال الربط من المستخدم.
- لم يتغير ترتيب المجموعات أو حدود النطاق: العربية العامة أولًا، الإنجليزية العامة ثانيًا، ثم 18+ مع فصل العربية والإنجليزية. لا تعديل لمسارات التفاصيل والفصول والصفحات المشتركة أو القارئ، ولا خلط بين `PROCOMIC` و`PROCHAN`.


## 2026-11-20 — نتيجة فحص Hijala بالمتصفح السحابي

- تم ربط TinyFish بنجاح وأصبح متاحًا في هذه الجلسة.
- فحصت صفحة القائمة `https://hijala.com/manga/list-mode/` وصفحة العمل `https://hijala.com/lookism/` في جلستين مستقلتين.
- كلتا الصفحتين توقفتا عند Cloudflare verification challenge قبل محتوى الموقع؛ إحداهما تعرض "Verify you are human" والأخرى "Performing security verification". لم تتم محاولة حل أو تجاوز التحدي.
- لا توجد selectors حالية مؤكدة لقائمة الأعمال أو الفصول أو صور القارئ. لا يجوز نسخ selectors من scraper قديم أو اعتمادها دون إثبات أنها ما زالت مستخدمة.
- يضاف هذا كعائق وصول من بيئة الفحص، وليس دليلًا على أن Cloudflare هو السبب الوحيد لتعطل Parser. سبب عدم التوافق البنيوي مع `ZeistMangaParser` يظل قائمًا حسب مراجعة كود upstream وصفحات الموقع المفهرسة.
- لا تعديل كود تطبيق، ولا إزالة `@Broken`، ولا تغيير إصدار.

## 2026-11-20 — الإجراء البرمجي التالي المقترح

- إصلاح local override ordering: تقديم `LocalSourceParserRegistry` على فحص `parserSource.isBroken` في `getCatalogSources()` و`resolveParser()`.
- إضافة `supports(sourceId)` أو API مكافئ، لكن لا يُعتبر المصدر صالحًا إلا بعد نجاح إنشاء parser محلي فعلي.
- اختبارات: المصدر العادي غير Broken يستمر بالـupstream parser؛ مصدر Broken له override محلي صالح يصبح متاحًا؛ مصدر Broken دون override يبقى غير متاح؛ وإذا رمى إنشاء override استثناءً يبقى غير متاح.
- الحفاظ على shared details/chapter/pages/reader paths دون تعديل؛ الاختبارات تقيس فقط الإنشاء/الكتالوج والتوجيه.
- لم يُنفذ التعديل بعد في هذه الجولة.


## 2026-11-20 — تأكيد ثانٍ لحاجز Cloudflare على صفحة Lookism

- أعادت محاولة `TinyFish.fetch_content` المباشرة إلى `https://hijala.com/lookism/` الخطأ `bot_blocked`.
- هذا يتفق مع نتيجة المتصفح الحي السابقة التي عرضت `Performing security verification` من Cloudflare قبل تحميل محتوى صفحة العمل.
- لا تتوفر حاليًا بيانات DOM للعنوان أو قائمة الفصول أو صور القارئ، ولا توجد selectors مؤكدة. لم نحاول حل التحدي أو تجاوزه.
- لا يُستنتج من هذا أن Cloudflare هو السبب الوحيد لتعطل Parser؛ عدم توافق محلل Blogger القديم مع البنية الحالية ما زال سببًا بنيويًا مستقلًا بحسب المراجعة السابقة.
- لم يتغير كود التطبيق ولا رقم الإصدار. الخطوة التالية داخل المهمة 3 تظل إصلاح أولوية Local Parser Override واختبارها قبل بناء Parser Hijala.
