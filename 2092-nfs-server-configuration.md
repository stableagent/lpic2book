# 209.2. پیکربندی سرور NFS

## **209.2 پیکربندی سرور NFS**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند یک NFS server را پیکربندی کنند؛ شامل export کردن دایرکتوری‌ها، تنظیم دسترسی‌ها و عیب‌یابی اولیه.

**نواحی کلیدی دانش:**

* فایل‌ها و ابزارهای پیکربندی NFS
* /etc/exports و مفهوم export optionها
* سرویس‌های مرتبط مثل rpcbind
* ابزارهای تست و عیب‌یابی مثل showmount و mount

**اصطلاحات و ابزارها:**

* /etc/exports
* exportfs
* showmount
* mount
* nfsd
* rpcbind

## NFS چیست؟

NFS (Network File System) پروتکلی برای share کردن فایل‌سیستم روی شبکه است، به طوری که client بتواند یک دایرکتوری را از روی سرور mount کند.

## نصب و راه‌اندازی

* Debian/Ubuntu:

```bash
apt update
apt install nfs-kernel-server
systemctl enable --now nfs-server
```

* RHEL/CentOS:

```bash
yum install nfs-utils
systemctl enable --now nfs-server
```

در برخی سناریوها rpcbind هم لازم است:

```bash
systemctl enable --now rpcbind
```

## /etc/exports

دایرکتوری‌هایی که می‌خواهید share کنید در `/etc/exports` تعریف می‌شوند.

نمونه:

```conf
/srv/nfs/share 192.168.1.0/24(rw,sync,no_subtree_check)
```

چند گزینه مهم:

* `ro` / `rw`: فقط خواندنی یا خواندن/نوشتن
* `sync`: نوشتن‌ها با sync انجام شود (امن‌تر، کندتر)
* `no_root_squash`: mapping نکردن root (خطرناک؛ فقط در صورت نیاز)
* `root_squash`: تبدیل درخواست‌های root به کاربر nobody (پیش‌فرض امن)

بعد از تغییر exports:

```bash
exportfs -ra
systemctl reload nfs-server
```

## سمت client

نصب ابزار client (در صورت نیاز) و mount:

```bash
mount -t nfs server:/srv/nfs/share /mnt
```

برای mount دائمی از `/etc/fstab` استفاده می‌شود.

## تست و عیب‌یابی

دیدن exportهای سرور:

```bash
showmount -e server
```

بررسی سرویس‌ها:

```bash
systemctl status nfs-server
rpcinfo -p server
```

نکته مهم: اگر firewall فعال است، باید پورت‌های لازم برای NFS باز باشند (وابسته به نسخه NFS و توزیع).
