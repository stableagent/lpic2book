# 200.2. پیش‌بینی نیازهای منابع در آینده

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند مصرف منابع را مانیتور کنند تا نیازهای منابع در آینده را پیش‌بینی نمایند.

**نواحی کلیدی دانش:**

* استفاده از ابزارهای مانیتورینگ و اندازه‌گیری برای مانیتورینگ مصرف زیرساخت IT.
* پیش‌بینی نقطه شکست ظرفیت یک پیکربندی
* مشاهده نرخ رشد مصرف ظرفیت
* نمودار روندهای مصرف ظرفیت
* آگاهی از راه‌حل‌های مانیتورینگ مانند Icinga2, Nagios, collectd, MRTG و Cacti

**لیست زیر لیست جزئی از فایل‌ها، اصطلاحات و ابزارهای استفاده شده است:**

* تشخیص (diagnose)
* پیش‌بینی رشد (predict growth)
* تخلیه منابع (resource exhaustion)

پیش‌بینی آینده یکی از سخت‌ترین کارهاست، آیا "دوروتی و جادوگران از" را به یاد می‌آورید؟ :) بسیاری از مدیران سیستم آرزو دارند اگر چیزی مثل توپ شیشه‌ای داشتند. آن‌ها آرزو می‌کنند که بتوانند متوجه شوند چه نوع مشکلاتی ممکن است فردا رخ دهد تا امروز از آن جلوگیری کنند. اما متاسفانه این کار غیرممکن است. بنابراین آن را فراموش کنید و بیایید در مورد محیط واقعی صحبت کنیم. در علوم کامپیوتر، پیش‌بینی آینده فقط با بایگانی و پردازش آنچه در گذشته اتفاق افتاده است امکان‌پذیر است. علوم ریاضی و آمار می‌توانند داده‌های قدیمی را تحلیل کرده و روندها را نمودار کنند تا آینده را پیش‌بینی کنند.

![](.gitbook/assets/tuxpi.com.1511961969.jpg)

چندین ابزار برای کمک به ما وجود دارد:

* Icinga2
* Nagios
* collectd
* MRTG
* Cacti

ما می‌توانیم یک دوره کامل را برای هر یک از این ابزارها صرف کنیم، اما برای آزمون LPI باید از وجود آن‌ها آگاه باشیم. در ابتدا ما زمانی را روی collectd صرف می‌کنیم تا حداقل یک راه‌حل را نشان دهیم و ببینیم چگونه به نظر می‌رسد. و سپس ما فقط سایر راه‌حل‌ها را معرفی می‌کنیم.

## collectd

Collectd، یک daemon در یونیکس است که آمارهای مرتبط با عملکرد سیستم را جمع‌آوری می‌کند و همچنین امکاناتی برای ذخیره مقادیر در فرمت‌های مختلف مانند فایل‌های RRD (پایگاه داده Round Robin) را فراهم می‌کند. آمارهای جمع‌آوری شده توسط Collectd کمک می‌کنند تا موانع فعلی عملکرد را شناسایی کرده و بار سیستم را در آینده پیش‌بینی کنند.

collectd به طور پیش‌فرض در بسیاری از توزیع‌ها نصب شده است، با این حال می‌تواند به راحتی از مخازن در سیستم‌های redhat یا debian نصب شود، در اینجا ما از سیستم CentOS7 استفاده می‌کنیم:

```
[root@centos7-2 ~]# yum install collectd
```

برای پیکربندی آنچه مانیتور شود، فایل collect.conf را تغییر دهید:

```
[root@centos7-2 ~]# rpm -ql collectd | grep etc
/etc/collectd.conf
/etc/collectd.d

[root@centos7-2 etc]# ls -l  | grep collectd
-rw-r--r--.  1 root root     42077 May 17 04:46 collectd.conf
drwxr-xr-x.  2 root root         6 May 17 04:46 collectd.d

[root@centos7-2 etc]# wc -l collectd.conf
1820 collectd.conf
```

