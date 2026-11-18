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
