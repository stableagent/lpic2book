# 211.2. مدیریت تحویل E-Mail

## **211.2 مدیریت تحویل E-Mail**

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند نرم‌افزارهای مدیریت ایمیل در سمت کلاینت را پیاده‌سازی کنند تا ایمیل‌های ورودی کاربران را filter، sort و monitor کنند.

**نواحی کلیدی دانش:**

* درک کارکرد Sieve، syntax و operatorهای آن
* استفاده از Sieve برای filter و sort کردن ایمیل بر اساس sender، recipient(ها)، headerها و اندازه
* آگاهی از procmail

**اصطلاحات و ابزارها:**

* شرط‌ها (conditions) و operatorهای مقایسه (comparison operators)
* keep, fileinto, redirect, reject, discard, stop
* Dovecot vacation extension

در درس قبلی دیدیم که postfix ایمیل‌ها را در دایرکتوری `/var/spool/mail` ذخیره می‌کند. در این درس با نرم‌افزار `procmail` برای مدیریت تحویل ایمیل (mail delivery) و روش‌های مختلف انجام این کار آشنا می‌شویم.

## procmail ![](.gitbook/assets/mail-delivery-procmail.gif)

بیشتر MTAها (Mail Transfer Agent / Mail Server) با procmail کار می‌کنند یا حداقل امکان کار کردن با procmail را دارند.

![](.gitbook/assets/mail-delivery-procmail-concept.jpg)

`procmail` که یک Mail Delivery Agent (MDA) است، می‌تواند ایمیل‌های ورودی را به دایرکتوری‌های مختلف sort کند و پیام‌های spam را فیلتر کند. procmail نرم‌افزار پایداری است، اما دیگر نگهداری (maintain) نمی‌شود و از آخرین release آن تا امروز چندین آسیب‌پذیری امنیتی کشف شده است. نویسنده procmail (Philip Guenther) به کاربرانی که می‌خواهند از یک برنامه نگهداری‌شده استفاده کنند پیشنهاد می‌کند از ابزارهای جایگزین (مثل maildrop یا sieve) استفاده کنند، چون از سال 2001 به‌روزرسانی نشده است. با این حال چون برای آزمون LPIC-2 آماده می‌شویم، ادامه می‌دهیم.

postfix ایمیل‌های کاربران را در مسیر `/var/spool/mail/<user name>` ذخیره می‌کند. در این مسیر یک فایل ساخته می‌شود و هر ایمیلِ کاربر به انتهای همان فایل متنی بزرگ append می‌شود؛ در همین حین از فضای `/var` استفاده می‌کند.

procmail به ما اجازه می‌دهد ایمیل‌ها را در یکی از دایرکتوری‌های مشخص‌شده کاربر (معمولاً `/home/<user name>/mail/`) ذخیره کنیم و آن‌ها را sort کنیم.

procmail می‌تواند ایمیل‌ها را در دو فرمت ذخیره‌سازی نگه دارد: فرمت `mbox` یا فرمت `maildir`.

![](.gitbook/assets/mail-procmail-mbox-vs-mailddir.jpg)

`mbox` سیستم ذخیره‌سازی قدیمی‌تر است. در حالی که در mbox همه پیام‌ها در یک فایل روی سرور قرار می‌گیرند، در Maildir پیام‌ها در فایل‌های جداگانه (با نام‌های یکتا) ذخیره می‌شوند.

#### فرمت Mbox

Mbox فرمت سنتی ذخیره‌سازی ایمیل است. در این فرمت یک فایل متنی معمولی وجود دارد که نقش mailbox کاربر را بازی می‌کند. معمولاً نام این فایل `/var/spool/mail/<user name>` است. در mbox هنگام انجام عملیات روی mailbox، فایل lock می‌شود و بعد از انجام عملیات unlock می‌شود.

مزایا:

* فرمت mbox تقریباً همه‌جا پشتیبانی می‌شود.
* append کردن ایمیل جدید سریع است.
* جستجو در یک mailbox (یک فایل) سریع است.

معایب:

* mbox به مشکلات lock معروف است.
* فرمت mbox مستعد خراب شدن (corruption) است.

#### فرمت Maildir

Maildir فرمت جدیدتر ذخیره‌سازی ایمیل است. برای هر کاربر یک دایرکتوری maildir ساخته می‌شود (معمولاً داخل home directory کاربر). زیر دایرکتوری maildir به صورت پیش‌فرض سه دایرکتوری دیگر وجود دارد: `new`، `cur` و `tmp`.

مزایا:

* پیدا کردن، دریافت و حذف یک ایمیل خاص سریع است؛ مخصوصاً وقتی یک folder صدها پیام داشته باشد.
* نیاز به file locking بسیار کم است یا اصلاً لازم نیست.
* می‌تواند روی network file system استفاده شود.
* در برابر خرابی mailbox مقاوم است (به شرطی که سخت‌افزار خراب نشود).

معایب:

* بعضی filesystemها ممکن است با تعداد زیادی فایل کوچک به شکل کارآمد برخورد نکنند.
* جستجوی متنی (که نیاز دارد همه فایل‌های ایمیل باز شوند) کند است.

ممکن است procmail روی توزیع شما نصب باشد؛ در غیر این صورت نصبش کنید:

```
[root@localhost ~]# yum install procmail
```

با این کار فقط چند فایل binary نصب می‌شود که postfix از آن‌ها استفاده می‌کند. procmail هیچ سرویسی (service) ندارد و فایل‌های پیکربندی از پیش آماده هم ندارد!

Procmail به روش‌های مختلفی می‌تواند اجرا شود. وقتی ایمیل در spool file قرار می‌گیرد، می‌توان procmail را طوری تنظیم کرد که اجرا شود، ایمیل را بر اساس تنظیمات شما filter کند، آن را به locationهایی که MUA استفاده می‌کند ببرد و سپس خارج شود. یا می‌توان MUA را طوری تنظیم کرد که هر زمان پیام جدید دریافت شد procmail را اجرا کند تا پیام‌ها به mailbox درست منتقل شوند (کاری که اینجا انجام می‌دهیم).

پس کاری که باید بکنیم این است که postfix را طوری پیکربندی کنیم که از procmail استفاده کند؛ با تنظیم `mailbox_command = procmail` در فایل `main.cf`.

```
[root@localhost ~]# cd /etc/postfix/
[root@localhost postfix]# ls
access     generic        main.cf    relocated  virtual
canonical  header_checks  master.cf  transport  virtual.db
[root@localhost postfix]# vim main.cf 
[root@localhost postfix]# cat main.cf | grep mailbox_command
# The mailbox_command parameter specifies the optional external
# Unlike other Postfix configuration parameters, the mailbox_command
#mailbox_command = /some/where/procmail
#mailbox_command = /some/where/procmail -a "$EXTENSION"
# has precedence over the mailbox_command, fallback_transport and
mailbox_command = procmail

[root@localhost postfix]# systemctl restart postfix
```

حالا postfix هر چیزی را که به صورت local تحویل می‌دهد به procmail می‌فرستد، اما هنوز خود procmail پیکربندی نشده است.

دو لایه فایل پیکربندی برای procmail وجود دارد:

1. پیکربندی سراسری (Global): ‎`/etc/procmailrc`
2. پیکربندی مخصوص کاربر (Recipeها): ‎`~/.procmailrc`

وقتی Procmail شروع می‌شود، پیام ایمیل را می‌خواند و body را از header جدا می‌کند. سپس فایل `/etc/procmailrc` و فایل‌های داخل دایرکتوری `/etc/procmailrcs` را برای متغیرهای محیطی و recipeهای پیش‌فرض و سراسری بررسی می‌کند. بعد از آن، به دنبال فایل `.procmailrc` در home directory کاربر می‌گردد تا ruleهای مخصوص همان کاربر را پیدا کند. بسیاری از کاربران فایل‌های rc اضافی هم می‌سازند که توسط `.procmailrc` به آن‌ها ارجاع داده می‌شود و می‌توانند در صورت بروز مشکل در mail filtering سریعاً فعال/غیرفعال شوند.

