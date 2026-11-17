# المهمة 2 — فصل قراءة الصفحات المحلية عن واجهة المستخدم

## المشكلة المؤكدة
كان `MangaViewModel.downloadReaderPageImage` ينشئ `File(localPath)` ويقرأ الملف مباشرة عبر `readBytes()` على `Dispatchers.IO`. هذا يضع الوصول إلى نظام الملفات داخل طبقة العرض، ويتجاوز طبقة البيانات، ولا يقيّد المسار إلى مجلد تنزيلات التطبيق.

## التعديل
- أضيف `readDownloadedPageImageBytes(localPath)` إلى عقد `MangaAppRepository`.
- `MangaRepository` يمرر العملية إلى `DownloadManager`.
- `DownloadManager` ينفذ القراءة على `Dispatchers.IO`، ويحوّل المسارات إلى canonical files، ويرفض الملفات خارج مجلد `filesDir/manga_downloads`، ويرفض الملفات المفقودة أو الفارغة.
- `MangaViewModel` لم يعد يستورد `File` أو `Dispatchers` أو `withContext` لهذا الغرض؛ يطلب القراءة من repository.
- عند فشل قراءة الملف المحلي، يبقى مسار التنزيل من المصدر متاحًا كما كان سابقًا.

## الاختبار
- أضيف اختبار Robolectric للتحقق من قراءة bytes صحيحة من ملف داخل مجلد التنزيلات.
- الاختبار يرفض ملفًا خارج المجلد، ومسار `../`، والمسار الفارغ.
- CI: https://github.com/osamahosny2/iruma_manga_v2/actions/runs/44112248004
- commit: `96ec0369446839814bd9de96cc113284ffaafd2c`
- حالة التحقق وقت التوثيق: قيد التنفيذ؛ لا تُسجل النتيجة حتى تظهر حالة workflow النهائية.

## ما لم يتغير
- لم تُنقل جدولة التنزيلات أو استمراريتها إلى هذا التعديل؛ ذلك ضمن المهمة 6.
- لم تتغير واجهة المستخدم أو سلوك القارئ أو بنية Room.
