# 206.1. ساخت و نصب برنامه‌ها از سورس

## **206.1 Make and install programs from source**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند یک برنامه قابل اجرا را از سورس بسازند و نصب کنند. این هدف شامل توانایی باز کردن بایگ سورس‌ها است.

**نواحی کلیدی دانش:**

* باز کردن کد سورس با استفاده از ابزارهای فشرده‌سازی و بایگ معمول
* درک مبانی فراخوانی make برای کامپایل برنامه‌ها
* اعمال پارامترها به اسکریپت configure
* دانستن اینکه سورس‌ها به طور پیش‌فرض کجا ذخیره می‌شوند

**اصطلاحات و ابزارها:**

* /usr/src/
* gunzip
* gzip
* bzip2
* xz
* tar
* configure
* make
* uname
* install
* patch

## ساخت و نصب برنامه‌ها از سورس

همانطور که می‌دانید هر توزیع مدرن لینوکس مخزن خود را دارد، که در آن می‌توانیم به راحتی برای یک برنامه جستجو کنیم، اطلاعات بگیریم و از آن نصب کنیم. اگرچه نصب یک برنامه از مخزن آسان است و نوعی امنیت ایجاد می‌کند، اما سورس آن برنامه را به یک توزیع خاص محدود می‌کند. بنابراین بسیاری از توسعه‌دهندگان ترجیح می‌دهند که سورس کد برنامه خود را منتشر کنند و اجازه دهند افراد دیگر آن را بر اساس توزیع و نیازهای خود پیکربندی و کامپایل کنند.

3 مرحله اساسی برای نصب یک برنامه از کد سورس عبارتند از:

![](.gitbook/assets/make.jpg)

قبل از شروع ما نیاز به نصب برخی ابزارهای توسعه داریم. در redhat از `yum groupinstall "Development Tools"` استفاده کنید و در debian based و ubuntu `apt-get install build-essentia` را اجرا کنید:

```
root@server1:~# apt install build-essential
Reading package lists... Done
Building dependency tree
Reading state information... Done
build-essential is already the newest version (12.1ubuntu2).
0 upgraded, 0 newly installed, 0 to remove and 172 not upgraded.
```

برای نمایش بیایید با cpuminer بیت‌کوین استخراج کنیم و سرگرم شویم. اول سورس را دانلود کنید:

```
root@server1:~# wget https://sourceforge.net/projects/cpuminer/files/pooler-cpuminer-2.5.0.tar.gz
--2018-02-04 02:37:14--  https://sourceforge.net/projects/cpuminer/files/pooler-cpuminer-2.5.0.tar.gz
Resolving sourceforge.net (sourceforge.net)... 216.34.181.60
Connecting to sourceforge.net (sourceforge.net)|216.34.181.60|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://sourceforge.net/projects/cpuminer/files/pooler-cpuminer-2.5.0.tar.gz/download [following]
--2018-02-04 02:37:19--  https://sourceforge.net/projects/cpuminer/files/pooler-cpuminer-2.5.0.tar.gz/download
Connecting to sourceforge.net (sourceforge.net)|216.34.181.60|:443... connected.
HTTP request sent, awaiting response... 302 Found
