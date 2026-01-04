# 208.3. پیاده‌سازی یک سرور Proxy

## **208.3 پیاده‌سازی یک سرور Proxy**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند یک proxy server را نصب و پیکربندی کنند؛ شامل access policy، authentication و کنترل مصرف منابع.

**نواحی کلیدی دانش:**

* فایل‌های پیکربندی، اصطلاحات و ابزارهای Squid 3.x
* روش‌های محدودسازی دسترسی (access restriction)
* روش‌های احراز هویت کاربران
* ساختار ACL و نحوه استفاده از آن در فایل‌های پیکربندی Squid

**اصطلاحات و ابزارها:**

* squid.conf
* acl
* http_access

## Proxy چیست؟

Proxy server سیستمی است که بین client و اینترنت قرار می‌گیرد و درخواست‌ها را به صورت غیرمستقیم به سمت serverهای مقصد ارسال می‌کند. کاربردهای رایج:

* اشتراک‌گذاری اینترنت در LAN
* caching برای افزایش سرعت دسترسی به وب
* اعمال سیاست‌های دسترسی (policy)
* احراز هویت کاربران و ثبت گزارش‌ها (logging)

## Squid چیست؟

`Squid` یک web proxy و cache server متن‌باز است که معمولاً برای HTTP استفاده می‌شود و قابلیت‌هایی مثل caching، ACL، authentication و محدودسازی منابع را ارائه می‌دهد.

![](.gitbook/assets/squid_logo.png)

## نصب (نمونه)

* در CentOS/RHEL:

```bash
yum install squid
systemctl enable --now squid
```

* در Debian/Ubuntu:

```bash
apt update
apt install squid
systemctl enable --now squid
```

## فایل پیکربندی

مسیر فایل معمولاً یکی از این‌هاست:

* `/etc/squid/squid.conf`
* `/etc/squid3/squid.conf`

## مفاهیم مهم: ACL و http_access

در Squid معمولاً ابتدا ACLها تعریف می‌شوند و سپس با `http_access` اجازه/محدودیت اعمال می‌شود.

نمونه ACL بر اساس شبکه داخلی:

```conf
acl localnet src 192.168.1.0/24
```

نمونه ACL بر اساس دامنه:

```conf
acl blocked dstdomain .example.com
```

اعمال policy:

```conf
http_access deny blocked
http_access allow localnet
http_access deny all
```

نکته: ترتیب `http_access` مهم است (اولین match اعمال می‌شود).

## احراز هویت کاربران (آگاهی)

Squid می‌تواند با روش‌های مختلف کاربران را authenticate کند (basic auth، LDAP، و ...). برای LPIC-2 کافی است بدانید این قابلیت وجود دارد و با helperها پیاده‌سازی می‌شود.

## کنترل منابع و logging

چند گزینه رایج:

* تعیین اندازه cache
* محل logها (access.log/cache.log)
* محدود کردن clientها با ACL

بعد از تغییر تنظیمات، syntax را بررسی و سرویس را reload/restart کنید:

```bash
squid -k parse
systemctl restart squid
```
