# 212.2. ایمن‌سازی سرورهای FTP

## **212.2 ایمن‌سازی سرورهای FTP**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند یک FTP server را برای دانلود/آپلود anonymous پیکربندی کنند. این هدف شامل اقداماتی است که در صورت اجازه دادن به anonymous upload باید انجام شود و همچنین پیکربندی دسترسی کاربران.

**نواحی کلیدی دانش:**

* فایل‌های پیکربندی، ابزارها و utilityهای Pure-FTPd و vsftpd
* آگاهی از ProFTPd
* درک تفاوت اتصال‌های Passive و Active در FTP

**اصطلاحات و ابزارها:**

* vsftpd.conf
* گزینه‌های مهم command line در Pure-FTPd

## FTP چیست؟

FTP (File Transfer Protocol) یک پروتکل قدیمی و رایج برای انتقال فایل بین client و server روی شبکه است. نکته مهم این است که FTP به صورت پیش‌فرض امن نیست، چون credentialها (نام کاربری/رمز عبور) و داده‌ها را بدون encryption منتقل می‌کند.

اگر مجبور به استفاده از FTP هستید، بهتر است آن را با SSL/TLS (FTPS) پیکربندی کنید. در غیر این صورت معمولاً استفاده از جایگزین‌های امن مثل SFTP (روی SSH) توصیه می‌شود.

## پورت‌های FTP

FTP یک سرویس TCP است و UDP ندارد. FTP معمولاً از دو ارتباط استفاده می‌کند:

* **control/command**: معمولاً روی پورت 21
* **data**: بسته به mode می‌تواند پورت 20 یا یک پورت تصادفی (random high port) باشد

## Active FTP در مقابل Passive FTP

به دلیل وجود firewall و NAT، تفاوت Active و Passive مهم است.

### Active mode

در Active mode، client به server روی پورت 21 وصل می‌شود، سپس server برای data connection از پورت 20 به سمت client برمی‌گردد. این می‌تواند پشت NAT/firewall مشکل ایجاد کند.

### Passive mode (PASV)

در Passive mode، client هر دو connection را شروع می‌کند. server یک پورت تصادفی (P > 1023) را اعلام می‌کند و client برای data به همان پورت وصل می‌شود. معمولاً در محیط‌های امروزی (پشت NAT) استفاده از PASV رایج‌تر است.

### جمع‌بندی سریع

* **Active**:
  * command: client >1023 -> server 21
  * data: client >1023 <- server 20
* **Passive**:
  * command: client >1023 -> server 21
  * data: client >1024 -> server >1023

## vsftpd (Very Secure FTP Daemon)

`vsftpd` یک FTP server سبک، پایدار و نسبتاً امن برای سیستم‌های UNIX-like است. با وجود نامش، اگر از SSL/TLS استفاده نکنید هنوز ترافیک رمزگذاری نمی‌شود؛ اما گزینه‌های امنیتی زیادی دارد.

### نصب و راه‌اندازی (نمونه در CentOS)

```bash
yum install vsftpd
systemctl enable --now vsftpd
```

فایل پیکربندی اصلی معمولاً این است:

* `/etc/vsftpd/vsftpd.conf`

### تنظیمات رایج امنیتی

چند گزینه مهم در `vsftpd.conf`:

```ini
# فعال/غیرفعال کردن anonymous
anonymous_enable=YES

# اجازه login کاربران local
local_enable=YES

# اجازه write (برای upload)
write_enable=YES

# محدود کردن کاربران local داخل home (chroot)
chroot_local_user=YES

# Passive mode و رنج پورت‌ها
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
```

### anonymous upload (با احتیاط)

اگر anonymous upload را فعال می‌کنید، باید بسیار محتاط باشید چون می‌تواند به محلی برای آپلود بدافزار/فایل‌های غیرمجاز تبدیل شود.

نکات مهم:

* anonymous upload را فقط روی یک دایرکتوری جداگانه فعال کنید.
* permissionها را طوری تنظیم کنید که کاربر anonymous نتواند در کل سیستم بنویسد.
* در صورت امکان روی آن مسیر گزینه‌های mount مثل `noexec`, `nosuid`, `nodev` را فعال کنید.
* logها را بررسی کنید و quota/محدودیت حجم در نظر بگیرید.

نمونه گزینه‌ها:

```ini
anon_upload_enable=YES
anon_mkdir_write_enable=YES
# مسیر روت anonymous (بسته به توزیع ممکن است متفاوت باشد)
anon_root=/var/ftp/pub
```

و سپس:

```bash
systemctl restart vsftpd
```

## Pure-FTPd و ProFTPd

برای آزمون LPIC-2 باید با وجود Pure-FTPd و ProFTPd هم آشنا باشید و بدانید که گزینه‌های امنیتی مشابهی (مثل محدودسازی کاربران، chroot، passive ports و SSL/TLS) دارند. در عمل، اصول ایمن‌سازی مستقل از محصول هستند:

* فعال کردن encryption (SSL/TLS) در صورت استفاده از FTP
* محدودسازی دسترسی کاربران و anonymous
* محدود کردن رنج پورت‌های passive و باز کردن همان‌ها روی firewall
* فعال کردن log و مانیتورینگ
