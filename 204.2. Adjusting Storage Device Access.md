# 204.2. تنظیم دسترسی و تنظیمات دستگاه‌های ذخیره‌سازی

## **204.2 تنظیم دسترسی و تنظیمات دستگاه‌های ذخیره‌سازی**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند گزینه‌های kernel را برای پشتیبانی از انواع driveها پیکربندی کنند. این هدف همچنین شامل ابزارهای نرم‌افزاری برای مشاهده و تغییر تنظیمات دیسک (از جمله iSCSI) و تحلیل منابع سیستم است.

**نواحی کلیدی دانش:**

* ابزارها و utilityها برای تنظیم DMA برای دستگاه‌های IDE (شامل ATAPI و SATA)
* ابزارها و utilityها برای SSDها (شامل AHCI و NVMe)
* ابزارها و utilityها برای تحلیل منابع سیستم (مثل interruptها)
* آگاهی از sdparm و کاربردهای آن
* ابزارها و utilityهای iSCSI
* آگاهی از SAN و پروتکل‌های مرتبط (AoE, FCoE)

**اصطلاحات و ابزارها:**

* hdparm, sdparm
* nvme
* tune2fs
* fstrim
* sysctl
* /dev/hd*, /dev/sd*, /dev/nvme*
* iscsiadm, scsi_id, iscsid, iscsid.conf
* WWID, WWN, LUN

در این هدف با چند مفهوم و ابزار مهم حوزه storage آشنا می‌شویم. نکته مهم این است که برخی ابزارها مستقیماً با سخت‌افزار کار می‌کنند و استفاده اشتباه از آن‌ها می‌تواند باعث از دست رفتن داده شود.

## انواع storage (مرور سریع)

* **HDD**: هاردهای مکانیکی سنتی
* **SSD**: حافظه فلش غیر فرّار (non-volatile)
* **NVMe**: SSDهایی که از طریق PCIe و پروتکل NVMe کار می‌کنند

از منظر اتصال:

* IDE/ATA
* SATA
* SCSI/SAS
* Fibre Channel (FC)
* iSCSI (روی IP)

## hdparm

`hdparm` برای مشاهده و تغییر برخی پارامترهای دیسک‌های ATA/SATA استفاده می‌شود (مثل DMA، cache و power management).

نمونه نمایش اطلاعات:

```bash
hdparm -I /dev/sda
```

توجه: تغییر برخی گزینه‌ها (خصوصاً cache/power) ممکن است روی پایداری/ایمنی داده اثر بگذارد.

## sdparm

`sdparm` ابزار مشابهی برای دستگاه‌های SCSI/SAS است و می‌تواند برای مشاهده/تغییر برخی mode pageها استفاده شود. برای LPIC-2 کافی است بدانید وجود دارد و در برخی سناریوها جایگزین hdparm برای SCSI محسوب می‌شود.

## NVMe

برای NVMe معمولاً ابزار `nvme` استفاده می‌شود:

```bash
nvme list
nvme smart-log /dev/nvme0
```

## TRIM برای SSD (fstrim)

اگر SSD و filesystem از TRIM پشتیبانی کند، می‌توان با `fstrim` بلاک‌های آزاد را به SSD اعلام کرد:

```bash
fstrim -av
```

## منابع سیستم (interruptها و ...)

برای بررسی interruptها:

```bash
cat /proc/interrupts
```

و برای برخی تنظیمات runtime از `sysctl` استفاده می‌شود.

## iSCSI (مرور)

iSCSI اجازه می‌دهد storage از طریق شبکه IP ارائه شود.

اجزای رایج:

* **target**: سمت storage
* **initiator**: سمت client

ابزارهای رایج:

* `iscsiadm`
* سرویس `iscsid`

نمونه discovery (بسته به سناریو):

```bash
iscsiadm -m discovery -t sendtargets -p 192.168.1.50
```

و سپس login:

```bash
iscsiadm -m node --login
```

## SAN و مفاهیم WWN/WWID/LUN

در محیط‌های SAN معمولاً مفاهیمی مثل WWN/WWID و LUN برای شناسایی مسیرها و واحدهای ذخیره‌سازی استفاده می‌شوند. برای LPIC-2 باید با این اصطلاحات آشنا باشید.
