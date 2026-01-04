# 211.3. مدیریت تحویل ایمیل از راه دور

## **211.3 مدیریت تحویل ایمیل از راه دور**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند daemonهای POP و IMAP را نصب و پیکربندی کنند.

**نواحی کلیدی دانش:**

* پیکربندی و مدیریت Dovecot برای IMAP و POP3
* پیکربندی پایه TLS برای Dovecot
* آگاهی از Courier

**اصطلاحات و ابزارها:**

* /etc/dovecot/
* dovecot.conf
* doveconf
* doveadm

## POP3 و IMAP چیست؟

POP3 و IMAP دو پروتکل رایج برای دسترسی remote به mailbox هستند. معمولاً در کنار MTA (مثل Postfix/Exim) یک سرویس IMAP/POP3 اجرا می‌شود تا کاربران بتوانند با mail client یا webmail به mailbox خود دسترسی داشته باشند.

* **POP3**: معمولاً ایمیل‌ها را دانلود می‌کند (می‌تواند طوری تنظیم شود که کپی روی سرور بماند)
* **IMAP**: ایمیل‌ها را روی سرور نگه می‌دارد و client از راه دور آن‌ها را می‌خواند

به طور کلی IMAP برای دسترسی از چند دستگاه مناسب‌تر است، اما چون داده روی سرور می‌ماند باید برای backup و مدیریت فضای دیسک برنامه داشت.

## Courier (آگاهی)

Courier مجموعه‌ای از نرم‌افزارهای ایمیل است که می‌تواند IMAP/POP3 ارائه دهد. برای LPIC-2 کافی است بدانید وجود دارد و در برخی سناریوها به عنوان IMAP server در کنار MTAهای دیگر استفاده می‌شود.

## Dovecot

Dovecot یکی از رایج‌ترین سرویس‌ها برای ارائه IMAP و POP3 است.

### نصب

* Debian/Ubuntu:

```bash
apt update
apt install dovecot-imapd dovecot-pop3d
systemctl enable --now dovecot
```

* RHEL/CentOS:

```bash
yum install dovecot
systemctl enable --now dovecot
```

### فایل‌های پیکربندی

پیکربندی معمولاً زیر این مسیر است:

* `/etc/dovecot/`

و فایل اصلی می‌تواند `dovecot.conf` باشد یا مجموعه‌ای از فایل‌های include.

### فعال/غیرفعال کردن پروتکل‌ها

در تنظیمات Dovecot مشخص می‌کنیم کدام پروتکل‌ها فعال باشند:

```conf
protocols = imap pop3
```

### mailbox location

بسته به اینکه سیستم از mbox یا maildir استفاده کند، محل mailbox باید درست تنظیم شود.

### TLS (پایه)

برای امن کردن ارتباط IMAP/POP3 می‌توان TLS را فعال کرد:

```conf
ssl = required
ssl_cert = </etc/ssl/certs/mail.crt
ssl_key = </etc/ssl/private/mail.key
```

(مسیر فایل‌ها بسته به توزیع متفاوت است.)

### بررسی تنظیمات

نمایش تنظیمات نهایی:

```bash
doveconf
```

### مدیریت و عیب‌یابی

`doveadm` ابزار مدیریتی dovecot است.

نمونه:

```bash
doveadm who
```

## پورت‌های رایج

* IMAP: 143
* IMAPS: 993
* POP3: 110
* POP3S: 995

نکته: در کنار پیکربندی سرویس، firewall هم باید اجازه دسترسی به پورت‌های مورد نیاز را بدهد.
