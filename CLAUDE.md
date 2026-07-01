# سامانه آمار ورزشی

## پروژه
یک فایل HTML تک‌صفحه‌ای (3320 خط) — بدون build tool، بدون npm، بدون framework.
فایل اصلی: `سامانه_آمار_ورزشی_v8_نهایی.html`

## معماری
- **زبان:** Vanilla JS + HTML/CSS
- **جهت:** RTL (فارسی)
- **ذخیره‌سازی:** localStorage (کلید: `sportStats`)
- **شی اصلی:** `state` — تمام داده برنامه اینجاست
- **صفحات:** `showPage(id)` — سوئیچ بین بخش‌ها
- **اعلان:** `toast(msg)` — پیام کوتاه
- **نشان‌ها:** `updateBadges()` — بروزرسانی شمارنده‌ها

## صفحات موجود
`dashboard` | `national` | `championship` | `leagues` | `education` | `history` | `settings`

## خروجی
- Excel: XLSX-Lite داخلی (بدون کتابخانه خارجی)
- Word: Blob + msword MIME
- PDF: `window.open` + `print()`

## قوانین کار
- **فقط یک فایل را ویرایش کن** — همان HTML اصلی
- **بدون کامنت اضافه** در کد
- **بدون تست** — پروژه تست ندارد
- **بدون import/require** — همه چیز inline است
- قبل از تغییر: فقط بخش مرتبط را بخوان، نه کل فایل
- سازنده: امین عزیزی مقدم
