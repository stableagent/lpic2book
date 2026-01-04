# 202.1. سفارشی‌سازی راه‌اندازی سیستم SysV-init

## **202.1 Customizing SysV-init system startup**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند رفتار سرویس‌های سیستم را در اهداف / سطوح اجرای مختلف پرس‌وجو و تغییر دهند. درک کامل از systemd، SysV Init و فرآیند بوت لینوکس مورد نیاز است. این هدف شامل تعامل با اهداف systemd و سطوح اجرای SysV init است.

**نواحی کلیدی دانش:**

* Systemd
* SysV init
* مشخصات پایه استاندارد لینوکس (LSB)

**اصطلاحات و ابزارها:**

* /usr/lib/systemd/
* /etc/systemd/
* /run/systemd/
* systemctl
* systemd-delta
* /etc/inittab
* /etc/init.d/
* /etc/rc.d/
* chkconfig
* update-rc.d
* init و telinit

### نمای کلی

در طول درس‌های قبلی ما در مورد initrd/initramfs صحبت کردیم. وقتی هسته کاملاً بارگذاری شد، به دنبال فرآیند init می‌گردد تا آن را شروع کند. فرآیند init می‌تواند init، upstart یا systemd باشد. به طور سنتی System v init برای شروع سرویس‌های دیگر استفاده می‌شود اما برخی کاستی‌ها را دارد. بنابراین راه‌حل‌های دیگر مانند upstart و systemd اختراع شد.

/sbin/init می‌تواند به upstart یا systemd لینک شده باشد. برای بررسی اینکه از کدام سیستم استفاده می‌کنید، وجود هر دایرکتوری را بررسی کنید:

| دایرکتوری          | توضیحات \[اگر وجود داشته باشد]               |
| ------------------ | ------------------------------------- |
| /etc/init.d        | نشان می‌دهد که SysV در سیستم لینوکس شما دارید |
| /usr/share/upstart | شما در یک سیستم مبتنی بر Upstart هستید     |
| /usr/lib/systemd   | شما از سیستم مبتنی بر Systemd استفاده می‌کنید    |

همچنین stat /proc/1/exe را امتحان کنید

| لینک                                          | توضیحات                |
| --------------------------------------------- | -------------------------- |
| File: '/proc/1/exe' -> '/sbin/init'           | در سیستم SysV و upstart |
| File: '/proc/1/exe' -> '/lib/systemd/systemd' | در جعبه Systemd             |

### Sys v

سیستم "5" یا Sys "v" یک روش باستانی برای مدیریت سرویس‌های سیستم از دنیای یونیکس باز به سال‌های 1980 است. SysV از بارگذاری سریالی سرویس‌ها استفاده می‌کند، به زبان دیگر هر سرویس باید به ترتیب (پس از هم) بارگذاری شود. SysV از مفهوم runlevels برای تعریف اینکه سرور باید در کدام حالت بوت شود استفاده می‌کند. در هر runlevel مقدار مشخصی از اسکریپت‌های shell پردازش می‌شود تا به وضعیت مورد نظر خود برسیم.

runlevels از 0 تا 6 شروع می‌شوند و در سیستم‌های مبتنی بر Redhat و Debian متفاوت هستند.

| runlevel | Redhat                              | Debian                                 |
| -------- | ----------------------------------- | -------------------------------------- |
| 0        | توقف سیستم (به عنوان پیش‌فرض تنظیم نکنید) | توقف سیستم (به عنوان پیش‌فرض استفاده نکنید)    |
| 1        | حالت کاربر تک                    | حالت کاربر تک                       |
| 2        | چند کاربر بدون NFS              | حالت کامل چند کاربر با GUI(پیش‌فرض) |
| 3        | حالت کامل چند کاربر                | --همانطور که 2 است-- --استفاده نشده--               |
| 4        | --استفاده نشده--                         | --همانطور که 2 است-- --استفاده نشده--               |
| 5        | X11/حالت کامل چند کاربر(پیش‌فرض)   | --همانطور که 2 است-- --استفاده نشده--               |
| 6        | راه‌اندازی مجدد (به عنوان initdefault تنظیم نکنید)  | راه‌اندازی مجدد (به عنوان پیش‌فرض تنظیم نکنید)         |

/etc/inittab فایل پیکربندی SysV است که در آن می‌توان runlevel پیش‌فرض را تنظیم کرد، ما از CentOS 5 برای نمایش استفاده می‌کنیم:

```
#
# inittab       This file describes how the INIT process should set up
#               system in a certain run-level.
#
# Author:       Miquel van Smoorenburg, <miquels@drinkel.nl.mugnet.org>
#               Modified for RHS Linux by Marc Ewing and Donnie Barnes
#

# Default runlevel. The runlevels used by RHS are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
#
id:5:initdefault:

# System initialization.
si::sysinit:/etc/rc.d/rc.sysinit

l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5
l6:6:wait:/etc/rc.d/rc 6

# Trap CTRL-ALT-DELETE
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# When our UPS tells us power has failed, assume we have a few minutes
