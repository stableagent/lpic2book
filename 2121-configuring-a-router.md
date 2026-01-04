# 212.1. پیکربندی یک روتر

## **212.1 پیکربندی یک روتر**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند یک سیستم را طوری پیکربندی کنند که packetهای IP را forward کند و NAT (یا IP masquerading) انجام دهد و اهمیت آن را در حفاظت از شبکه توضیح دهند. این هدف شامل پیکربندی port redirection، مدیریت ruleهای فیلترینگ و جلوگیری از برخی حملات رایج است.

**نواحی کلیدی دانش:**

* فایل‌ها، ابزارها و utilityهای پیکربندی iptables و ip6tables
* ابزارها و دستورهای مدیریت routing table
* رنج‌های private در IPv4 و همچنین Unique Local Address و Link Local Address در IPv6
* IP forwarding و Port redirection
* نوشتن ruleهایی که packetهای IP را بر اساس protocol، port و آدرس مبدا/مقصد accept یا block می‌کنند
* ذخیره و reload کردن پیکربندی فیلترینگ

**اصطلاحات و ابزارها:**

* /proc/sys/net/ipv4/
* /proc/sys/net/ipv6/
* /etc/services
* iptables
* ip6tables

در لینوکس (مثل هر سیستم عامل مدرن دیگر) قابلیت firewall وجود دارد. در سطح kernel این قابلیت با `netfilter` پیاده‌سازی می‌شود و ترافیکی که از kernel عبور می‌کند (ورودی/خروجی/forward) می‌تواند توسط netfilter بررسی و بر اساس ruleها پذیرفته یا رد شود.

رابط اصلی برای مدیریت netfilter در سطح کاربر `iptables` است (برای IPv4) و `ip6tables` برای IPv6. ابزارهایی مثل `ufw` و `firewalld` هم در پشت صحنه روی iptables کار می‌کنند و مدیریت را ساده‌تر می‌کنند، اما در LPIC-2 تمرکز روی خود iptables است.

## مفاهیم پایه iptables

iptables بر اساس «table» و «chain» کار می‌کند:

### tableها

* **filter**: فیلتر کردن packetها (ACCEPT/DROP/REJECT و ...)
* **nat**: ترجمه آدرس‌ها (SNAT/DNAT/masquerade)
* **mangle**: پردازش‌های خاص روی packetها

### chainها

chainها نقاطی هستند که packet از آن‌ها عبور می‌کند و ruleها در آن‌ها بررسی می‌شوند. chainهای رایج:

* **PREROUTING** (معمولاً در جدول nat و mangle): قبل از routing تصمیم‌گیری می‌شود؛ اغلب برای DNAT/port forward
* **INPUT**: packetهایی که مقصدشان همین ماشین است
* **FORWARD**: packetهایی که از این ماشین عبور می‌کنند (router)
* **OUTPUT**: packetهایی که خود ماشین تولید می‌کند
* **POSTROUTING** (معمولاً در جدول nat و mangle): بعد از تصمیم routing؛ اغلب برای SNAT/masquerade

### targetها (اقدام نهایی rule)

target با گزینه `-j` تعیین می‌شود. رایج‌ترین targetها:

* **ACCEPT**: اجازه عبور
* **DROP**: حذف بی‌سروصدا (بدون پاسخ)
* **REJECT**: رد کردن با ارسال پاسخ (معمولاً ICMP)
* **LOG**: فقط log کردن
* **MASQUERADE**: نوعی SNAT پویا (در جدول nat)

### policy

هر chain یک policy پیش‌فرض دارد (اگر هیچ ruleای match نشود). معمولاً policy پیش‌فرض را روی DROP می‌گذارند و سپس اجازه‌های لازم را add می‌کنند:

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

## دیدن و مدیریت ruleها

نمایش ruleها:

```bash
iptables -L -n -v
iptables -t nat -L -n -v
```

پاک کردن ruleها (flush):

```bash
iptables -F
iptables -t nat -F
```

حذف یک rule مشخص (با شماره):

```bash
iptables -L --line-numbers
iptables -D INPUT 3
```

## فعال کردن IP forwarding

برای اینکه لینوکس نقش router داشته باشد باید IP forwarding فعال شود.

بررسی مقدار:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

فعال کردن موقت تا reboot:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# یا
sysctl -w net.ipv4.ip_forward=1
```

فعال کردن دائمی (مثال):

```bash
# /etc/sysctl.conf یا /etc/sysctl.d/*.conf
net.ipv4.ip_forward = 1
```

و اعمال:

```bash
sysctl -p
```

## NAT / Masquerade (برای خروج اینترنت)

سناریوی رایج: شبکه داخلی (LAN) با رنج private پشت یک gateway قرار دارد و می‌خواهیم ماشین‌های داخلی از طریق IP عمومی سرور به اینترنت دسترسی داشته باشند.

مثال masquerade:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

نکته: در کنار NAT معمولاً باید ruleهای FORWARD هم تنظیم شوند (مثلاً اجازه دادن به trafficهای established/related):

```bash
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```

## Port forwarding (DNAT)

اگر بخواهیم درخواست‌های ورودی روی یک port را به یک ماشین داخلی منتقل کنیم (مثلاً web server داخلی)، از DNAT در PREROUTING استفاده می‌کنیم:

```bash
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
```

و باید در FORWARD هم اجازه بدهیم:

```bash
iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 -j ACCEPT
```

## ذخیره و بازیابی ruleها

روش ذخیره/restore بسته به توزیع متفاوت است، اما ابزارهای استاندارد این‌ها هستند:

```bash
iptables-save > /root/iptables.rules
iptables-restore < /root/iptables.rules
```

برای IPv6 هم مشابه است:

```bash
ip6tables-save > /root/ip6tables.rules
ip6tables-restore < /root/ip6tables.rules
```

در نهایت فراموش نکنید که ترتیب ruleها مهم است ("exit on match") و در یک chain وقتی packet با یک rule match شود، معمولاً بقیه ruleها بررسی نمی‌شوند.
