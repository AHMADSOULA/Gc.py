# تشغيل BOT_AHMED2027 على Fly.io عبر GitHub

## 1) GitHub
ارفع محتويات هذا المجلد إلى جذر مستودع GitHub (لا ترفع ملف `.env` ولا أي HAR أو Cookies أو مفاتيح خاصة).

## 2) إعداد Fly.io
أنشئ تطبيق Fly بالاسم الموجود في `fly.toml` أو غيّر `app = "bot-ahmed2027"` إلى اسم تطبيقك الفعلي.
من الطرفية داخل مجلد المشروع نفّذ:

```bash
fly auth login
fly launch --no-deploy --copy-config
```
إذا كان التطبيق موجودًا مسبقًا فلا تنشئ تطبيقًا جديدًا؛ تأكد أن `app` في fly.toml يطابق الاسم الحالي.

## 3) الأسرار
ضع توكن تيليجرام كـ secret ولا تكتبه في GitHub:

```bash
fly secrets set BOT_TOKEN="YOUR_BOT_TOKEN" ADMIN_ID="7119011124" -a bot-ahmed2027
```

## 4) تخزين دائم لقاعدة البيانات وملف المتصفح
قاعدة البيانات في `/data` تحتاج Fly Volume حتى لا تضيع عند استبدال الجهاز:

```bash
fly volumes create bot_data --region ams --size 1 -a bot-ahmed2027
```
أضف إلى `fly.toml`:

```toml
[mounts]
  source = "bot_data"
  destination = "/data"
```
ثم أعد النشر.

## 5) النشر والتشغيل

```bash
fly deploy -a bot-ahmed2027
fly scale count 1 -a bot-ahmed2027
fly status -a bot-ahmed2027
fly logs -a bot-ahmed2027
```

## ملاحظات إصلاح الأعطال
- هذا المشروع بوت Telegram polling وليس خادم ويب؛ لذلك لا يحتاج `http_service` ولا منفذ 8080 ولا عنوان IPv4/IPv6 عام. عدم الاستماع على 8080 ليس خطأ لهذا النوع من البوتات.
- أزل أي `[http_service]` أو `[[services]]` قديمة إذا كانت موجودة في إعداد Fly؛ ملف fly.toml المرفق لا يعرّف خدمات عامة.
- شغّل نسخة واحدة فقط من البوت لكل توكن لتجنب `Conflict: terminated by other getUpdates request`.
- قاعدة البيانات والمسارات تستخدم `/data` بعد تركيب الـ Volume. لا تستخدم `/tmp` إذا أردت الاحتفاظ بالبيانات بين عمليات النشر.
- PAGE_TIMEOUT و NAV_TIMEOUT بالثواني، لأن config.py يحولها إلى milliseconds.
- إذا ظهر خطأ من Playwright/Chromium، راجع السجلات؛ صورة Docker الرسمية تتضمن متصفح Playwright المطابق للإصدار 1.48.0.