و آن یک فایل پیکربندی عظیم است که شامل بخش‌های مختلف است. که بخشی از آزمون ما نیستند! (فایل خلاصه شده است):

```
##############################################################################
# Global                                                                     #
#----------------------------------------------------------------------------#
# Global settings for the daemon.                                            #
##############################################################################

#Hostname    "localhost"
#FQDNLookup   true
#BaseDir     "/var/lib/collectd"
#PIDFile     "/var/run/collectd.pid"
#PluginDir   "/usr/lib64/collectd"
#TypesDB     "/usr/share/collectd/types.db"

#----------------------------------------------------------------------------#
# When enabled, plugins are loaded automatically with default options    #
# when an appropriate <Plugin ...> block is encountered.                     #
# Disabled by default.                                                       #
#----------------------------------------------------------------------------#
#AutoLoadPlugin false

#----------------------------------------------------------------------------#
# When enabled, internal statistics are collected, using "collectd" as the   #
# plugin name.                                                               #
# Disabled by default.                                                       #
#----------------------------------------------------------------------------#
#CollectInternalStats false

#----------------------------------------------------------------------------#
# Interval at which to query values. This may be overwritten on a per-plugin #
# base by using the 'Interval' option of the LoadPlugin block:               #
#   <LoadPlugin foo>                                                         #
#       Interval 60                                                          #
#   </LoadPlugin>                                                            #
#----------------------------------------------------------------------------#
#Interval     10

#MaxReadInterval 86400
#Timeout         2
#ReadThreads     5
#WriteThreads    5

# Limit size of the write queue. Default is no limit. Setting up a limit is
# recommended for servers handling a high volume of traffic.
#WriteQueueLimitHigh 1000000
#WriteQueueLimitLow   800000

##############################################################################
# Logging                                                                    #
#----------------------------------------------------------------------------#
# Plugins which provide logging functions should be loaded first, so log     #
# messages generated when loading or configuring other plugins can be        #
# accessed.                                                                  #
##############################################################################

LoadPlugin syslog
#LoadPlugin logfile
#LoadPlugin log_logstash

#<Plugin logfile>
#    LogLevel info
#    File STDOUT
#    Timestamp true
#    PrintSeverity false
#</Plugin>

#<Plugin log_logstash>
#    LogLevel info
#    File "/var/log/collectd.json.log"
#</Plugin>

#<Plugin syslog>
#    LogLevel info
#</Plugin>

##############################################################################
# LoadPlugin section                                                         #
#----------------------------------------------------------------------------#
# Lines beginning with a single `#' belong to plugins which have been built  #
# but are disabled by default.                                               #
#                                                                            #
# Lines beginning with `##' belong to plugins which have not been built due  #
# to missing dependencies or because they have been deactivated explicitly.  #
##############################################################################
.
..
....

LoadPlugin cpu
#LoadPlugin cpufreq
#LoadPlugin cpusleep

#LoadPlugin df
#LoadPlugin disk
#LoadPlugin dns

LoadPlugin interface
#LoadPlugin ipc
#LoadPlugin ipmi
#LoadPlugin iptables
#LoadPlugin ipvs
#LoadPlugin irq
#LoadPlugin java
LoadPlugin load
##LoadPlugin lpar
#LoadPlugin lua
#LoadPlugin lvm
#LoadPlugin madwifi
#LoadPlugin mbmon
#LoadPlugin mcelog
#LoadPlugin md
#LoadPlugin memcachec
#LoadPlugin memcached
LoadPlugin memory
##LoadPlugin mic
#LoadPlugin modbus
#LoadPlugin mqtt
#LoadPlugin multimeter
#LoadPlugin mysql
##LoadPlugin netapp
#LoadPlugin netlink
LoadPlugin network
#LoadPlugin nfs

