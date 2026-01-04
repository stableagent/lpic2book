### **202.3 Bootloaderهای جایگزین**

**وزن:** 2

**توضیحات:** داوطلبان باید از bootloaderهای دیگر و ویژگی‌های اصلی آن‌ها آگاه باشند.

**نواحی کلیدی دانش:**

* SYSLINUX، ISOLINUX، PXELINUX
* درک PXE برای هر دو BIOS و UEFI
* آگاهی از systemd-boot و U-Boot

**اصطلاحات و ابزارها:**

* syslinux
* extlinux
* isolinux.bin
* isolinux.cfg
* isohdpfx.bin
* efiboot.img
* pxelinux.0
* pxelinux.cfg/
* uefi/shim.efi
* uefi/grubx64.efi

#### Linux Boot Loader

جد همه bootloaderهای لینوکس LiLo (Linux boot Loader) است. LiLo فایل پیکربندی خود را در /etc/lilo.conf دارد که به دودویی کامپایل می‌شد و در بخش‌های اول دیسک سخت قرار می‌گرفت. اما تمام آن روزهای خوب سادگی گذشته است.

```
#نمونه lilo.conf سیستم پیکربندی شده برای بوت 2 سیستم عامل
boot=/dev/hda
map=/boot/map
install=/boot/boot.b
prompt
timeout=50
message=/boot/message
lba32
default=linux

image=/boot/vmlinuz-2.4.0-0.43.6
    label=linux
    initrd=/boot/initrd-2.4.0-0.43.6.img
    read-only
    root=/dev/hda5

other=/dev/hda1
    label=dos
```

### SYSLINUX

SYSLINUX bootloader ساده‌ای است که عمدتاً برای بوت از فایل‌سیستم FAT استفاده می‌شود. معمولاً برای:

* دیسک‌های بوت USB
* فلش درایوها
* پارتیشن‌های FAT

نصب SYSLINUX:
```
syslinux --install /dev/sdb1
```

### ISOLINUX

ISOLINUX برای بوت از CD/DVD استفاده می‌شود. معمولاً در ISOهای لایو استفاده می‌شود.

فایل‌های مورد نیاز:
* `isolinux.bin`: bootloader اصلی
* `isolinux.cfg`: فایل پیکربندی
* `isohdpfx.bin`: هدر El Torito

### PXELINUX

PXELINUX برای بوت از شبکه استفاده می‌شود. بر اساس DHCP و TFTP کار می‌کند.

### systemd-boot

systemd-boot bootloader ساده‌ای است که با systemd کار می‌کند. در سیستم‌های UEFI استفاده می‌شود.

### U-Boot

U-Boot bootloader قدرتمندی است که عمدتاً در سیستم‌های embedded استفاده می‌شود.

### مقایسه bootloaderها

| ویژگی | GRUB 2 | SYSLINUX | systemd-boot | U-Boot |
| :--- | :--- | :--- | :--- | :--- |
| پشتیبانی BIOS | بله | بله | خیر | بله |
| پشتیبانی UEFI | بله | محدود | بله | بله |
| پیچیدگی | متوسط | ساده | ساده | پیچیده |
| استفاده رایج | دسکتاپ/سرور | USB/CD | سیستم‌های مدرن | embedded |

### انتخاب bootloader

انتخاب bootloader بستگی به:

* نوع سیستم (BIOS/UEFI)
* نیازهای پروژه
* سطح تخصص مورد نیاز
* محیط استفاده (دسکتاپ، سرور، embedded)

### نکات مهم

* همیشه از bootloader مناسب برای سیستم خود استفاده کنید
* پیکربندی را بررسی کنید قبل از نصب
* از فایل‌های پیکربندی بکاپ بگیرید
* برای سیستم‌های production از bootloaderهای پایدار استفاده کنید