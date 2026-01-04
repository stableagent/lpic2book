# 212.3 پوسته امن (SSH)

## **212.3 پوسته امن (SSH)**

**وزن:** 4

**توضیحات:** داوطلبان باید بتوانند SSH client و SSH server را پیکربندی و استفاده کنند. این هدف شامل پیکربندی گزینه‌های کلیدی SSH، استفاده از key-based authentication، انتقال فایل امن و tunneling/port forwarding است.

**نواحی کلیدی دانش:**

* فایل‌های پیکربندی و گزینه‌های مهم ssh و sshd
* استفاده از key pair (public/private) و مدیریت known_hosts
* روش‌های انتقال فایل امن: scp و sftp
* agent forwarding و ssh-agent
* Local/Remote/Dynamic port forwarding

**اصطلاحات و ابزارها:**

* ssh
* sshd
* ssh-keygen
* ssh-agent
* ssh-add
* scp
* sftp
* ~/.ssh/ (authorized_keys, known_hosts, config)
* /etc/ssh/sshd_config

## SSH چیست؟

SSH (Secure Shell) یک پروتکل برای دسترسی remote امن به سیستم‌ها است که ارتباط را رمزگذاری می‌کند و جایگزین telnet و rsh شده است.

## فایل‌های مهم

* تنظیمات سرور: `/etc/ssh/sshd_config`
* کلیدهای کاربر: `~/.ssh/id_rsa` و `~/.ssh/id_rsa.pub` (یا ed25519)
* کلیدهای مجاز سرور برای کاربر: `~/.ssh/authorized_keys`
* کلیدهای شناخته‌شده: `~/.ssh/known_hosts`

## تولید و استفاده از key

ساخت key pair:

```bash
ssh-keygen -t ed25519
# یا
ssh-keygen -t rsa -b 4096
```

کپی کردن public key روی سرور (روش ساده):

```bash
ssh-copy-id user@server
```

یا دستی:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

## گزینه‌های مهم در sshd_config

چند گزینه کلیدی:

```ini
# غیرفعال کردن login مستقیم root (پیشنهادی)
PermitRootLogin no

# فقط اجازه به کلیدها (در صورت نیاز)
PasswordAuthentication no

# محدود کردن کاربران مجاز
AllowUsers user1 user2

# کنترل Port
Port 22

# فعال/غیرفعال کردن X11 forwarding
X11Forwarding no
```

بعد از تغییر تنظیمات:

```bash
sshd -t
systemctl restart sshd
```

## scp و sftp

کپی فایل با scp:

```bash
scp file.txt user@server:/tmp/
scp -r dir/ user@server:/tmp/
```

انتقال تعاملی با sftp:

```bash
sftp user@server
```

## ssh-agent

ssh-agent برای نگهداری کلیدها در حافظه استفاده می‌شود تا مجبور نباشید هر بار passphrase را وارد کنید:

```bash
eval "$(ssh-agent)"
ssh-add ~/.ssh/id_ed25519
```

## Port forwarding

### Local forwarding

وقتی می‌خواهیم یک سرویس روی شبکه داخلی را به لوکال خودمان بیاوریم:

```bash
ssh -L 8080:127.0.0.1:80 user@server
```

### Remote forwarding

وقتی می‌خواهیم یک پورت روی سرور remote به سمت لوکال ما تونل شود:

```bash
ssh -R 2222:127.0.0.1:22 user@server
```

### Dynamic forwarding (SOCKS proxy)

```bash
ssh -D 1080 user@server
```

## نکات امنیتی مهم

* استفاده از key-based authentication به جای password (در صورت امکان)
* محدود کردن کاربران مجاز و غیرفعال کردن root login
* تغییر Port به تنهایی امنیت نیست، اما noise حملات را کم می‌کند
* بررسی logها (مثلاً `/var/log/auth.log` یا journal)
