# 207.3. ایمن‌سازی سرور DNS

## **207.3 ایمن‌سازی سرور DNS**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند یک DNS server را ایمن کنند. این هدف شامل محدود کردن recursion، کنترل دسترسی به queryها، محدود کردن zone transfer و آگاهی از DNSSEC است.

**نواحی کلیدی دانش:**

* کنترل recursion و cache-only resolver
* محدود کردن access به query و zone transfer
* آگاهی از DNSSEC
* logging و مانیتورینگ

**اصطلاحات و ابزارها:**

* BIND
* named.conf
* rndc

## اصول پایه ایمن‌سازی DNS

DNS هم مثل سایر سرویس‌ها می‌تواند هدف حمله باشد (مثلاً amplification، cache poisoning، سوءاستفاده از recursion باز، zone transfer غیرمجاز و ...). چند اصل مهم:

1. **recursion را فقط برای کلاینت‌های مجاز فعال کنید**
2. **zone transfer را محدود کنید**
3. **نسخه و patchها را به‌روز نگه دارید**
4. **logging و monitoring داشته باشید**

## محدود کردن recursion

اگر سرور شما authoritative است و قرار نیست resolver عمومی باشد، recursion را غیرفعال کنید یا فقط برای شبکه داخلی اجازه دهید.

نمونه (BIND):

```conf
options {
    recursion no;
};
```

یا اجازه دادن فقط به شبکه داخلی:

```conf
acl trusted {
    127.0.0.1;
    192.168.1.0/24;
};

options {
    recursion yes;
    allow-recursion { trusted; };
};
```

## محدود کردن queryها

```conf
options {
    allow-query { trusted; };
};
```

## محدود کردن zone transfer

zone transfer (AXFR) باید فقط به secondary DNS serverها اجازه داده شود:

```conf
zone "example.com" IN {
    type master;
    file "/var/named/db.example.com";
    allow-transfer { 192.168.1.11; };
};
```

در صورت نیاز می‌توان از TSIG key برای امن کردن transfer استفاده کرد.

## DNSSEC (آگاهی)

DNSSEC مکانیزمی برای امضای پاسخ‌های DNS است تا از دستکاری/جعلی‌سازی رکوردها جلوگیری شود. برای LPIC-2 کافی است مفهوم کلی و هدف DNSSEC را بدانید.

## rndc و کنترل از راه دور

BIND معمولاً برای مدیریت سرویس از `rndc` استفاده می‌کند. اطمینان حاصل کنید که تنظیمات rndc/keyها محافظت شده‌اند و دسترسی غیرمجاز ندارند.

## logging و مانیتورینگ

* logهای named را بررسی کنید.
* حجم queryها و رفتار غیرعادی (مثل افزایش ناگهانی درخواست‌ها) را مانیتور کنید.

## نکات تکمیلی

* اجرای سرویس با کمترین دسترسی (least privilege)
* در صورت پشتیبانی، استفاده از chroot یا container
* محدود کردن دسترسی firewall به پورت 53 (TCP/UDP) فقط برای شبکه‌های مورد نیاز
