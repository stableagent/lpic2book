# 201.2. کامپایل کردن هسته

## **201.2 Compiling a kernel**

## **وزن:** 3

**توضیحات:** داوطلبان باید بتوانند به درستی هسته را پیکربندی کنند تا ویژگی‌های خاص هسته لینوکس را در صورت لزوم شامل یا غیرفعال کنند. این هدف شامل کامپایل و کامپایل مجدد هسته لینوکس در صورت نیاز، به‌روزرسانی و توجه به تغییرات در هسته جدید، ایجاد یک تصویر initrd و نصب هسته‌های جدید است.

**نواحی کلیدی دانش:**

* /usr/src/linux/
* Kernel Makefiles
* اهداف make هسته 2.6.x/3.x
* سفارشی‌سازی پیکربندی هسته فعلی.
* ساخت یک هسته جدید و ماژول‌های مناسب هسته.
* نصب یک هسته جدید و هر ماژولی.
* اطمینان از اینکه boot manager بتواند هسته جدید و فایل‌های مرتبط را پیدا کند.
* فایل‌های پیکربندی ماژول
* استفاده از DKMS برای کامپایل ماژول‌های هسته.
* آگاهی از dracut

**اصطلاحات و ابزارها:**

* mkinitrd
* mkinitramfs
* make
* اهداف make (all, config, xconfig, menuconfig, gconfig, oldconfig, mrproper, zImage, bzImage, modules, modules\_install, rpm-pkg, binrpm-pkg, deb-pkg)
* gzip
* bzip2
* ابزارهای ماژول
* /usr/src/linux/.config
* /lib/modules/kernel-version/
* depmod
* dkms

## هسته

چرا باید هسته لینوکس خود را کامپایل یا ارتقا دهیم؟ راستش را بگویم چون هسته لینوکس بسیار ماژولار و انعطاف‌پذیر است، اکثر مواقع نیازی به دستکاری هسته نیست. به خصوص وقتی صحبت از نسخه‌های تجاری لینوکس مثل redhat می‌شود، دستکاری هسته باعث مشکلات پشتیبانی می‌شود. در مقابل در سیستم لینوکس تعبیه شده، شما باید سعی کنید هسته را تا حد ممکن کوچک کنید. بنابراین کامپایل یا ارتقای هسته فقط اگر دلیل خوبی وجود داشته باشد.

ارتقای هسته و کامپایل آن بسیار مخصوص توزیع است. روشی که ما برای رسیدن به این هدف طی می‌کنیم ممکن است در Ubuntu، Open Suse یا cent OS متفاوت باشد. ارتقای هسته شامل کامپایل است، بنابراین اینجا ما با دانلود نسخه جدید هسته و کامپایل آن شروع می‌کنیم، اما به عنوان یک مدیر ممکن است شما نسخه فعلی لینوکس خود را دوباره کامپایل کنید تا برخی ماژول‌ها و قابلیت‌ها را اضافه / حذف کنید.

### ارتقای هسته

ابتدا به [https://kernel.org](https://kernel.org) بروید تا نسخه دلخواه هسته را بگیرید:

![](.gitbook/assets/KernelOrg.jpg)

ما آخرین نسخه پایدار هسته را انتخاب می‌کنیم، فراموش نکنید که ما باید در /usr/src/ باشیم و از این به بعد فقط در این دایرکتوری کار کنیم، در این نمایش از CentOS7 استفاده می‌کنیم:

```
[root@server1 ~]# cd /usr/src/
[root@server1 src]# ls -l
total 0
drwxr-xr-x. 2 root root 6 Nov  5 2016 debug
drwxr-xr-x. 2 root root 6 Nov  5 2016 kernels
[root@server1 src]# wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.3.tar.xz
--2017-12-04 08:34:00--  https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.3.tar.xz
Resolving cdn.kernel.org (cdn.kernel.org)... 151.101.1.176, 151.101.65.176, 151.101.129.176, ...
Connecting to cdn.kernel.org (cdn.kernel.org)|151.101.1.176|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 100778240 (96M) [application/x-xz]
Saving to: 'linux-4.14.3.tar.xz'

100%[=============================================>] 100,778,240 247KB/s   in 8m 15s

2017-12-04 08:42:20 (199 KB/s) - 'linux-4.14.3.tar.xz' saved [100778240/100778240]

[root@server1 src]# ls -l
total 98420
drwxr-xr-x. 2 root root         6 Nov  5 2016 debug
drwxr-xr-x. 2 root root         6 Nov  5 2016 kernels
-rw-r--r--. 1 root root 100778240 Nov 30 03:52 linux-4.14.3.tar.xz
```

اکنون بیایید فایل .xz را استخراج کنیم. همچنین باید یک لینک نمادی به نام linux ایجاد کنیم که به هسته دانلود شده اشاره دارد:

```
[root@server1 src]# tar -Jxvf linux-4.14.3.tar.xz
********
******
****
**
[root@server1 src]# ls -l
total 98424
drwxr-xr-x. 2 root root         6 Nov  5 2016 debug
drwxr-xr-x. 2 root root         6 Nov  5 2016 kernels
drwxrwxr-x. 24 root root      4096 Nov 30 03:41 linux-4.14.3
-rw-r--r--. 1 root root 100778240 Nov 30 03:52 linux-4.14.3.tar.xz
[root@server1 src]# ln -s linux-4.14.3 linux
[root@server1 src]# ls -l
total 98424
drwxr-xr-x. 2 root root         6 Nov  5 2016 debug
drwxr-xr-x. 2 root root         6 Nov  5 2016 kernels
lrwxrwxrwx. 1 root root        12 Dec  4 08:48 linux -> linux-4.14.3
drwxrwxr-x. 24 root root      4096 Nov 30 03:41 linux-4.14.3
-rw-r--r--. 1 root root 100778240 Nov 30 03:52 linux-4.14.3.tar.xz
```

قبل از ادامه، پوشه document در داخل هسته دانلود شده وجود دارد، فایل‌های متن کمک فوق‌العاده‌ای در آنجا است:

