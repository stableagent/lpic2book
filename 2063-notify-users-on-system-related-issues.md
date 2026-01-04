# 206.3. اطلاع‌رسانی به کاربران درباره مسائل مرتبط با سیستم

### **206.3 اطلاع‌رسانی به کاربران درباره مسائل مرتبط با سیستم**

**وزن:** 1

**توضیحات:** داوطلبان باید بتوانند کاربران را درباره مشکلات فعلی مرتبط با سیستم مطلع کنند.

**نواحی کلیدی دانش:**

* خودکارسازی ارتباط با کاربران از طریق پیام‌های زمان ورود (logon messages)
* اطلاع دادن به کاربران فعال درباره نگهداری (maintenance) سیستم

**اصطلاحات و ابزارها:**

* /etc/issue
* /etc/issue.net
* /etc/motd
* wall
* /sbin/shutdown
* systemctl

این درس درباره روش‌هایی است که می‌توانیم با آن‌ها سایر کاربران را مطلع کنیم. ممکن است نیاز به نگهداری سخت‌افزاریِ غیر برنامه‌ریزی‌شده باشد، سیستم نیاز به reboot داشته باشد، هسته جدید کامپایل شده باشد و ...

در لینوکس، روش‌های فعال و غیرفعالی برای اطلاع‌رسانی به سایر کاربران وجود دارد. با توجه به این‌که ممکن است بعضی کاربران logout کرده باشند، موضوع کمی پیچیده‌تر می‌شود. با ساده‌ترین دستور شروع می‌کنیم.

### wall

دستور `wall` یک پیام (یا محتوای یک فایل، یا ورودی استاندارد) را روی ترمینالِ همه کاربرانی که در حال حاضر وارد سیستم هستند نمایش می‌دهد. (در این مثال از CentOS 7 استفاده می‌کنیم):

```
[root@server1 ~]# cat message.txt 
Hello! This is from message.txt!
[root@server1 ~]# cat message.txt | wall
[root@server1 ~]# 
Broadcast message from root@server1 (Tue Feb  6 02:17:35 2018):

Hello! This is from message.txt!
logout

root@server1 ~]$ echo " This is using echo" | wall

Broadcast message from root@server1 (Tue Feb  6 02:23:30 2018):

 This is using echo
```

و از دید یک کاربر دیگر، چیزی که می‌بیند:

```
[user1@server1 ~]$ whoami
user1
[user1@server1 ~]$ 
Broadcast message from root@server1 (Tue Feb  6 02:17:35 2018):

Hello! This is from message.txt!

Broadcast message from root@server1 (Tue Feb  6 02:23:30 2018):

 This is using echo
```

اما کاربرانی که از طریق SSH به سرور ما وصل می‌شوند چه؟

#### پیام banner در زمان ورود (SSH login banner)

یکی از راه‌های محافظت و ایمن‌سازی loginهای SSH این است که یک پیام هشدار به کاربران غیرمجاز نمایش بدهیم یا یک پیام خوش‌آمدگویی/اطلاعاتی به کاربران مجاز نشان دهیم.

```
                                                                 #####
                                                                #######
                   @                                            ##O#O##
  ######          @@#                                           #VVVVV#
    ##             #                                          ##  VVV  ##
    ##         @@@   ### ####   ###    ###  ##### ######     #          ##
    ##        @  @#   ###    ##  ##     ##    ###  ##       #            ##
    ##       @   @#   ##     ##  ##     ##      ###         #            ###
    ##          @@#   ##     ##  ##     ##      ###        QQ#           ##Q
    ##       # @@#    ##     ##  ##     ##     ## ##     QQQQQQ#       #QQQQQQ
    ##      ## @@# #  ##     ##  ###   ###    ##   ##    QQQQQQQ#     #QQQQQQQ
  ############  ###  ####   ####   #### ### ##### ######   QQQQQ#######QQQQQ
```

به عنوان یک system administrator، تنظیم کردن security banner برای loginهای SSH یک عادت خوب است. این banner می‌تواند شامل هشدارهای امنیتی یا اطلاعات عمومی باشد.

```
###############################################################
#                 Authorized access only!                     # 
# Disconnect IMMEDIATELY if you are not an authorized user!!! #
#         All actions Will be monitored and recorded          #
###############################################################
```

