# 203.2. نگهداری فایل‌سیستم لینوکس

## **203.2 نگهداری فایل‌سیستم لینوکس**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند با استفاده از ابزارهای سیستم، فایل‌سیستم‌های لینوکسی را به درستی نگهداری کنند. این هدف شامل کار با فایل‌سیستم‌های استاندارد و مانیتور کردن دستگاه‌های SMART است.

**نواحی کلیدی دانش:**

* ابزارها و utilityها برای کار با ext2/ext3/ext4
* ابزارها و utilityها برای عملیات پایه Btrfs (subvolume و snapshot)
* ابزارها و utilityها برای XFS
* آگاهی از ZFS

**اصطلاحات و ابزارها:**

* mkfs (mkfs.*)
* mkswap
* fsck (fsck.*)
* tune2fs, dumpe2fs, debugfs
* btrfs, btrfs-convert
* xfs_info, xfs_check, xfs_repair, xfsdump, xfsrestore
* smartd, smartctl

## مرور فایل‌سیستم‌ها

در LPIC-1 با چند فایل‌سیستم آشنا شدیم. در LPIC-2 بیشتر با نگهداری و ابزارهای عملیاتی کار داریم.

## ساخت فایل‌سیستم (mkfs)

`mkfs` یک wrapper برای ابزارهای فایل‌سیستم‌های مختلف است.

مثال ساخت ext4:

```bash
mkfs -t ext4 /dev/sdb1
# یا
mkfs.ext4 /dev/sdb1
```

## بررسی و تعمیر فایل‌سیستم (fsck)

`fsck` برای بررسی/تعمیر فایل‌سیستم‌ها استفاده می‌شود. توجه کنید که معمولاً fsck باید روی فایل‌سیستم unmount شده اجرا شود.

مثال:

```bash
umount /dev/sdb1
fsck -f /dev/sdb1
```

## ابزارهای ext2/ext3/ext4

### dumpe2fs

نمایش اطلاعات فایل‌سیستم ext*:

```bash
dumpe2fs /dev/sdb1 | less
```

### tune2fs

تغییر برخی تنظیمات ext* (با احتیاط):

```bash
tune2fs -l /dev/sdb1
```

## Btrfs (مفاهیم پایه)

در Btrfs مفهوم subvolume و snapshot وجود دارد.

نمونه:

```bash
btrfs subvolume list /mnt
btrfs subvolume snapshot /mnt/@ /mnt/@_snap
```

## XFS

XFS برای فایل‌سیستم‌های بزرگ و workloadهای خاص استفاده می‌شود.

نمونه ابزارها:

```bash
xfs_info /mountpoint
xfs_repair /dev/sdb1
```

## SSD و TRIM (fstrim)

برای SSDها، TRIM می‌تواند به حفظ عملکرد کمک کند (بسته به پشتیبانی دستگاه/filesystem):

```bash
fstrim -av
```

## SMART (smartctl/smartd)

SMART اطلاعات سلامت دیسک را ارائه می‌دهد.

بررسی وضعیت:

```bash
smartctl -a /dev/sda
```

فعال/غیرفعال کردن و تست (مثال):

```bash
smartctl -s on /dev/sda
smartctl -t short /dev/sda
```

برای مانیتورینگ دائمی معمولاً از `smartd` استفاده می‌شود.
