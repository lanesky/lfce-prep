# 使用LDAP进行用户管理  

https://cloud.tencent.com/developer/article/1026304
https://blog.51cto.com/11093860/2318657
https://blog.51cto.com/11093860/2161809
https://wiki.gentoo.org/wiki/Centralized_authentication_using_OpenLDAP/zh

## 安装LDAP Server

### 事先准备

事先准备如下3个文件。

- initial.ldif
```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=la,dc=local

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=la,dc=local

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}
```
- ous.ldif
```
dn: dc=la,dc=local
dc: la
objectClass: top
objectClass: domain

dn: cn=ldapadm ,dc=la,dc=local
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=People,dc=la,dc=local
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=la,dc=local
objectClass: organizationalUnit
ou: Group
```
- users.ldif
```
dn: uid=pinehead,ou=People,dc=la,dc=local
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: pinehead
uid: pinehead
uidNumber: 9999
gidNumber: 100
homeDirectory: /home/pinehead
loginShell: /bin/bash
gecos: pinehead [Lead Penguin (at) Linux Academy]
userPassword: {crypt}x
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7

dn: uid=tcox,ou=People,dc=la,dc=local
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: tcox
uid: tcox
uidNumber: 10000
gidNumber: 100
homeDirectory: /home/tcox
loginShell: /bin/bash
gecos: Terry Cox [Super Dude (at) Linux Academy]
userPassword: {crypt}x
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7

```
### 安装

安装所需部件。

```
yum -y install openldap compat-openldap openldap-clients openldap-servers nss-pam-ldapd
```
安装完后启动sladpd。
```
systemctl start slapd
systemctl enable slapd
```

打印初始密码的hash。
```
slappasswd -h {SSHA} -s password
```

将hash填写到`intial.ldif`的`olcRootPW {SSHA}<OUR_HASH>`中去。
```
vim initial.ldif
```
导入初始设定。
```
ldapmodify -Y external -H ldapi:/// -f initial.ldif
```
导入几个schema。
```
for i in cosine nis inetorgperson; do ldapadd -Y external -H ldapi:/// -f  /etc/openldap/schema/$i.ldif; done
```
导入事先设定的OU。
```
ldapadd -x -W -D "cn=ldapadm, dc=la,dc=local" -f ous.ldif
```
导入事先设定的User。
```
ldapadd -x -W -D "cn=ldapadm, dc=la,dc=local" -f users.ldif
```
启动客户端认证。
```
authconfig --enableldap --enableldapauth --ldapserver=localhost --ldapbasedn="dc=la,dc=local" --enablemkhomedir --update
```
启动name service local dameon。
```
systemctl restart nslcd
```

### 确认
使用下面命令确认确实可以切换到初始导入user `pinehead`。
```
id pinehead
su - pinehead
pwd pinehead
```

## 客户端安装方法1: (nslcd(nss-pam-ldapd)+nscd)

### 安装

安装所需部件。
```
yum install openldap-clients nss-pam-ldapd -y
```

设定认证方式以及远程LDAP server。
```
authconfig --enableldap --enableldapauth --ldapserver=172.31.44.207 --ldapbasedn="dc=la,dc=local" --enablemkhomedir --update
```

启动name service local dameon。
```
systemctl restart nslcd
```

### 确认

为测试用户`pinehead`设定新密码。
```
ldappasswd -s somepassword -W -D "cn=ldapadm, dc=la,dc=local" -x "uid=pinehead,ou=People,dc=la,dc=local"
```

确认可以login成功。
```
ssh pinehead@localhost
```


## 客户端安装方法2: (sssd+nscd) 

### 安装

安装所需部件。
```
yum install sssd openldap-clients nscd -y
```

设定sssd。

```
[sssd]
config_file_version = 2
services = nss, pam
domains = LDAP

[domain/LDAP]
id_provider = ldap
auth_provider = ldap

ldap_uri = ldap://172.31.44.207
ldap_search_base = dc=la,dc=local

ldap_id_use_start_tls = true
ldap_tls_reqcert = allow
ldap_tls_cacert = /etc/pki/tls/certs/ca-bundle.crt

```

启动服务。

```
systemctl start sssd
```

设定客户端认证方式。
```
authconfig --enablesssd --enablesssdauth --enablemkhomedir --update
```

### 确认安装成功

确认`nsswitch.conf` 已经挂上了sss。

```
# cat /etc/nsswitch.conf
#
# /etc/nsswitch.conf
#
# An example Name Service Switch config file. This file should be
# sorted with the most-used services at the beginning.
#
# The entry '[NOTFOUND=return]' means that the search for an
# entry should stop if the search in the previous entry turned
# up nothing. Note that if the search failed due to some other reason
# (like no NIS server responding) then the search continues with the
# next entry.
#
# Valid entries include:
#
#	nisplus			Use NIS+ (NIS version 3)
#	nis			Use NIS (NIS version 2), also called YP
#	dns			Use DNS (Domain Name Service)
#	files			Use the local files
#	db			Use the local database (.db) files
#	compat			Use NIS on compat mode
#	hesiod			Use Hesiod for user lookups
#	[NOTFOUND=return]	Stop searching if not found so far
#

# To use db, put the "db" in front of "files" for entries you want to be
# looked up first in the databases
#
# Example:
#passwd:    db files nisplus nis
#shadow:    db files nisplus nis
#group:     db files nisplus nis

passwd:     files sss
shadow:     files sss
group:      files sss
#initgroups: files

#hosts:     db files nisplus nis dns
hosts:      files dns myhostname

# Example - obey only what nisplus tells us...
#services:   nisplus [NOTFOUND=return] files
#networks:   nisplus [NOTFOUND=return] files
#protocols:  nisplus [NOTFOUND=return] files
#rpc:        nisplus [NOTFOUND=return] files
#ethers:     nisplus [NOTFOUND=return] files
#netmasks:   nisplus [NOTFOUND=return] files     

bootparams: nisplus [NOTFOUND=return] files

ethers:     files
netmasks:   files
networks:   files
protocols:  files
rpc:        files
services:   files sss

netgroup:   files sss

publickey:  nisplus

automount:  files
aliases:    files nisplus

```

用一个测试用户`pinehead` 确认可以取得ID。

```
id pinehead
uid=9999(pinehead) gid=100(users) groups=100(users)
```
使用不同方法继续确认。
```
getent passwd pinehead
pinehead:*:9999:100:pinehead [Lead Penguin (at) Linux Academy]:/home/pinehead:/bin/bash
```
使用ssh继续确认。
```
[root@ce9ce2f42a1c ~]# ssh pinehead@localhost
Password: 
Creating home directory for pinehead.
[pinehead@ce9ce2f42a1c ~]$ pwd
/home/pinehead
```

## 参考

- [RedHat OPENLDAP](https://access.redhat.com/documentation/en-Us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap)
- [RedHat CONFIGURING IDENTITY AND AUTHENTICATION PROVIDERS FOR SSSD](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/configuring_domains)
- [Name Service Switch](https://en.wikipedia.org/wiki/Name_Service_Switch)