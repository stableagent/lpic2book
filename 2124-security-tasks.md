# 212.4. وظایف امنیتی

## **212.4 وظایف امنیتی**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند از منابع مختلف هشدارهای امنیتی دریافت کنند، سیستم‌های IDS را نصب/پیکربندی/اجرا کنند و patchها و bugfixهای امنیتی را اعمال کنند.

**نواحی کلیدی دانش:**

* ابزارها و utilityها برای scan و تست کردن portهای یک سرور
* آشنایی با منابع گزارش هشدارهای امنیتی مثل Bugtraq، CERT و منابع مشابه
* ابزارها و utilityها برای پیاده‌سازی Intrusion Detection System (IDS)
* آگاهی از OpenVAS و Snort

**اصطلاحات و ابزارها:**

* telnet
* nmap
* fail2ban
* nc
* iptables

امنیت لینوکس موضوع گسترده‌ای است و در LPIC-3 عمیق‌تر پوشش داده می‌شود، اما برای LPIC-2 باید با چند مفهوم و ابزار کلیدی آشنا باشیم.

## چه کارهایی برای افزایش امنیت سیستم انجام می‌دهیم؟

### 1) Updates و Patch management

* سیستم‌ها را به‌روز نگه دارید.
* در محیط production ممکن است آپدیت‌های روزانه عملی نباشد، اما آپدیت کردن سرویس‌های مهم (خصوصاً سرویس‌های دسترسی remote) باید منظم انجام شود.
* اعلان‌های امنیتی توزیع (Debian Security, Ubuntu Security Notices, Red Hat Errata و ...) را دنبال کنید.

### 2) Monitoring و Audit

* بررسی logها، مانیتورینگ سرویس‌ها و alerting نقش مهمی در کشف به‌موقع حملات دارد.
* auditهای دوره‌ای دستی مفید است، اما کافی نیست؛ مانیتورینگ خودکار ضروری است.

### 3) ابزارهای امنیتی

برخی ابزارهای رایج:

* nmap
* nc
* telnet
* iptables
* fail2ban
* snort
* OpenVAS

## منابع دریافت هشدارهای امنیتی

### CERT

CERT (Computer Emergency Response Team) و خصوصاً CERT-CC (Coordination Center) از منابع مهم برای اطلاع از آسیب‌پذیری‌ها و incidentها هستند.

### US-CERT

یکی از منابع هشدار و گزارش incidentهای امنیتی.

### BugTraq

Bugtraq یک mailing list شناخته‌شده برای اعلام و بحث درباره آسیب‌پذیری‌هاست.

## اسکن و تست portها

### nc (netcat)

`nc` یک ابزار ساده و بسیار کاربردی برای کار با TCP/UDP است و می‌تواند برای تست باز/بسته بودن portها استفاده شود.

نمونه اسکن یک رنج port روی localhost:

```bash
nc -zv localhost 21-29
```

### nmap

`nmap` ابزار استاندارد برای port scanning و network discovery است.

نمونه scan سریع:

```bash
nmap -sS -Pn 192.168.1.10
```

چند گزینه رایج:

* `-sS`: SYN scan
* `-sV`: تشخیص سرویس/نسخه
* `-O`: تشخیص سیستم عامل (ممکن است نیاز به دسترسی بیشتر داشته باشد)
* `-Pn`: عدم استفاده از ping قبل از scan

## fail2ban (دفاع در برابر brute force)

`fail2ban` با بررسی logها، الگوهای شکست login (مثلاً در sshd) را تشخیص می‌دهد و با ruleهای firewall (iptables/nftables) IPهای مهاجم را موقتاً ban می‌کند.

نمونه فعال‌سازی jail برای ssh (بسته به توزیع):

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 5
bantime = 3600
```

و سپس:

```bash
systemctl enable --now fail2ban
fail2ban-client status
fail2ban-client status sshd
```

## IDS / IPS

### Snort

Snort می‌تواند به عنوان IDS/IPS برای تشخیص الگوهای حمله روی شبکه استفاده شود. برای LPIC-2 کافی است مفهوم و کاربرد آن را بدانید.

### OpenVAS

OpenVAS (امروزه بیشتر با نام Greenbone شناخته می‌شود) یک vulnerability scanner است که برای ارزیابی امنیتی و اسکن آسیب‌پذیری‌های سیستم‌ها و سرویس‌ها استفاده می‌شود.

## اعمال patchهای امنیتی

* در Debian/Ubuntu:

```bash
apt update
apt upgrade
```

* در RHEL/CentOS:

```bash
yum update
# یا
DNF update
```

نکته: در محیط production معمولاً یک فرآیند مشخص برای تست و سپس rollout آپدیت‌ها وجود دارد.
