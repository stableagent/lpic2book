# 206.2. عملیات پشتیبان‌گیری

## **206.2 عملیات پشتیبان‌گیری**

**وزن:** 3

**توضیحات:** داوطلبان باید بتوانند از ابزارهای سیستم برای پشتیبان‌گیری از داده‌های مهم سیستم استفاده کنند.

**نواحی کلیدی دانش:**

* آشنایی با دایرکتوری‌هایی که باید در backupها لحاظ شوند
* آگاهی از راهکارهای backup شبکه‌ای مثل Amanda، Bacula، Bareos و BackupPC
* آشنایی با مزایا و معایب tape، CDR، دیسک یا سایر رسانه‌های backup
* انجام backupهای دستی و جزئی (partial)
* بررسی یکپارچگی (integrity) فایل‌های backup
* بازیابی (restore) جزئی یا کامل از backup

**اصطلاحات و ابزارها:**

* /bin/sh
* dd
* tar
* /dev/st\* و /dev/nst\*
* mt
* rsync

## چرا به Backup نیاز داریم؟

قبلاً درباره RAID و LVM صحبت کرده‌ایم. هرچند ساختن گروه RAID یا ساختن LVM تا حدی قابلیت اطمینان و امنیت ایجاد می‌کند، اما به عنوان راهکار backup در نظر گرفته نمی‌شوند. ما می‌توانیم به دلایل مختلف داده از دست بدهیم یا دچار failure شویم:

* قطع شدن برق
* خرابی سخت‌افزار (mobo، cpu، ram، hard disks، ...)
* اشتباهات انسانی و misconfiguration
* ...

واقعیت این است که دلیل آخر رایج‌ترین و خطرناک‌ترینِ آن‌هاست. برای جلوگیری از این موارد باید backup بگیریم؛ مجبوریم backup بگیریم.

## از چه چیزهایی Backup بگیریم؟

همه دایرکتوری‌ها و فایل‌ها لازم نیست backup شوند، مخصوصاً وقتی مشکل فضای ذخیره‌سازی backup داریم. در File System Hierarchy Standard (FHS) لینوکس، دایرکتوری‌ها و فایل‌ها برای backup کردن اولویت‌های متفاوتی دارند:

| دایرکتوری   | اولویت | توضیحات |
| ----------- | ------ | ------- |
| /etc/       | بالا   | فایل‌های تنظیمات سراسری سیستم که برای همه برنامه‌ها لازم هستند |
| /home/      | بالا   | Home directory کاربران برای نگهداری فایل‌های شخصی |
| /usr/local/ | بالا   | شامل برنامه‌هایی که کاربران از سورس نصب کرده‌اند |
| /var/lib/   | متوسط  | شامل داده‌های زیاد؛ گرفتن backup کامل این بخش معمولاً امن‌تر است |
| /var/mail/  | متوسط  | ایمیل‌های local |
| /var/www/   | متوسط  | web root پیش‌فرض |
| /var/spool/ | متوسط  | صف‌های چاپ (و ممکن است توسط برخی برنامه‌ها استفاده شود) |
| /var/log/   | پایین  | فایل‌های log سیستم |
| /opt/       | متوسط  | شامل برنامه‌های add-on از vendorهای مختلف |
| /usr/       | پایین  | شامل binaryها، libraryها، مستندات و سورس‌کد برنامه‌های سطح دوم |

## چطور Backup بگیریم؟

در هر پلتفرمی معمولاً هم ابزارهای داخلی (native) و هم برنامه‌های third-party برای backup وجود دارد:

| Package                                | Licence  | Language | رابط گرافیکی (GUI)               | رابط خط فرمان (CMD) |
| -------------------------------------- | -------- | -------- | -------------------------------- | ------------------- |
| ![](.gitbook/assets/logo_amanda.png)   | BSD      | C , Perl | خیر (مگر در Amanda Enterprise)   | بله                 |
| ![](.gitbook/assets/logo-backuppc.gif) | GPLv2.0  | Perl     | بله                              | بله                 |
| ![](.gitbook/assets/logo-bacula.png)   | AGPLv3.0 | C , C++  | بله                              | بله                 |

هر سه package نسخه‌های Linux، Windows و MacOS دارند. حالا کمی وقت می‌گذاریم و درباره چند ابزار «سنتی/داخلی» برای backup صحبت می‌کنیم.

### tape