دو روش رایج برای نمایش این پیام‌ها وجود دارد: یکی با استفاده از فایل `issue.net` و دیگری با استفاده از فایل `motd`.

* `issue.net`: نمایش banner قبل از نمایش prompt رمز عبور
* `motd`: نمایش banner بعد از این‌که کاربر وارد سیستم شد

### نمایش پیام هشدار SSH قبل از login (‎/etc/issue.net)

برای نمایش پیام خوش‌آمدگویی یا هشدار برای کاربران SSH قبل از login، از فایل `issue.net` استفاده می‌کنیم:

```
[root@server1 ~]# find /etc/ -name issue.net
/etc/issue.net
[root@server1 ~]# cat /etc/issue.net 
\S
Kernel \r on an \m

[root@server1 ~]# echo "This is from /etc/issue.net" >> /etc/issue.net 
[root@server1 ~]# cat /etc/issue.net 
\S
Kernel \r on an \m
This is from /etc/issue.net
```

حالا در فایل `/etc/ssh/sshd_config` باید مقدار `Banner` را به مسیر فایل banner تنظیم کنیم، مثل این:

```
Banner /etc/issue.net
```

مرحله آخر این است که daemon مربوط به SSH را restart کنیم تا تغییرات اعمال شود (بسته به توزیع و init system شما: SysV، upstart یا systemd):

```
[root@server1 ~]# systemctl restart sshd.service
```

برای دیدن نتیجه، از server2 به server1 وصل می‌شویم:

```
root@server2:~# ssh root@192.168.10.132
The authenticity of host '192.168.10.132 (192.168.10.132)' can't be established.
ECDSA key fingerprint is SHA256:QtfM2iXh5pxZeFdAUXEBEnRXNSP40MWIhnSYvpOBMoY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.132' (ECDSA) to the list of known hosts.
\S
Kernel \r on an \m
This is from /etc/issue.net
root@192.168.10.132's password:
```

### نمایش پیام هشدار SSH بعد از login (‎/etc/motd)

برای نمایش پیام banner بعد از login کاربر، از فایل `motd` استفاده می‌کنیم:

```
[root@server1 ~]# find /etc/ -name motd
/etc/motd
[root@server1 ~]# cat /etc/motd 
[root@server1 ~]# echo "This is from /etc/motd" > /etc/motd 
[root@server1 ~]# cat /etc/motd 
This is from /etc/motd
```

دوباره از server2 به server1 وصل می‌شویم:

```
root@192.168.10.132's password: 
Connection to 192.168.10.132 closed by remote host.
Connection to 192.168.10.132 closed.
root@server2:~# ssh root@192.168.10.132
\S
Kernel \r on an \m
This is from /etc/issue.net
root@192.168.10.132's password: 
Last login: Tue Feb  6 06:16:00 2018
This is from /etc/motd
```

و تمام.

### shutdown

دستور `shutdown` یک زمان برای خاموش شدن سیستم برنامه‌ریزی می‌کند. می‌توان از آن برای halt، power-off یا reboot کردن ماشین استفاده کرد.

| نمونه‌های دستور shutdown | توضیحات |
| ------------------------ | ------- |
| shutdown                 |         |
| shutdown now             |         |
| shutdown 10:10           | استفاده از “hh:mm” برای ساعت/دقیقه |
| shutdown -p now          | poweroff کردن ماشین |
| shutdown -H now          | halt کردن ماشین |
| shutdown -r10:10         | reboot کردن ماشین در ساعت 10:10AM |
| shutdown -c              | لغو کردن shutdown برنامه‌ریزی‌شده |

همچنین می‌توان هنگام استفاده از `shutdown` پیام هم ارسال کرد:

```
root@server1 etc]# shutdown +15 "we goes down after 15 min"
Shutdown scheduled for Tue 2018-02-06 07:52:33 EST, use 'shutdown -c' to cancel.
[root@server1 etc]# 
Broadcast message from root@server1 (Tue 2018-02-06 07:37:33 EST):

we goes down after 15 min
The system is going down for power-off at Tue 2018-02-06 07:52:33 EST!
```

و کاربران دیگر این را می‌بینند:

```
Broadcast message from root@server1 (Tue 2018-02-06 07:37:33 EST):

we goes down after 15 min
The system is going down for power-off at Tue 2018-02-06 07:52:33 EST!
```

همین.

## تبریک! LPIC-2 201 تمام شد.
