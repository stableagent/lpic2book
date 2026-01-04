# 201.3. مدیریت و عیب‌یابی هسته در زمان اجرا

## **201.3 مدیریت و عیب‌یابی هسته در زمان اجرا**

**وزن:** 4

**توضیحات:** داوطلبان باید بتوانند یک kernel نسخه 2.6.x/3.x/4.x و ماژول‌های قابل load آن را مدیریت و query کنند. همچنین باید بتوانند مشکلات رایج boot و runtime را شناسایی و برطرف کنند. درک device detection و مدیریت با udev و عیب‌یابی ruleهای udev هم جزو این هدف است.

**نواحی کلیدی دانش:**

* استفاده از ابزارهای خط فرمان برای گرفتن اطلاعات درباره kernel در حال اجرا و kernel moduleها
* load و unload کردن ماژول‌های kernel به صورت دستی
* تشخیص اینکه چه زمانی یک ماژول می‌تواند unload شود
* بررسی پارامترهای قابل قبول یک ماژول
* پیکربندی سیستم برای load کردن ماژول‌ها با نام‌هایی متفاوت از نام فایل
* فایل‌سیستم /proc
* محتوای /, /boot/ و /lib/modules/
* ابزارهای بررسی سخت‌افزار موجود
* udev rules

**اصطلاحات و ابزارها:**

* /lib/modules/kernel-version/modules.dep
* فایل‌های پیکربندی ماژول در /etc/
* /proc/sys/kernel/
* depmod
* rmmod
* modinfo
* dmesg
* lspci
* lsmod
* modprobe
* insmod
* uname
* lsusb
* /etc/sysctl.conf, /etc/sysctl.d/
* sysctl
* udevadm monitor
* /etc/udev/

## اطلاعات درباره kernel در حال اجرا

نمایش نسخه kernel:

```bash
uname -a
uname -r
```

نمایش پیام‌های kernel (برای عیب‌یابی):

```bash
dmesg
```

## Kernel moduleها

Kernel از نسخه‌های قدیمی ماژولار است. یعنی بخش‌هایی از driverها و قابلیت‌ها می‌توانند به صورت module جداگانه load/unload شوند.

### مشاهده ماژول‌های load شده

```bash
lsmod
```

### اطلاعات درباره یک ماژول

```bash
modinfo e1000
```

### Load/Unload

* load کردن مستقیم:

```bash
insmod /path/to/module.ko
```

* روش پیشنهادی (با resolve کردن dependencyها):

```bash
modprobe e1000
```

* unload:

```bash
rmmod e1000
# یا
modprobe -r e1000
```

### dependencyها

بعد از تغییر یا اضافه شدن moduleها، dependencyها با `depmod` ساخته می‌شوند:

```bash
depmod -a
```

## /proc و sysctl

بخشی از تنظیمات runtime kernel از طریق `/proc` و `sysctl` قابل مشاهده/تغییر است.

مثال:

```bash
sysctl -a | head
sysctl net.ipv4.ip_forward
sysctl -w net.ipv4.ip_forward=1
```

برای دائمی کردن تنظیمات از `/etc/sysctl.conf` یا `/etc/sysctl.d/*.conf` استفاده می‌شود.

## udev

`udev` مدیریت deviceها در فضای کاربر را انجام می‌دهد و هنگام اضافه/حذف شدن سخت‌افزار، nodeهای device زیر `/dev` ساخته/حذف می‌شوند.

### مانیتور کردن eventها

```bash
udevadm monitor
```

### ruleها

ruleها معمولاً زیر `/etc/udev/rules.d/` یا `/lib/udev/rules.d/` قرار دارند. در عیب‌یابی ruleها معمولاً از `udevadm` و logها استفاده می‌شود.