استفاده از tape برای backup تا حدی قدیمی شده است، اما هنوز هم استفاده می‌شود چون ارزان است و ظرفیت بالایی دارد؛ البته خیلی کند است. اگر تجربه مدیریت سیستمی را داشته باشید که دستگاه tape به آن وصل باشد، این deviceها را می‌بینید:

* /dev/st0
* /dev/nst0

دستگاه `/dev/nst0` یک non-rewinding tape device است، در حالی که `/dev/st0` یک rewinding tape device است. اینکه کدام را استفاده کنید به هدف شما بستگی دارد. هر دو برای یک سخت‌افزار هستند اما رفتار متفاوتی دارند. می‌توانیم `/dev/st0` را با نرم‌افزار rewind کنیم، اما برای `/dev/nst0` این‌طور نیست و باید به صورت فیزیکی rewind شود.

### درک tape file mark و block size

![](.gitbook/assets/tape-tapefilemark.jpg)

هر دستگاه tape می‌تواند چندین فایل backup را ذخیره کند. فایل‌های backup روی tape می‌توانند با ابزارهایی مثل `cpio`، `tar`، `dd` و ... ساخته شوند. دستگاه tape توسط برنامه‌های مختلف می‌تواند open شود، روی آن داده نوشته شود و سپس close شود. ما می‌توانیم چندین backup (فایل tape) را روی یک tape فیزیکی ذخیره کنیم. بین هر فایل tape یک «tape file mark» وجود دارد؛ این علامت مشخص می‌کند یک فایل tape کجا تمام می‌شود و فایل بعدی روی tape فیزیکی کجا شروع می‌شود. برای جابه‌جا کردن موقعیت tape (جلو/عقب بردن و رفتن روی markها) باید از دستور `mt` استفاده کنید.

### دستور mt

دستور `mt` برای کنترل عملیات tape drive استفاده می‌شود؛ مثل گرفتن status، جابه‌جایی بین فایل‌های روی tape یا نوشتن tape control mark روی tape.

| چند نمونه از دستور mt      | توضیحات |
| -------------------------- | ------- |
| mt -f /dev/st0 rewind      | rewind کردن tape drive |
| mt -f /dev/st0 status      | نمایش status دستگاه tape |
| mt -f /dev/st0 erase       | پاک کردن tape |
| mt -f /dev/st0 eject       | eject کردن tape drive |
| mt -f /dev/st0 eof         | نوشتن n عدد EOF mark در موقعیت فعلی tape |

لیست دستورهای مربوط به موقعیت‌یابی (position) tape:

```
       fsf    Forward space count files.  The tape is positioned on the first block of the next file.

       fsfm   Forward space count files.  The tape is positioned on the last block of the previous file.

       bsf    Backward space count files.  The tape is positioned on the last block of the previous file.

       bsfm   Backward space count files.  The tape is positioned on the first block of the next file.

       asf    The tape is positioned at the beginning of the count file.
              Positioning is done by first rewinding the tape and then spacing forward over count filemarks.

       fsr    Forward space count records.

       bsr    Backward space count records.

       fss    (SCSI tapes) Forward space count setmarks.

       bss    (SCSI tapes) Backward space count setmarks.
```

و گزینه‌های بسیار زیاد دیگر.

### داده روی tape drive چطور ذخیره می‌شود؟

![](.gitbook/assets/tape-hawstored.jpg)

