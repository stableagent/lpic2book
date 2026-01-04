# 210.4. پیکربندی یک سرور OpenLDAP

## **210.4 پیکربندی یک سرور OpenLDAP**

**وزن:** 4

**توضیحات:** داوطلبان باید بتوانند OpenLDAP را نصب و پیکربندی کنند و یک directory ساده بسازند. این هدف شامل مدیریت داده با LDIF، تنظیم access control و آگاهی از TLS است.

**نواحی کلیدی دانش:**

* مفاهیم پایه LDAP (DIT, DN, RDN, objectClass)
* فایل‌ها و ابزارهای OpenLDAP (slapd، ldapadd، ldapsearch و ...)
* مدیریت داده با LDIF
* تنظیم ACL و آگاهی از TLS

**اصطلاحات و ابزارها:**

* slapd
* slapd.conf یا cn=config (بسته به نسخه)
* ldapadd, ldapmodify, ldapdelete
* ldapsearch
* ldappasswd
* /etc/openldap/ یا /etc/ldap/

## LDAP چیست؟

LDAP (Lightweight Directory Access Protocol) یک پروتکل برای دسترسی به سرویس directory است. directory معمولاً برای نگهداری اطلاعاتی مثل کاربران، گروه‌ها، آدرس‌ها و سیاست‌ها استفاده می‌شود.

مفاهیم مهم:

* **DN**: Distinguished Name (مثلاً: `uid=user1,ou=People,dc=example,dc=com`)
* **DIT**: ساختار درختی داده‌ها
* **objectClass**: تعیین نوع entry و attributeهای مجاز

## نصب OpenLDAP (نمونه)

* Debian/Ubuntu:

```bash
apt update
apt install slapd ldap-utils
```

* RHEL/CentOS:

```bash
yum install openldap-servers openldap-clients
systemctl enable --now slapd
```

## ایجاد base DN و ساختار اولیه (با LDIF)

معمولاً ابتدا base DN را تعریف می‌کنیم (dc=example,dc=com) و سپس OUها را می‌سازیم.

نمونه LDIF:

```ldif
dn: dc=example,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: Example

dc: example


dn: ou=People,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: Groups
```

و اضافه کردن:

```bash
ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f base.ldif
```

## اضافه کردن یک کاربر (نمونه)

```ldif
dn: uid=user1,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
cn: User One
sn: One
uid: user1
mail: user1@example.com
userPassword: {SSHA}...
```

## جستجو (ldapsearch)

```bash
ldapsearch -x -b "dc=example,dc=com" "(uid=user1)"
```

## نکات امنیتی

* دسترسی admin را محدود کنید و ACLها را با دقت تنظیم کنید.
* برای ارتباط امن از TLS/SSL استفاده کنید (StartTLS یا LDAPS).
* از anonymous bind در صورت عدم نیاز جلوگیری کنید.
* برای production حتماً backup و monitoring در نظر بگیرید.