#LoadPlugin rrdtool
......
....
..
.
##############################################################################
# Plugin configuration                                                       #
#----------------------------------------------------------------------------#
# In this section configuration stubs for each plugin are provided. A desc-  #
# ription of those options is available in the collectd.conf(5) manual page. #
##############################################################################
.
..
...
#<Plugin apache>
#  <Instance "local">
#    URL "http://localhost/status?auto"
#    User "www-user"
#    Password "secret"
#    CACert "/etc/ssl/ca.crt"
#  </Instance>
#</Plugin>

#<Plugin rrdtool>
#       DataDir "/var/lib/collectd/rrd"
#       CreateFilesAsync false
#       CacheTimeout 120
#       CacheFlush   900
#       WritesPerSecond 50
#</Plugin>

...
..
##############################################################################
# Filter configuration                                                       #
#----------------------------------------------------------------------------#
# The following configures collectd's filtering mechanism. Before changing   #
# anything in this section, please read `FILTER CONFIGURATION' section   #
# in the collectd.conf(5) manual page.                                       #
##############################################################################

# Load required matches:
#LoadPlugin match_empty_counter
#LoadPlugin match_hashed
#LoadPlugin match_regex
#LoadPlugin match_value
#LoadPlugin match_timediff
..
...
..
#############################################################################
# Threshold configuration                                                    #
#----------------------------------------------------------------------------#
# The following outlines how to configure collectd's threshold checking      #
# plugin. The plugin and possible configuration options are documented in    #
# collectd-threshold(5) manual page.                                     #
##############################################################################

#LoadPlugin "threshold"
#<Plugin threshold>
#  <Type "foo">
#    WarningMin    0.00
#    WarningMax 1000.00
#    FailureMin    0.00
#    FailureMax 1200.00
#    Invert false
#    Instance "bar"
#  </Type>
#
#  <Plugin "interface">
#    Instance "eth0"
#    <Type "if_octets">
#      FailureMax 10000000
#      DataSource "rx"
#    </Type>
#  </Plugin>
#
#  <Host "hostname">
#    <Type "cpu">
#      Instance "idle"
#      FailureMin 10
#    </Type>
...
..
```

برای مانیتور کردن مورد خاص، می‌توانیم آن را با حذف علامت توضیح در بخش LoadPlugin و پیکربندی plugin فعال کنیم. کاری که ما در اینجا انجام داده‌ایم فعال کردن ماژول network است.

بیایید سرویس را ریستارت کنیم:

```
[root@centos7-2 etc]# systemctl restart collectd.service
```

حالا ما باید به collectd زمان کافی بدهیم تا اطلاعات در مورد شبکه و سایر ماژول‌هایی که فعال شده‌اند را جمع‌آوری کند. متاسفانه collectd به طور پیش‌فرض با یک رابط برای مشاهده داده‌های جمع‌آوری شده همراه نمی‌شود، بنابراین ما نیاز به برخی کارهای اضافی داریم تا یک رابط وب برای آن نصب کنیم.

### پیکربندی Collectd-web برای مانیتورینگ یک سرور

**Collectd-web** یک ابزار مانیتورینگ فرانت‌اند وب است که بر اساس RRDtool (ابزار پایگاه داده Round-Robin) ساخته شده است، که داده‌های جمع‌آوری شده توسط سرویس Collectd روی سیستم‌های لینوکس را تفسیر کرده و به صورت گرافیکی خروجی می‌دهد.

بنابراین اول بیایید rrdtool module را نصب و فعال کنیم:

```
[root@centos7-2 ~]# yum install  collectd-rrdtool.x86_64 rrdtool rrdtool-devel rrdtool-perl perl-HTML-Parser perl-JSON perl-CGI
```

و این بخش‌های فایل collectd.conf را تغییر دهید:

```
.
..
...
LoadPlugin rrdtool
..
..

<Plugin rrdtool>
        DataDir "/var/lib/collectd/rrd"
#       CreateFilesAsync false
#       CacheTimeout 120
#       CacheFlush   900
#       WritesPerSecond 50
</Plugin>
...
..
.
```

نصب git "

```
[root@centos7-2 ~]# yum install git
```

و کلون کردن collectd-web از مخزن آن:

```
[root@centos7-2 opt]# ls -l | grep collectd
drwxr-xr-x. 7 root root 251 Oct 15 19:29 collectd-web
[root@centos7-2 opt]# cd collectd-web/
[root@centos7-2 collectd-web]# ls
AUTHORS  CHANGELOG      COPYING  index.html  media       runserver.py
cgi-bin  check_deps.sh  docs     iphone      README.rst
```

ما از اسکریپت‌های **Collectd-web CGI** استفاده می‌کنیم که داده‌ها را تفسیر کرده و صفحه آمار گرافیکی HTML تولید می‌کند. اول ما باید آن را برای استفاده بیشتر اجرایی کنیم:

```
[root@centos7-2 collectd-web]# chmod +x cgi-bin/graphdefs.cgi
```

به طور پیش‌فرض، collectd-web برای اجرا روی آدرس loopback 127.0.0.1 پیکربندی شده است. شما باید اسکریپت runserver.py را ویرایش کنید و 127.0.0.1 را به 0.0.0.0 تغییر دهید، تا روی تمام آدرس‌های IP رابط شبکه بایند شده و از یک ماشین از راه دور به رابط collectd-web دسترسی پیدا کنید.

```
[root@centos7-2 collectd-web]# vim runserver.py
[root@centos7-2 collectd-web]# cat runserver.py
#!/usr/bin/env python

import CGIHTTPServer
import BaseHTTPServer
from optparse import OptionParser

class Handler(CGIHTTPServer.CGIHTTPRequestHandler):
    cgi_directories = ["/cgi-bin"]

PORT = 8888

def main():
    parser = OptionParser()
    opts, args = parser.parse_args()
    if args:
        httpd = BaseHTTPServer.HTTPServer((args[0], int(args[1])), Handler)
        print "Collectd-web server running at http://%s:%s/" % (args[0], args[1])
    else:
        httpd = BaseHTTPServer.HTTPServer(("0.0.0.0", PORT), Handler)
        print "Collectd-web server running at http://%s:%s/" % ("127.0.0.1", PORT)
    httpd.serve_forever()

if __name__ == "__main__":
    main()
```

بعد ما نیاز داریم /etc/collectd directory و فایل collection.conf داخل آن را ایجاد کنیم، این مشخص می‌کند که collectd-web می‌تواند داده‌ها را از کجا بخواند:

```
[root@centos7-2 collectd-web]# cd /etc/
[root@centos7-2 etc]# mkdir collectd
[root@centos7-2 etc]# cd collectd
[root@centos7-2 collectd]# touch collection.conf
[root@centos7-2 collectd]# vim collection.conf
[root@centos7-2 collectd]# cat collection.conf
DataDir: "/var/lib/collectd/rrd/"
```

بعد از تغییر اسکریپت سرور Python، می‌توانیم سرور را با اجرای runserver.py شروع کنیم:

```
[root@centos7-2 collectd-web]# ./runserver.py &
```

اگر فایروال بالا و در حال اجرا باشد، پورت مورد نیاز برای رابط collect-web را اضافه کنید. در اینجا ما از CentOS استفاده می‌کنیم بنابراین باید با firewalld سروکار داشته باشیم:

```
[root@centos7-2 ~]# firewall-cmd --zone=public --permanent --add-port=8888/tcp
success
[root@centos7-2 ~]# firewall-cmd --reload
success
```

در نهایت، می‌توانیم به رابط collectd-web در URL [`http://our-server-ip:8888`](http://our-server-ip:8888) در مرورگر وب خود دسترسی پیدا کنیم:

![](.gitbook/assets/collectd-collectdweb.jpg)

چگونه کار می‌کند؟ Collectd از data sets استفاده می‌کند که در types.db تعریف شده‌اند:

