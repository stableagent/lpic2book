# 208.4. پیاده‌سازی Nginx به عنوان web server و reverse proxy

## **208.4 پیاده‌سازی Nginx به عنوان web server و reverse proxy**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند Nginx را نصب و پیکربندی کنند تا به عنوان web server و reverse proxy استفاده شود.

**نواحی کلیدی دانش:**

* ساختار فایل‌های پیکربندی Nginx
* server blockها (virtual host)
* reverse proxy با proxy_pass
* تنظیم headerهای رایج و نکات پایه امنیتی

**اصطلاحات و ابزارها:**

* nginx
* nginx.conf
* sites-available / conf.d (بسته به توزیع)
* proxy_pass

## Nginx چیست؟

Nginx یک web server و reverse proxy سبک و پرکاربرد است که برای serving محتوای static و همچنین قرار گرفتن جلوی application serverها (مثل Node.js، PHP-FPM، Tomcat و ...) استفاده می‌شود.

## نصب (نمونه)

* Debian/Ubuntu:

```bash
apt update
apt install nginx
systemctl enable --now nginx
```

* RHEL/CentOS (ممکن است نیاز به repo مناسب داشته باشد):

```bash
yum install nginx
systemctl enable --now nginx
```

## ساختار پیکربندی

فایل اصلی معمولاً:

* `/etc/nginx/nginx.conf`

و فایل‌های شامل‌شونده:

* `/etc/nginx/conf.d/*.conf`
* یا `/etc/nginx/sites-available` و symlink به `sites-enabled` (در برخی توزیع‌ها)

## نمونه server block (وب‌سایت ساده)

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    access_log /var/log/nginx/example.access.log;
    error_log  /var/log/nginx/example.error.log;
}
```

## reverse proxy

اگر backend شما روی localhost:3000 اجرا می‌شود:

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## بررسی و reload

قبل از reload حتماً syntax را چک کنید:

```bash
nginx -t
systemctl reload nginx
```
