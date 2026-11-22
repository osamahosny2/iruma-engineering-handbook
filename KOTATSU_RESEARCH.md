

## نتائج دراسة إضافية — 2026-11-22

### كود Kotatsu الأصلي وKotatsu-Redo
- تمت قراءة AndroidCookieJar وPreferencesCookieJar وNetworkModule وCloudFlareInterceptor وCaptchaHandler وCaptchaContinuationClient وWebViewExecutor وMangaLoaderContextImpl من Kotatsu الأصلي وKotatsu-Redo.
- AndroidCookieJar في المرجعين يستخدم CookieManager.getCookie(url) ويحوّل أزواج request-header إلى OkHttp Cookie. PreferencesCookieJar يخزن الكوكيز الدائمة في SharedPreferences، ويحتفظ بغير الدائمة في الذاكرة. NetworkModule يستخدم AndroidCookieJar عند توفر WebView وPreferencesCookieJar كبديل عند تعذر WebView؛ لا توجد مزامنة ثنائية الاتجاه كاملة بين الاثنين في المسار المعتاد.
- بعض مسارات Kotatsu تحاول حل CAPTCHA باستخدام WebView مخفي/آلي. هذا السلوك لا يُنقل إلى إيروما لأن المطلوب هو إتمام المستخدم للتحقق البشري داخل WebView، لا تجاوز التحدي.

### Mihon وCloudflare
- تمت مراجعة Mihon كمرجع Tachiyomi الحالي، بما في ذلك AndroidCookieJar وCloudflareInterceptor ودليل Troubleshooting.
- دليل Mihon يطلب فتح المصدر داخل WebView وإكمال CAPTCHA يدويًا، ويذكر أن بعض مستويات Cloudflare قد تستمر في إعادة التحميل. توثيق Cloudflare Turnstile يذكر ثبات User-Agent، وتمكين JavaScript وDOM storage والكوكيز، والسماح باتصالات challenges.cloudflare.com.
- توثيق Cloudflare يحدد cf-mitigated: challenge كإشارة موثوقة لاكتشاف استجابة Challenge Page.

### Android وOkHttp
- CookieManager.getCookie(url) يعيد أزواج key=value بتنسيق Cookie request-header فقط؛ لا يمكن استعادة كل خصائص Set-Cookie الأصلية منه.
- setCookie(url, value, callback) غير متزامنة ويعيد callback نتيجة النجاح، وflush() يكتب الكوكيز المتاحة عبر getCookie إلى التخزين الدائم.
- OkHttp CookieJar مسؤول عن سياسة قبول الكوكيز وتخزينها وإعادتها للطلب المناسب. يجب احترام host-only/domain وpath وSecure وانتهاء الصلاحية والاستبدال والحذف.
- لا يجوز ادعاء أن getCookie يسمح باسترجاع HttpOnly أو SameSite أو expiry أو Domain/Path الأصلية كاملة.

### GitHub Issues وReddit وX
- Mihon issue #2455 يوثق أن حذف Cookie يتطلب تطابق domain وpath الأصليين.
- Mihon issue #1722 يوثق أن استيراد Cookies من متصفح خارجي لم يكن كافيًا وحده لحل Cloudflare في حالة صاحب البلاغ.
- Keiyoushi issue #20975 يوثق حالة كان فيها WebView يفتح بينما يفشل القارئ الأصلي مع Cloudflare.
- Reddit يقدم تجارب فردية حول WebView وUser-Agent وإعادة تشغيل التطبيق؛ لا تُعامل هذه التجارب كضمان تقني.
- تمت محاولة البحث العام في X وفتح صفحة البحث، لكن لم تُسترجع منشورات X قابلة للتحقق. يظل هذا جزءًا غير مكتمل من البحث.

### أثر النتائج على التصميم
- المطلوب مخزن OkHttp داخلي مستقل ودائم، مع حفظ كامل خصائص Cookies القادمة من Set-Cookie متى وفرها OkHttp.
- بعد التحقق اليدوي في WebView، يمكن قراءة request-scoped cookie pairs من CookieManager، لكن يجب توثيق فقدان خصائصها وعدم الادعاء باستعادة البيانات الأصلية كاملة.
- يجب انتظار نتيجة callback عند نقل Cookies إلى WebView، ثم flush، ومنع التكرار والتسريب بين النطاقات.
- لا يعتمد التصميم النهائي على Room أو DataStore قبل فحص اعتماديات المشروع واختيار الأنسب لها.
- لا توجد تعديلات برمجية منفذة حتى الآن في جلسة 2026-11-22.