در استفاده از فایل سراسری `/etc/procmailrc` مراقب باشید. این فایل معمولاً به عنوان root خوانده و پردازش می‌شود. این یعنی یک recipe بد طراحی‌شده در آن فایل می‌تواند آسیب جدی ایجاد کند. مثلاً یک typo می‌تواند باعث شود procmail به جای استفاده از یک binary مهم، آن را overwrite کند. به همین دلیل بهتر است پردازش سراسری procmail را به حداقل برسانید و به جای آن روی استفاده از `~/.procmailrc` برای پردازش ایمیلِ حساب‌های کاربری تمرکز کنید.

خب، از Global Configuration شروع کنیم. به صورت پیش‌فرض فایل global وجود ندارد و باید فایل `procmailrc` را زیر `/etc` بسازیم:

```
[root@localhost postfix]# cd /etc/
[root@localhost etc]# vim procmailrc
[root@localhost etc]# cat procmailrc 
### Define where we want to emails be stored
MAILDIR=$HOME/mail

### Defining mail storage Format

# For mbox format
DEFAULT=$HOME/mail/inbox

#For maildir format
#DEFAULT=$HOME/mail/inbox/
```

دقت کنید `/` آخر مسیر رفتار را تغییر می‌دهد! فعلاً با mbox جلو می‌رویم و آن را بررسی می‌کنیم:

```
[user1@localhost ~]$ ls
dead.letter
[user1@localhost ~]$ mkdir mail
[user1@localhost ~]$ ls mail/
```

```
[user2@localhost ~]$ mail -s "test mbox for user 1" user1@localhost
Hi this is my first message for testing mbox
.
EOT
[user2@localhost ~]$ mail -s "test mbox for user 1 #2" user1@localhost
Hi this is my second message for testing mbox 
.
EOT
```

```
[user1@localhost ~]$ ls -l mail/
total 4
-rw-------. 1 user1 user1 1327 May 23 03:36 inbox
[user1@localhost ~]$ cd mail
[user1@localhost mail]$ cat inbox 
From user2@centos7.example.com  Wed May 23 03:36:33 2018
Return-Path: <user2@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1002)
    id D0AC76128210; Wed, 23 May 2018 03:36:33 -0400 (EDT)
Date: Wed, 23 May 2018 03:36:33 -0400
To: user1@localhost.example.com
Subject: test mbox for user 1
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180523073633.D0AC76128210@centos7.example.com>
From: user2@centos7.example.com

Hi this is my first messege for testing mbox

From user2@centos7.example.com  Wed May 23 03:36:58 2018
Return-Path: <user2@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1002)
    id C9B0F6128210; Wed, 23 May 2018 03:36:58 -0400 (EDT)
Date: Wed, 23 May 2018 03:36:58 -0400
To: user1@localhost.example.com
Subject: test mbox for user 1 #2
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180523073658.C9B0F6128210@centos7.example.com>
From: user2@centos7.example.com

Hi this is my second message for testing mbox
```

ایمیل‌ها پشت سر هم به یک فایل واحد چسبیده‌اند. البته استفاده از ابزار `mail` راحت‌تر است، اما باید با گزینه `-f` پوشه/فایلی را که ایمیل‌ها داخل آن هستند مشخص کنیم:

```
[user1@localhost mail]$ mail -f ~/mail/inbox 
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/home/user1/mail/inbox": 2 messages 2 new
>N  1 user2@centos7.exampl  Wed May 23 03:36  18/661   "test mbox for user 1"
 N  2 user2@centos7.exampl  Wed May 23 03:36  18/666   "test mbox for user 1 #2"
&
```

