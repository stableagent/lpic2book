**203.3 ایجاد و پیکربندی گزینه‌های فایل‌سیستم**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند فایل‌سیستم‌های mount خودکار را با استفاده از AutoFS پیکربندی کنند. این هدف شامل پیکربندی automount برای فایل‌سیستم‌های شبکه و دستگاه است. همچنین شامل ایجاد فایل‌سیستم‌ها برای دستگاه‌هایی مانند CD-ROMها و دانش ویژگی‌های پایه فایل‌سیستم‌های رمزنگاری شده است.

**نواحی کلیدی دانش:**

* فایل‌های پیکربندی autofs
* درک units automount
* ابزارها و utilities UDF و ISO9660
* آگاهی از فایل‌سیستم‌های CD-ROM دیگر (HFS)
* آگاهی از extensions فایل‌سیستم CD-ROM (Joliet, Rock Ridge, El Torito)
* دانش ویژگی‌های پایه رمزنگاری داده (dm-crypt / LUKS)

**اصطلاحات و ابزارها:**

* /etc/auto.master
* /etc/auto.[dir]
* mkisofs
* cryptsetup

#### autofs

ما قبلاً با fstab و استفاده از آن کار کرده‌ایم. وقتی دستگاهی را با استفاده از fstab mount می‌کنیم، همیشه mount شده و آماده است. این خوب است مگر اینکه از NFS، CIFS، SMB، ... از طریق شبکه استفاده کنیم. ایده autofs این است که وقتی نیاز داری mount کنی. این همه! به این ترتیب از network overhead جلوگیری می‌کنیم.

```
root@server1:~# apt install autofs
```

autofs فایل‌های پیکربندی خود را در دایرکتوری /etc قرار می‌دهد:

```
root@server1:~# ls -l /etc/auto*
-rw-r--r-- 1 root root 12596 Jun 22  2017 /etc/autofs.conf
-rw-r--r-- 1 root root   797 Jun 22  2017 /etc/auto.master
-rw-r--r-- 1 root root   524 Jun 22  2017 /etc/auto.misc
-rwxr-xr-x 1 root root  1039 Jun 22  2017 /etc/auto.net
-rwxr-xr-x 1 root root  2191 Jun 22  2017 /etc/auto.smb
```

#### auto.master

auto.master پیکربندی اصلی autofs است و اولین فایل پیکربندی است که autofs بررسی می‌کند. داخل auto.master ما ذکر می‌کنیم که mount-point را کجا می‌خواهیم و فایل پیکربندی مرتبط کجاست. عجیب است اما این روش کار autofs است. پس کنار فایل auto.master، ممکن است به فایل‌های پیکربندی auto.[*] دیگری نیاز داشته باشید که auto.master به آن‌ها اشاره می‌کند تا ایجاد و mount کند. فرمت master map:

```
mount-point map-name options
```

![](/assets/autofs.jpg)

و نتیجه /mynfs/dir1 خواهد بود. بیایید دست‌هایمان را کثیف کنیم و ببینیم داخل auto.master چه چیزی هست:

```
#
# نمونه فایل auto.master
# این یک 'master' automounter map است و فرمت زیر را دارد:
# mount-point [map-type[,format]:]map [options]
# برای جزئیات فرمت به auto.master(5) مراجعه کنید.
#
#/misc    /etc/auto.misc
#
# توجه: mountهای انجام شده از یک hosts map با
#    گزینه‌های "nosuid" و "nodev" mount می‌شوند مگر اینکه گزینه‌های "suid" و "dev"
#    به صراحت داده شوند.
#
#/net    -hosts
#
# شامل /etc/auto.master.d/*.autofs
# فایل‌های شامل شده باید با فرمت این فایل مطابقت داشته باشند.
#
+dir:/etc/auto.master.d
#
# شامل master map مرکزی اگر بتوان با استفاده از
# منابع nsswitch پیدا کرد.
#
# توجه کنید که اگر برای /net یا /misc (مانند
# بالا) در master map شامل شده entries وجود داشته باشد، هر کلیدی که یکسان باشد
# دیده نخواهد شد زیرا اولین کلید خوانده شده اولویت دارد.
#
+auto.master

# اضافه شده توسط من و شما :)
/root/mynfs    /etc/auto.nfs
```

پس دایرکتوری mynfs ایجاد می‌شد و به عنوان mount point استفاده می‌شد، سپس به فایل پیکربندی /etc/auto.nfs اشاره می‌کند. می‌توانیم از `--timeout=60` برای تعریف timeout mount بر حسب ثانیه استفاده کنیم.

#### auto.nfs

`vi /etc/auto.nfs`

```
dir1    192.168.10.150:/root/nfsshared/mydir1
```

و سپس سرویس autofs را restart کنید تا تغییرات اعمال شود:

```
systemctl restart autofs
```

حالا بیایید تست کنیم:

```
ls /root/mynfs/dir1
```

### ISO9660

ISO9660 فایل‌سیستم استاندارد برای CD-ROM است. ابزارهای مهم:

#### mkisofs

برای ایجاد فایل ISO:

```
mkisofs -o image.iso -R -J directory/
```

گزینه‌ها:
* `-o`: فایل خروجی
* `-R`: پسوندهای Rock Ridge
* `-J`: پسوندهای Joliet

### UDF

UDF (Universal Disk Format) فایل‌سیستم مدرن برای DVD و Blu-ray است.

### رمزنگاری فایل‌سیستم

#### dm-crypt و LUKS

برای رمزنگاری فایل‌سیستم:

```
# ایجاد volume رمزنگاری شده
cryptsetup luksFormat /dev/sdb1

# باز کردن volume رمزنگاری شده
cryptsetup luksOpen /dev/sdb1 encrypted

# ایجاد فایل‌سیستم
mkfs.ext4 /dev/mapper/encrypted

# mount کردن
mount /dev/mapper/encrypted /mnt

# unmount و بستن
umount /mnt
cryptsetup luksClose encrypted
```

### HFS

HFS (Hierarchical File System) فایل‌سیستم Apple است.

### خلاصه

* autofs برای mount خودکار فایل‌سیستم‌های شبکه استفاده می‌شود
* ISO9660 فایل‌سیستم استاندارد CD-ROM است
* dm-crypt/LUKS برای رمزنگاری فایل‌سیستم استفاده می‌شود
* هر فایل‌سیستم کاربرد خاص خود را دارد