# 使用LDAP进行用户管理

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

## 安装LDAP Client 1: (nslcd(nss-pam-ldapd)+nscd)

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


## 安装LDAP Client 2: (sssd+nscd) ------ TODO

### 安装

```
yum install sssd sssd-client sssd-ldap openldap-clients
```


## 参考
- [OPENLDAP](https://access.redhat.com/documentation/en-Us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap)
