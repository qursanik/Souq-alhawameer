# دليل إعداد التحقق عبر الرسائل النصية (SMS)
# SMS Verification Setup Guide

## المتطلبات الأساسية / Prerequisites

لتفعيل خدمة التحقق عبر الرسائل النصية في Firebase، اتبع الخطوات التالية:

### 1. تفعيل Phone Authentication في Firebase Console

1. افتح [Firebase Console](https://console.firebase.google.com/)
2. اختر مشروعك: **souq-alhawameer**
3. انتقل إلى: **Authentication** → **Sign-in method**
4. ابحث عن **Phone** في قائمة موفري المصادقة
5. اضغط على **Phone** ثم **Enable**
6. احفظ التغييرات

### 2. إعداد Test Phone Numbers (للاختبار - اختياري)

أثناء التطوير، يمكنك إضافة أرقام اختبارية لتجنب استهلاك رصيد الرسائل:

1. في صفحة **Authentication** → **Sign-in method**
2. انتقل لأسفل إلى **Phone numbers for testing**
3. أضف أرقاماً مع رموز تحقق ثابتة (مثال: `+966501234567` مع رمز `123456`)

### 3. التحقق من reCAPTCHA Settings

Firebase تستخدم reCAPTCHA للحماية من الاستخدام غير المصرح به:

- للبيئة المحلية: Firebase يوفر reCAPTCHA تلقائياً
- للإنتاج: تأكد من إضافة domain الخاص بك في إعدادات Firebase Authentication

### 4. الحصص المجانية / Free Tier Quotas

Firebase تقدم حصة مجانية من الرسائل النصية:
- **10 SMS يومياً** في الخطة المجانية (Spark Plan)
- بعد تجاوز الحد، ستحتاج للترقية لخطة Blaze (الدفع حسب الاستخدام)

## استكشاف الأخطاء / Troubleshooting

### الخطأ: "auth/operation-not-allowed"
**الحل:** تأكد من تفعيل Phone provider في Firebase Console

### الخطأ: "auth/quota-exceeded"
**الحل:** تم تجاوز الحد المجاني للرسائل. انتظر حتى اليوم التالي أو قم بالترقية لخطة Blaze

### الخطأ: "auth/invalid-phone-number"
**الحل:** 
- تأكد من أن رقم الجوال بالصيغة الدولية (مثال: `+966501234567`)
- يجب أن يبدأ الرقم السعودي بـ `05` (10 أرقام)
- التطبيق يحول الأرقام تلقائياً من `05...` إلى `+9665...`

### الخطأ: "auth/captcha-check-failed"
**الحل:** أعد تحميل الصفحة وحاول مرة أخرى. قد تكون مشكلة مؤقتة في reCAPTCHA

### الخطأ: "auth/timeout" أو "Timeout"
**الحل:** 
- تحقق من اتصالك بالإنترنت
- التطبيق يعيد المحاولة تلقائياً 3 مرات
- reCAPTCHA timeout تم رفعه إلى 120 ثانية

### الرسالة لا تصل (SMS not received)
**الحل:**
1. تحقق من صحة رقم الجوال
2. تأكد من أن Phone Authentication مفعّل في Firebase
3. تحقق من عدم تجاوز الحصة المجانية
4. انتظر بضع دقائق (قد يتأخر وصول الرسالة)
5. تحقق من console في المتصفح (F12) للحصول على تفاصيل الخطأ
6. تأكد من أن شبكة الجوال تسمح باستقبال الرسائل من أرقام دولية

## ملاحظات مهمة / Important Notes

- **التكرار (Retry Logic):** التطبيق يعيد المحاولة تلقائياً 3 مرات مع تأخير متزايد (2 ثانية، 4 ثوان)
- **التنسيق التلقائي:** الأرقام السعودية يتم تحويلها تلقائياً من `05...` إلى `+9665...`
- **السجلات (Logs):** افتح Developer Console (F12) لمشاهدة تفاصيل عملية إرسال SMS
- **الأمان:** Firebase reCAPTCHA يحمي من الاستخدام الآلي للخدمة

## روابط مفيدة / Useful Links

- [Firebase Phone Authentication Docs](https://firebase.google.com/docs/auth/web/phone-auth)
- [Firebase Console](https://console.firebase.google.com/)
- [Firebase Pricing](https://firebase.google.com/pricing)

## الدعم الفني / Support

إذا استمرت المشكلة بعد اتباع هذه الخطوات، يرجى:
1. فتح Developer Console (F12) ونسخ رسالة الخطأ الكاملة
2. التحقق من Firebase Console → Authentication → Users لمعرفة إذا تم إنشاء المستخدم
3. مراجعة Firebase Console → Authentication → Usage للتأكد من عدم تجاوز الحصة