### Recipeها

همان‌طور که گفتیم، Procmail اجازه می‌دهد ایمیل را هنگام دریافت از یک سرور ایمیل remote یا هنگام قرار گرفتن در spool file در یک سرور ایمیل local/remote فیلتر کنیم. این فیلتر کردن با استفاده از Recipeها (پیکربندی‌های مخصوص کاربر) انجام می‌شود. Recipeها با ساختن فایل `.procmailrc` در home directory کاربر پیکربندی می‌شوند.

سخت‌ترین بخش یادگیری procmail، ساختن recipeهاست، چون برای تطبیق پیام‌ها از regular expression استفاده می‌کنند. با این حال، ساختن regular expressionها آن‌قدر هم سخت نیست و حتی خواندنشان از ساختنشان هم ساده‌تر است؛ مشکل اصلی درک ساختار recipe در procmail است.

Recipeهای procmail به شکل زیر هستند:

```
:0 [flags] [: lockfile-name ]
* [ condition_1_special-condition-character condition_1_regular_expression ]
* [ condition_2_special-condition-character condition-2_regular_expression ]
* [ condition_N_special-condition-character condition-N_regular_expression ]
        special-action-character
        action-to-perform
```

دو کاراکتر اول در یک recipe، یک colon و یک صفر است. بعد از صفر می‌توان flagهای مختلفی گذاشت تا نحوه پردازش recipe توسط procmail کنترل شود. اگر بعد از بخش `flags` یک colon بیاید، یعنی برای این پیام یک lockfile ساخته می‌شود. اگر lockfile ساخته شود، می‌توان نامش را با جایگزین کردن `lockfile-name` مشخص کرد.

یک recipe می‌تواند چندین condition برای match شدن در برابر پیام داشته باشد. اگر condition نداشته باشد، همه پیام‌ها match می‌شوند. regular expressionها داخل conditionها قرار می‌گیرند تا match کردن پیام‌ها انجام شود. اگر چند condition وجود داشته باشد، همه آن‌ها باید match شوند تا action انجام شود. conditionها بر اساس flagهای تنظیم‌شده در خط اول بررسی می‌شوند. کاراکترهای خاص اختیاری بعد از `*` می‌توانند کنترل بیشتری روی condition داشته باشند.

آرگومان `action-to-perform` مشخص می‌کند وقتی پیام یکی از conditionها را match کرد چه کاری انجام شود. هر recipe فقط یک action می‌تواند داشته باشد. در بسیاری از موارد، نام یک mailbox اینجا قرار می‌گیرد تا پیام‌های match شده به آن فایل هدایت شوند و عملاً sort انجام شود. همچنین می‌توان قبل از action از special action character استفاده کرد.

فعلاً یک recipe ساده را می‌بینیم که ایمیل‌هایی با Subject شامل "lpi" را به فایل `linuxcert` منتقل می‌کند:

```
:0:
* ^Subject:.lpi
linuxcert
```

تست می‌کنیم:

```
[user1@localhost mail]$ pwd
/home/user1/mail

[user1@localhost mail]$ ls -l
total 4
-rw-------. 1 user1 user1 1347 May 26 02:25 inbox
[user1@localhost mail]$ cd
[user1@localhost ~]$ vim .procmailrc

[user1@localhost ~]$ mail -s "lpi" user1@localhost
EOT
Null message body; hope that's ok
[user1@localhost ~]$ ls -l mail/
total 8
-rw-------. 1 user1 user1 1347 May 26 02:25 inbox
-rw-------. 1 user1 user1  613 May 26 02:27 linuxcert

[user1@localhost ~]$ cat mail/linuxcert 
From user1@centos7.example.com  Sat May 26 02:27:57 2018
Return-Path: <user1@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1001)
    id 1515B612820A; Sat, 26 May 2018 02:27:57 -0400 (EDT)
Date: Sat, 26 May 2018 02:27:56 -0400
To: user1@localhost.example.com
Subject: lpi
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: quoted-printable
Message-Id: <20180526062757.1515B612820A@centos7.example.com>
From: user1@centos7.example.com

=
```