```
[root@centos7-2 ~]# ls /usr/share/collectd/
types.db
[root@centos7-2 ~]# cat /usr/share/collectd/types.db | grep cpu
cpu                     value:DERIVE:0:U
cpu_affinity            value:GAUGE:0:1
cpufreq                 value:GAUGE:0:U
ps_cputime              user:DERIVE:0:U, syst:DERIVE:0:U
vcpu                    value:GAUGE:0:U
virt_cpu_total          value:DERIVE:0:U
virt_vcpu               value:DERIVE:0:U
```

برای جمع‌آوری داده‌ها در فرمت خاص (Round Robin DataBase) در دایرکتوری /var/lib/directory/rrd که قبلاً در collectd.conf فعال کرده بودیم:

```
[root@centos7-2 ~]# cat /etc/collectd.conf | grep rrd
#LoadPlugin rrdcached
LoadPlugin rrdtool
#<Plugin rrdcached>
#    DaemonAddress "unix:/tmp/rrdcached.sock"
#    DataDir "/var/lib/collectd/rrd"
<Plugin rrdtool>
#    DataDir "/var/lib/collectd/rrd"


  [root@centos7-2 ~]# tree /var/lib/collectd/rrd
/var/lib/collectd/rrd
└── centos7-2.example.com
    ├── cpu-0
    │   ├── cpu-idle.rrd
    │   ├── cpu-interrupt.rrd
    │   ├── cpu-nice.rrd
    │   ├── cpu-softirq.rrd
    │   ├── cpu-steal.rrd
    │   ├── cpu-system.rrd
    │   ├── cpu-user.rrd
    │   └── cpu-wait.rrd
    ├── cpu-1
    │   ├── cpu-idle.rrd
    │   ├── cpu-interrupt.rrd
    │   ├── cpu-nice.rrd
    │   ├── cpu-softirq.rrd
    │   ├── cpu-steal.rrd
    │   ├── cpu-system.rrd
    │   ├── cpu-user.rrd
    │   └── cpu-wait.rrd
    ├── cpu-2
    │   ├── cpu-idle.rrd
    │   ├── cpu-interrupt.rrd
    │   ├── cpu-nice.rrd
    │   ├── cpu-softirq.rrd
    │   ├── cpu-steal.rrd
    │   ├── cpu-system.rrd
    │   ├── cpu-user.rrd
    │   └── cpu-wait.rrd
    ├── cpu-3
    │   ├── cpu-idle.rrd
    │   ├── cpu-interrupt.rrd
    │   ├── cpu-nice.rrd
    │   ├── cpu-softirq.rrd
    │   ├── cpu-steal.rrd
    │   ├── cpu-system.rrd
    │   ├── cpu-user.rrd
    │   └── cpu-wait.rrd
    ├── interface-ens33
    │   ├── if_dropped.rrd
    │   ├── if_errors.rrd
    │   ├── if_octets.rrd
    │   └── if_packets.rrd
    ├── interface-lo
    │   ├── if_dropped.rrd
    │   ├── if_errors.rrd
    │   ├── if_octets.rrd
    │   └── if_packets.rrd
    ├── interface-virbr0
    │   ├── if_dropped.rrd
    │   ├── if_errors.rrd
    │   ├── if_octets.rrd
    │   └── if_packets.rrd
    ├── interface-virbr0-nic
    │   ├── if_dropped.rrd
    │   ├── if_errors.rrd
    │   ├── if_octets.rrd
    │   └── if_packets.rrd
    ├── load
    │   └── load.rrd
    └── memory
        ├── memory-buffered.rrd
        ├── memory-cached.rrd
        ├── memory-free.rrd
        ├── memory-slab_recl.rrd
        ├── memory-slab_unrecl.rrd
        └── memory-used.rrd

11 directories, 55 files
```

خوب فراموش نکنید که فرآیند نصب بخشی از آزمون LPIC نیست و بر اساس توزیع و نسخه شما ممکن است مراحل نصب متفاوت از آنچه ما در اینجا انجام داده‌ایم باشد.

