# Iruma Engineering Handbook

مرجع المعرفة والهندسة لمشروع **ايروما منجا**. هذا المستودع منفصل عن كود التطبيق: يحتوي على المنهج، القرارات، نتائج البحث، حالة المهام، وقواعد العمل التي يجب أن يقرأها أي ذكاء اصطناعي أو مساهم قبل اقتراح تعديل أو تنفيذ عمل.

## ابدأ من هنا
1. اقرأ [AI_INSTRUCTIONS.md](AI_INSTRUCTIONS.md) أولًا.
2. اقرأ [HOW_TO_WORK_WITH_THE_USER.md](HOW_TO_WORK_WITH_THE_USER.md) لفهم آلية الأسئلة والموافقات.
3. اقرأ [PROGRAMMING_METHOD.md](PROGRAMMING_METHOD.md) عند بدء أي مهمة برمجية.
4. اقرأ [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) و[PROJECT_STATUS.md](PROJECT_STATUS.md) لفهم التطبيق وحالته.
5. اقرأ [DECISIONS.md](DECISIONS.md) قبل اقتراح تغيير سبق مناقشته.

## المستودعات
- **كود التطبيق:** https://github.com/osamahosny2/iruma_manga_v2
- **فرع العمل المسموح للتطبيق:** `fix/room-storage`
- **مستودع المعرفة الحالي:** https://github.com/osamahosny2/iruma-engineering-handbook
- المستودع القديم `iruma_manga` ممنوع تعديله.

## مبدأ الفصل
- محللات المصادر (Parsers) والكود التنفيذي الذي يحتاجه التطبيق تكون داخل مستودع التطبيق، مدمجة فيه وتعمل ضمن التطبيق.
- هذا المستودع يوثق تصميم المحللات، البحث، القواعد، القرارات، الاختبارات، وطريقة العمل؛ لا يُستخدم كمكتبة تشغيلية يحتاج التطبيق إلى تنزيلها من GitHub أثناء الاستخدام.

## حالة المهام
الحالة التفصيلية للمهام الثمانية في [PROJECT_STATUS.md](PROJECT_STATUS.md). لا تعتبر نجاح CI دليلًا على نجاح التطبيق على الهاتف.

## تحديث المعرفة
كلما تم بحث أو تعديل أو اختبار أو اتخاذ قرار جديد:
- حدّث الملف المختص، لا تكتفِ بكتابة ملخص في المحادثة.
- حدّث [PROJECT_STATUS.md](PROJECT_STATUS.md) عند تغير حالة مهمة.
- أضف القرارات المؤثرة إلى [DECISIONS.md](DECISIONS.md).
- سجّل التغييرات المهمة في [CHANGELOG.md](CHANGELOG.md).
- ميّز دائمًا بين: مقترح، قيد البحث، منفذ، اجتاز الاختبار الآلي، واختُبر عمليًا على الهاتف.

لا تضع كلمات مرور أو Tokens أو مفاتيح API أو بيانات شخصية حساسة في هذا المستودع.


## أحدث دراسة هندسية
- [تقرير دراسة محرك المصادر 2026-11-10](docs/SOURCE_ENGINE_RESEARCH_2026-11-10.md): مراجعة فعلية لكود التطبيق ومكتبة Kotatsu، نتائج الفحص، فرضيات الأسباب الجذرية، ومخاطر نقل المكتبة إلى Module محلي.


## تنفيذ محرك المصادر
- [سجل تنفيذ محرك المصادر 2026-11-10](docs/SOURCE_ENGINE_IMPLEMENTATION_2026-11-10.md): commits، تغييرات الوحدة المحلية ومحلل Azora، حالة CI، وخطة Manga Pro.

## Current source-engine research

- [Manga Pro / ProComic reader research and implementation plan](PROCOMIC_READER_RESEARCH.md)
