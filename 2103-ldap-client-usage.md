# 210.3. استفاده از کلاینت LDAP

**وزن:** 2

**توضیحات:** داوطلبان باید بتوانند کوئری‌ها و به‌روزرسانی‌ها را به یک سرور LDAP انجام دهند. همچنین شامل import و اضافه کردن آیتم‌ها، و همچنین اضافه کردن و مدیریت کاربران است.

**نواحی کلیدی دانش:**

* ابزارهای LDAP برای مدیریت داده و کوئری‌ها
* تغییر رمزهای عبور کاربر
* کوئری از دایرکتوری LDAP

**اصطلاحات و ابزارها:**

* ldapsearch
* ldappasswd
* ldapadd
* ldapdelete

این دوره درباره ابزارهای کلاینت LDAP و استفاده از آن‌ها است. واضح است که ما به یک سرور OpenLDAP در حال اجرا نیاز داریم تا کوئری‌ها انجام دهیم و تغییرات ایجاد کنیم.

## توصیه می‌کنیم قبل از شروع خواندن این دوره، دوره 210.4 را مطالعه کنید :-o

**توجه**: استفاده از ابزارهای کلاینت LDAP ممکن است بر اساس نسخه‌های سرور OpenLDAP متفاوت باشد. برای دریافت بهترین نتایج و پوشش اهداف آزمون LPIC-2، ما از OpenLDAP v2.3.x روی CentOS 5 در این دوره استفاده کرده‌ایم.

## ldapsearch

ldapsearch یک اتصال به سرور LDAP باز می‌کند، bind می‌کند و با استفاده از پارامترهای مشخص شده جستجو انجام می‌دهد. به طور پیش‌فرض ldapsearch کامپیوتر localhost را برای سرور LDAP کوئری می‌کند اما امکان اجرای آن از یک کلاینت راه دور با استفاده از گزینه -h و مشخص کردن سرور وجود دارد.

حالا که برخی پیکربندی‌های LDAP اضافه شده‌اند، بیایید نتایج را با استفاده از دستور ldapsearch بررسی کنیم:

```
[root@centos5-1 openldap]# ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingContexts
#

#
dn:
namingContexts: dc=example,dc=com

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

## ldapadd

دستور ldapadd یک ابزار اضافه کردن entry LDAP است. اینجا ما از دستور ldapadd استفاده می‌کنیم که تعاریف فایل ldif ما را می‌گیرد و آن را به پیکربندی ما اضافه می‌کند.

OU را برای ساختاردهی به سرور دایرکتوری ایجاد کنیم، دوباره ما به فایل ldif مورد نیاز نیاز داریم:

```
[root@centos5-1 openldap]# vi myou.ldif
[root@centos5-1 openldap]# cat myou.ldif 
dn: ou=managers,dc=example,dc=com
ou: managers
description : Managers in the company
objectclass: organizationalunit
```

و بیایید دستورات ldapadd را اجرا کنیم:

```
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f myou.ldif
Enter LDAP Password: 
adding new entry "ou=managers,dc=example,dc=com"
```

`-x` به معنای استفاده از احراز هویت ساده است، `-D` می‌گوید که حساب admin اینجا setup شده تا چیزهایی به پیکربندی ما اضافه کند، `-W` برای احراز هویت ساده درخواست می‌کند (این به جای مشخص کردن رمز عبور در خط فرمان استفاده می‌شود). `-f` فایل ldif را مشخص می‌کند.

از دستور slapcat برای دیدن نتایج استفاده کنید:

```
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,com.
structuralObjectClass: organization
entryUUID: 4c4b3fa6-3ee8-1038-85fd-a183bee33ca9
creatorsName: cn=ldapadm,dc=example,dc=com
modifiersName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828083000Z
modifyTimestamp: 20180828083000Z
entryCSN: 20180828083000Z#000000#00#000000

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
```

## اضافه کردن کاربران

حالا بیایید یک کاربر اضافه کنیم:

```
[root@centos5-1 openldap]# vi myuser.ldif
[root@centos5-1 openldap]# cat myuser.ldif 
dn: uid=john,ou=managers,dc=example,dc=com
uid: john
cn: John Doe
sn: Doe
mail: john@example.com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
userPassword: john123
```

```
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f myuser.ldif
Enter LDAP Password: 
adding new entry "uid=john,ou=managers,dc=example,dc=com"
```

## ldapmodify

برای تغییر یک entry موجود:

```
[root@centos5-1 openldap]# vi modifyuser.ldif
[root@centos5-1 openldap]# cat modifyuser.ldif 
dn: uid=john,ou=managers,dc=example,dc=com
changetype: modify
replace: mail
mail: john.doe@example.com
-
add: telephoneNumber
telephoneNumber: +1 555 1234
```

```
[root@centos5-1 openldap]# ldapmodify -x -D "cn=ldapadm,dc=example,dc=com" -W -f modifyuser.ldif
Enter LDAP Password: 
modifying entry "uid=john,ou=managers,dc=example,dc=com"
```

## ldapdelete

برای حذف یک entry:

```
[root@centos5-1 openldap]# ldapdelete -x -D "cn=ldapadm,dc=example,dc=com" -W "uid=john,ou=managers,dc=example,dc=com"
Enter LDAP Password: 
```

## ldappasswd

برای تغییر رمز عبور کاربر:

```
[root@centos5-1 openldap]# ldappasswd -x -D "cn=ldapadm,dc=example,dc=com" -W -S "uid=john,ou=managers,dc=example,dc=com"
Enter LDAP Password: 
New password: 
Re-enter new password: 
```

## جستجوهای پیچیده

ldapsearch می‌تواند جستجوهای پیچیده‌ای انجام دهد:

```
# جستجوی همه کاربران در ou=managers
ldapsearch -x -b "ou=managers,dc=example,dc=com" "(objectClass=person)"

# جستجوی کاربر با نام خاص
ldapsearch -x -b "dc=example,dc=com" "(uid=john)"

# جستجو با فیلتر پیچیده
ldapsearch -x -b "dc=example,dc=com" "(&(objectClass=person)(mail=*@example.com))"
```

## خلاصه

ابزارهای کلاینت LDAP برای مدیریت دایرکتوری LDAP ضروری هستند. با استفاده از این ابزارها می‌توانید:

* اطلاعات را از دایرکتوری جستجو کنید
* entries جدید اضافه کنید
* اطلاعات موجود را تغییر دهید
* entries را حذف کنید
* رمزهای عبور کاربران را مدیریت کنید

این ابزارها انعطاف‌پذیری زیادی برای مدیریت دایرکتوری فراهم می‌کنند و می‌توانند در اسکریپت‌ها و اتوماسیون استفاده شوند.