تمام داده‌ها به صورت sequential در قالب tape archive و معمولاً با استفاده از `tar` ذخیره می‌شوند. اولین archive از ابتدای فیزیکی tape شروع می‌شود (tar #0)، بعدی tar #1 و به همین ترتیب.

## tar

ما از `tar` (مخفف tape archive) برای ساختن فایل‌های tar استفاده می‌کنیم، اما در واقع tar از ابتدا برای آرشیو کردن فایل‌ها روی دستگاه tape طراحی شده بود.

```
root@server1:~# tree mydirectory/
mydirectory/
├── dir1
│   └── file1.txt
├── dir2
│   └── file2.txt
├── dir3
│   └── file3.txt
└── myfile

3 directories, 4 files
root@server1:~# tar -cvf backup-mydirectory.tar mydirectory/
mydirectory/
mydirectory/dir3/
mydirectory/dir3/file3.txt
mydirectory/myfile
mydirectory/dir1/
mydirectory/dir1/file1.txt
mydirectory/dir2/
mydirectory/dir2/file2.txt

root@server1:~# ls
backup-mydirectory.tar  mydirectory  pooler-cpuminer-2.5.0.tar.gz
cpuminer                mynfs
```

برای اینکه فایل‌ها را از backup برگردانیم، چند فایل/دایرکتوری را حذف می‌کنیم:

```
root@server1:~# rm -rf mydirectory/myfile , mydirectory/dir3
root@server1:~# tree mydirectory/
mydirectory/
├── dir1
│   └── file1.txt
└── dir2
    └── file2.txt

2 directories, 2 files
root@server1:~# ls
backup-mydirectory.tar  mydirectory  pooler-cpuminer-2.5.0.tar.gz
cpuminer                mynfs
```

برای لیست کردن فایل‌های داخل tar، از سوئیچ‌های `-tvf` استفاده می‌کنیم:

```
root@server1:~# tar -tvf backup-mydirectory.tar 
drwxr-xr-x root/root         0 2018-02-05 03:54 mydirectory/
drwxr-xr-x root/root         0 2018-02-05 03:54 mydirectory/dir3/
-rw-r--r-- root/root         0 2018-02-05 03:54 mydirectory/dir3/file3.txt
-rw-r--r-- root/root         0 2018-02-05 03:53 mydirectory/myfile
drwxr-xr-x root/root         0 2018-02-05 03:54 mydirectory/dir1/
-rw-r--r-- root/root         0 2018-02-05 03:54 mydirectory/dir1/file1.txt
drwxr-xr-x root/root         0 2018-02-05 03:54 mydirectory/dir2/
-rw-r--r-- root/root         0 2018-02-05 03:54 mydirectory/dir2/file2.txt
```

حالا باید `myfile` و `dir3` را از backup برگردانیم:

```
root@server1:~# tar -xvf backup-mydirectory.tar  mydirectory/myfile
mydirectory/myfile
root@server1:~# tree mydirectory/
mydirectory/
├── dir1
│   └── file1.txt
├── dir2
│   └── file2.txt
└── myfile

2 directories, 3 files
root@server1:~# tar -xvf backup-mydirectory.tar  mydirectory/dir3/file3.txt 
mydirectory/dir3/file3.txt
root@server1:~# tree mydirectory/
mydirectory/
├── dir1
│   └── file1.txt
├── dir2
│   └── file2.txt
├── dir3
│   └── file3.txt
└── myfile

3 directories, 4 files
```

بیایید چند سوئیچ پرکاربرد tar را به شکل یک مرور سریع جمع‌بندی کنیم:

| دستور tar                                     | توضیحات |
| --------------------------------------------- | ------- |
| tar -cvf mybackup.tar myfiles/                | ساخت فایل backup با tar |
| tar -cvzf mybackup.tar.gz myfiles/            | ساخت فایل tar.gz با gzip |
| tar -cvjf mybackup.tar.bz2 myfiles/           | ساخت فایل tar.bz2 با bzip2 |
| tar -xvf mybackup.tar                         | extract کردن فایل‌های tar/tar.gz/tar.bz2 |
| tar -tvf mybackup.tar                         | لیست کردن محتوای tar/tar.gz/tar.bz2 |
| tar -xvf mybackup.tar myfile                  | extract کردن یک فایل از tar/tar.gz/tar.bz2 |
| tar -xvf mybackup.tar "file1.txt" "file2.txt" | extract کردن چند فایل از tar/tar.gz/tar.bz2 |
| tar -xvf mybackup.tar --wildcards '\*.conf'   | extract کردن مجموعه فایل‌ها با wildcard |
| tar -rvf mybackup.tar xyz.txt                 | اضافه کردن فایل/دایرکتوری به tar/tar.gz/tar.bz2 |
| tar -xvfW mybackup.tar                        | verify کردن archive فایل tar/tar.gz/tar.bz2 |
| tar -czf mybackup.tar                         | بررسی اندازه فایل‌های tar/tar.gz/tar.bz2 |

## rsync

`rsync` (Remote Sync) یکی از رایج‌ترین دستورها برای کپی و همگام‌سازی فایل‌ها و دایرکتوری‌ها در لینوکس است؛ هم به صورت local و هم remote. با کمک rsync می‌توانیم داده‌ها را در مسیرهای مختلف، دیسک‌ها و شبکه‌ها کپی/همگام کنیم، backup بگیریم و بین دو ماشین لینوکسی mirroring انجام دهیم.

#### چند مزیت و ویژگی rsync:

* کپی و sync فایل‌ها به/از یک سیستم remote را به شکل کارآمد انجام می‌دهد.
* از کپی کردن linkها، deviceها، ownerها، groupها و permissionها پشتیبانی می‌کند.
* معمولاً از `scp` (Secure Copy) سریع‌تر است. چرا؟ چون rsync از remote-update protocol استفاده می‌کند و فقط تفاوت‌ها بین دو مجموعه فایل را منتقل می‌کند. بار اول کل محتوا را کپی می‌کند، اما دفعات بعد فقط blockها و byteهای تغییر کرده را منتقل می‌کند.
* پهنای باند کمتری مصرف می‌کند چون هنگام ارسال/دریافت داده از compression و decompression استفاده می‌کند.

ممکن است لازم باشد rsync را با `yum install rsync` یا در Debian با `apt install rsync` نصب کنیم.

Syntax پایه rsync به شکل `rsync options source destination` است. چند گزینه رایج:

| گزینه‌های رایج rsync | توضیحات |
| -------------------- | ------- |
| -v                   | حالت verbose |
| -r                   | کپی کردن به صورت recursive (بدون حفظ timestamp و permission) |
| -a                   | archive mode؛ کپی recursive و همچنین حفظ symbolic linkها، permissionها، مالکیت user/group و timestampها |
| -z                   | فشرده‌سازی داده هنگام انتقال |
| -h                   | human-readable؛ نمایش خروجی در قالب قابل خواندن |

بس است؛ rsync را در عمل ببینیم:

```
root@server1:~# ls
backup-mydirectory.tar  mydirectory  pooler-cpuminer-2.5.0.tar.gz
cpuminer                mynfs

### copy / sync a file on a local computer
root@server1:~# rsync -zvh backup-mydirectory.tar /tmp/backups/
backup-mydirectory.tar

sent 316 bytes  received 35 bytes  702.00 bytes/sec
total size is 10.24K  speedup is 29.17

### copy/sync a directory on a local computer
root@server1:~# tree mydirectory/
mydirectory/
├── dir1
│   └── file1.txt
├── dir2
│   └── file2.txt
├── dir3
│   └── file3.txt
└── myfile

3 directories, 4 files

root@server1:~# rsync -avzh mydirectory/ /tmp/backups/
sending incremental file list
./
myfile
dir1/
dir1/file1.txt
dir2/
dir2/file2.txt
dir3/
dir3/file3.txt

sent 379 bytes  received 115 bytes  988.00 bytes/sec
total size is 0  speedup is 0.00
```

و برای Copy یک دایرکتوری از سرور local به یک سرور remote:

```
root@server1:~# rsync -azv mydirectory root@192.168.10.151:/home/
root@192.168.10.151's password: 
sending incremental file list
mydirectory/
mydirectory/myfile
mydirectory/dir1/
mydirectory/dir1/file1.txt
mydirectory/dir2/
mydirectory/dir2/file2.txt
mydirectory/dir3/
mydirectory/dir3/file3.txt

sent 389 bytes  received 112 bytes  143.14 bytes/sec
total size is 0  speedup is 0.00
```

نتیجه:

```
root@server2:/home# ls
mydirectory  payam
```

و برعکس؛ Copy/Sync یک دایرکتوری remote به یک ماشین local:

```
root@192.168.10.151's password: 
receiving incremental file list
mydirectory/
mydirectory/myfile
mydirectory/dir1/
mydirectory/dir1/file1.txt
mydirectory/dir2/
mydirectory/dir2/file2.txt
mydirectory/dir3/
mydirectory/dir3/file3.txt

sent 124 bytes  received 389 bytes  146.57 bytes/sec
total size is 0  speedup is 0.00

root@server1:~# ls /tmp/
backups
_cafenv-appconfig_
config-err-aoEHBZ
mydirectory
systemd-private-5c9b83ef1a904073864354a680e17c01-colord.service-JNMAsj
systemd-private-5c9b83ef1a904073864354a680e17c01-rtkit-daemon.service-wp0M3z
systemd-private-5c9b83ef1a904073864354a680e17c01-systemd-timesyncd.service-fh7T2M
unity_support_test.0
VMwareDnD
vmware-payam
vmware-root
```

### rsync روی SSH

اغلب، rsync روی SSH اجرا می‌شود.
در موارد نادر ممکن است کسی یک rsync daemon راه‌اندازی کرده باشد که از پورت 873 استفاده می‌کند:

```
root@server1:~# cat /etc/services | grep rsync
rsync        873/tcp
rsync        873/udp
```

وقتی از SSH هنگام انتقال داده استفاده می‌کنیم مطمئن هستیم داده‌ها روی یک اتصال امن و رمزگذاری‌شده منتقل می‌شوند و کسی نمی‌تواند در حین انتقال آن‌ها را بخواند.

وقتی از rsync استفاده می‌کنیم معمولاً لازم است password کاربر/root را برای انجام آن کار بدهیم؛ پس گزینه SSH باعث می‌شود loginها به صورت رمزگذاری‌شده منتقل شوند و password امن بماند. با گزینه `-e` مطمئن می‌شویم rsync را روی ssh اجرا می‌کنیم:

```
### Copy a Directory from a Local Server to a Remote Server with SSH
root@server1:~# rsync -azve ssh mydirectory root@192.168.10.151:/home/
root@192.168.10.151's password: 
sending incremental file list

sent 233 bytes  received 20 bytes  72.29 bytes/sec
total size is 0  speedup is 0.00

### Copy a directory from a Remote Server to a Local Server with SSH
root@server1:~# rsync -azve ssh root@192.168.10.151:/home/mydirectory /tmp/
root@192.168.10.151's password: 
receiving incremental file list

sent 28 bytes  received 225 bytes  33.73 bytes/sec
total size is 0  speedup is 0.00
```

چند دستور مفید دیگر rsync:

* نمایش progress هنگام انتقال داده:

`rsync -azve ssh --progress mydirectory root@192.168.10.151:/home/`

* include و exclude:

`rsync -azve ssh --include 'D*' --exclude '*' mydirectory root@192.168.10.151:/home/` : فقط فایل‌ها/دایرکتوری‌هایی که با `D` شروع می‌شوند include می‌شوند و بقیه exclude می‌شوند.

* گزینه delete

اگر یک فایل یا دایرکتوری در source وجود نداشته باشد اما در destination وجود داشته باشد، ممکن است بخواهید هنگام sync شدن آن فایل/دایرکتوری در مقصد حذف شود.

می‌توانیم از گزینه `--delete` استفاده کنیم تا فایل‌هایی که در source نیستند حذف شوند:

`rsync -azv --delete root@192.168.10.151:/home/mydirectory`

| گزینه‌های مفید دیگر rsync (ممکن است در آزمون دیده شود) | توضیحات |
| ------------------------------------------------------ | ------- |
| --max-size='200K'                                      | تعیین حداکثر اندازه فایل‌هایی که منتقل می‌شوند |
| --remove-source-files                                  | حذف خودکار فایل‌های source بعد از انتقال موفق |
| --bwlimit=100                                          | محدود کردن پهنای باند انتقال |
| --dry-run                                              | اجرای آزمایشی؛ کاری انجام نمی‌دهد و فقط نشان می‌دهد چه انجام خواهد شد |

به صورت پیش‌فرض rsync فقط blockها و byteهای تغییر کرده را sync می‌کند. اگر می‌خواهید صراحتاً کل فایل sync شود می‌توانید از گزینه `-W` استفاده کنید:

```
rsync -zvhW backup.tar /tmp/backups/backup.tar
```

## dd

دستور `dd` مخفف «data duplicator» است و برای کپی کردن و تبدیل داده استفاده می‌شود. `dd` یک ابزار سطح پایین و بسیار قدرتمند در لینوکس است. باید هنگام کار با آن بسیار مراقب باشیم؛ اگر اشتباه کنیم می‌تواند باعث از دست رفتن داده شود و عملاً `dd` را برای ما به «data destroyer» تبدیل کند. به همین دلیل توصیه می‌شود تا زمانی که با آن کاملاً آشنا نشده‌اید، روی ماشین production از dd استفاده نکنید.

از dd می‌توان برای clone کردن volumeها و filesystemها، نوشتن image روی دیسک‌ها و حتی پاک کردن درایوها استفاده کرد. Syntax دستور dd به شکل `dd if=<source file name> of=<target file name> [Options]` است.

| دستور dd                                      | توضیحات |
| --------------------------------------------- | ------- |
| dd if=/dev/sda of=/dev/sdb                    | Clone کردن یک هارد به هارد دیگر |
| dd if=/dev/sda2 of=~/hddpar1.img              | backup گرفتن از یک پارتیشن در یک فایل |
| dd if=hddpar1.img of=/dev/sdb1                | restore کردن image روی دیسک/پارتیشن دیگر |
| dd if=/dev/sda2 \| bzip2 hddpar1.img.bz2      | استفاده از bzip2 برای فشرده‌سازی هنگام ساخت image |
| dd if=/home/myuser/abc.txt of=/mnt/abc.txt    | استفاده از dd به عنوان file copier |

`dd` می‌تواند خطرناک باشد؛ هنگام استفاده از آن حتماً مراقب باشید.