حالا یک Subject دیگر می‌فرستیم تا مطمئن شویم recipe درست کار می‌کند:

```
[user1@localhost ~]$ mail -s "Nothing" user1@localhost
nothing
EOT

[user1@localhost ~]$ cat mail/inbox 
From user2@centos7.example.com  Wed May 23 03:36:33 2018
Return-Path: <user2@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1002)
    id D0AC76128210; Wed, 23 May 2018 03:36:33 -0400 (EDT)
Date: Wed, 23 May 2018 03:36:33 -0400
To: user1@localhost.example.com
Subject: test mbox for user 1
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180523073633.D0AC76128210@centos7.example.com>
From: user2@centos7.example.com
Status: O

Hi this is my first messege for testing mbox

From user2@centos7.example.com  Wed May 23 03:36:58 2018
Return-Path: <user2@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1002)
    id C9B0F6128210; Wed, 23 May 2018 03:36:58 -0400 (EDT)
Date: Wed, 23 May 2018 03:36:58 -0400
To: user1@localhost.example.com
Subject: test mbox for user 1 #2
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180523073658.C9B0F6128210@centos7.example.com>
From: user2@centos7.example.com
Status: O

Hi this is my second message for testing mbox 

From user1@centos7.example.com  Sat May 26 02:29:23 2018
Return-Path: <user1@centos7.example.com>
X-Original-To: user1@localhost
Delivered-To: user1@localhost.example.com
Received: by centos7.example.com (Postfix, from userid 1001)
    id A0D5B612820A; Sat, 26 May 2018 02:29:23 -0400 (EDT)
Date: Sat, 26 May 2018 02:29:23 -0400
To: user1@localhost.example.com
Subject: Nothing
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180526062923.A0D5B612820A@centos7.example.com>
From: user1@centos7.example.com

nothing
```

حالا که فرمت mbox را دیدیم، فرمت maildir را هم می‌بینیم. فایل inbox را که توسط mbox ساخته شده حذف می‌کنیم:

```
[user1@localhost ~]$ ls mail/
inbox  linuxcert
[user1@localhost ~]$ rm mail/inbox mail/linuxcert
```

حالا فایل procmailrc سراسری را ویرایش می‌کنیم:

```
[root@localhost mail]# vim /etc/procmailrc 
[root@localhost mail]# cat /etc/procmailrc 
### Define where we want to emails be stored
MAILDIR=$HOME/mail

### Defining mail storage Format

# For mbox format
#DEFAULT=$HOME/mail/inbox

#For maildir format
DEFAULT=$HOME/mail/inbox/
```

حالا یک ایمیل به user1 می‌فرستیم:

```
[user2@localhost ~]$ mail -s "test maildir format" user1@localhost
Hi this a test message! .
.
EOT
```

و خروجی فرمت maildir را برای ذخیره‌سازی ایمیل می‌بینیم:

```
[user1@localhost ~]$ tree mail/
mail/
└── inbox
    ├── cur
    ├── new
    │   └── 1527398388.50350_0.localhost.localdomain
    └── tmp

4 directories, 1 file
[user1@localhost ~]$ ls mail/
inbox
[user1@localhost ~]$ ls mail/inbox/
cur  new  tmp
```

خبر خوب این است که ابزار `mail` با هر دو فرمت mbox و maildir کار می‌کند:

```
[user1@localhost ~]$ mail -f mail/inbox/
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"mail/inbox/": 1 message 1 new
>N  1 user2@centos7.exampl  Sun May 27 01:19  17/584   "test maildir format"
& q
"mail/inbox/" complete
```

خب، همین بود. اگر دوست دارید می‌توانید در اینترنت بیشتر درباره recipeها مطالعه کنید.
