# 207.2. ایجاد و نگهداری zoneهای DNS

## **207.2 ایجاد و نگهداری zoneهای DNS**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند فایل‌های zone را ایجاد و نگهداری کنند و تغییرات را به درستی reload کنند. این هدف شامل کار با رکوردهای رایج DNS و ابزارهای تست است.

**نواحی کلیدی دانش:**

* ساختار فایل‌های zone در BIND
* رکوردهای رایج: SOA, NS, A, AAAA, CNAME, MX, PTR
* reverse zone و رکوردهای PTR
* تست و validate کردن zone

**اصطلاحات و ابزارها:**

* named.conf
* zone fileها (معمولاً زیر /var/named/ یا /etc/bind/)
* named-checkconf
* named-checkzone
* dig
* host

## zone چیست؟

در DNS، یک **zone** بخشی از namespace است که توسط یک یا چند DNS server مدیریت می‌شود. در BIND معمولاً zoneها در فایل پیکربندی `named.conf` تعریف می‌شوند و به فایل‌های zone اشاره می‌کنند.

## تعریف zone در BIND (نمونه)

```conf
zone "example.com" IN {
    type master;
    file "/var/named/db.example.com";
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/db.192.168.1";
};
```

## ساختار یک zone file (forward)

نمونه ساده:

```dns
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2026010401 ; serial
        3600       ; refresh
        900        ; retry
        604800     ; expire
        86400 )    ; minimum

    IN  NS  ns1.example.com.
    IN  NS  ns2.example.com.

ns1 IN  A   192.168.1.10
ns2 IN  A   192.168.1.11

www IN  A   192.168.1.20
mail IN A   192.168.1.30

@   IN  MX 10 mail.example.com.
```

نکته‌ها:

* `SOA` شامل مشخصات zone و `serial` است. هر بار تغییر zone باید serial افزایش یابد.
* در نام‌های داخل zone، نقطه پایانی (`.`) معنی FQDN دارد.

## reverse zone (PTR)

نمونه:

```dns
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2026010401
        3600
        900
        604800
        86400 )

    IN  NS  ns1.example.com.

10  IN  PTR ns1.example.com.
20  IN  PTR www.example.com.
30  IN  PTR mail.example.com.
```

## بررسی و تست

چک کردن syntax پیکربندی:

```bash
named-checkconf
```

چک کردن یک zone:

```bash
named-checkzone example.com /var/named/db.example.com
```

تست query:

```bash
dig @127.0.0.1 www.example.com
host -t mx example.com
```

## reload کردن تغییرات

بعد از تغییر zone file می‌توانید سرویس را reload کنید (بسته به توزیع):

```bash
rndc reload
# یا
systemctl reload named
```
