# 208.2. پیکربندی Apache برای HTTPS

## **208.2 پیکربندی Apache برای HTTPS**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند Apache را برای استفاده از TLS/SSL پیکربندی کنند. این هدف شامل مدیریت certificateها، فعال‌سازی HTTPS روی VirtualHostها و اعمال تنظیمات امنیتی مرتبط است.

**نواحی کلیدی دانش:**

* مفاهیم TLS/SSL و certificate
* پیکربندی Apache برای HTTPS (mod_ssl / ssl)
* استفاده از VirtualHost برای HTTP و HTTPS
* آگاهی از Best Practiceهای ساده امنیتی (redirect، محدود کردن پروتکل‌ها و cipherها)

**اصطلاحات و ابزارها:**

* Apache 2.4
* mod_ssl / ssl
* openssl
* فایل‌های پیکربندی (sites-available / conf.d)

## HTTPS در Apache

برای فعال کردن HTTPS، Apache به یک certificate (و private key) نیاز دارد. این certificate می‌تواند self-signed باشد (برای تست) یا از یک CA مثل Let’s Encrypt صادر شود.

### ایجاد یک certificate self-signed (برای تست)

```bash
openssl req -x509 -nodes -newkey rsa:2048 \
  -keyout /etc/ssl/private/example.key \
  -out /etc/ssl/certs/example.crt \
  -days 365
```

### فعال کردن ماژول SSL

* در Debian/Ubuntu:

```bash
a2enmod ssl
systemctl reload apache2
```

* در RHEL/CentOS معمولاً mod_ssl با package جدا نصب/فعال می‌شود:

```bash
yum install mod_ssl
systemctl restart httpd
```

### نمونه VirtualHost برای HTTPS

```apache
<VirtualHost *:443>
    ServerName example.com

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.crt
    SSLCertificateKeyFile /etc/ssl/private/example.key

    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/example-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/example-ssl-access.log combined
</VirtualHost>
```

### redirect کردن HTTP به HTTPS

بهتر است برای وب‌سایت‌هایی که باید امن باشند، همه درخواست‌های HTTP را به HTTPS redirect کنیم:

```apache
<VirtualHost *:80>
    ServerName example.com

    Redirect permanent / https://example.com/
</VirtualHost>
```

یا با mod_rewrite.

### بررسی و reload کردن پیکربندی

```bash
apachectl configtest
systemctl reload apache2
# یا
systemctl reload httpd
```

## نکات امنیتی ساده

* غیرفعال کردن TLSهای قدیمی (در صورت نیاز و بر اساس سیاست سازمان)
* محدود کردن cipherها به cipher suiteهای امن
* فعال کردن HSTS در صورت اطمینان از HTTPS-only بودن سایت

نمونه (بسته به distro ممکن است مسیر فایل‌ها متفاوت باشد):

```apache
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite HIGH:!aNULL:!MD5
SSLHonorCipherOrder on

Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains" 
```
