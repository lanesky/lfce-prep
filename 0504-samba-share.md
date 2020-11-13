# 使用Samba共享文件

## 基本概念

关于基本概念的理解，推荐阅读 - `鸟哥的Linux的私房菜` [第十六章、文件服务器之二： SAMBA 服务器](http://cn.linux.vbird.org/linux_server/0370samba.php)

## 练习一

创建一个Samba的共享名为`data`，指向server的`/opt/sharedata`路径。同时创建Samba用户`toto`, password为`tata`。该用户对`/opt/sharedata`有写权限，directory mask为`0700`, create mask为`0664`。


Server端安装samba。
```
yum install -y samba samba-client
```


修改`/etc/samba/smb.conf`文件, 修改完后可以使用`testparm`测试格式是否正确。

```
[data]
    name = data share
    path = /opt/sharedata
    create mask = 0700
    directory mask = 0664
    browseable = yes
    writable = no
    printable = no
    valid users = toto
    write list = +toto
```

在Linux中创建用户`toto`。
```
useradd toto
```

在smb中创建用户`toto`，并指定password为`tata`。
```
smbpasswd -a toto
```

启动`smb`服务。

```
systemctl start smb
systemctl enable smb
smbpasswd -a toto
```

尝试连接。
```
smbclient -L //localhost -U toto
```

如果是SELinux，需要开发共享。

```
getsebool -a | grep samba
setsebool -P samba_export_all_ro=1 samba_export_all_rw=1
semanage fcontext -at samba_share_t "/opt/sharedata(/.*)?"
restorecon /opt/sharedata/
ll -Zd /opt/sharedata/
```

客户端mount的方法如下。
```
mount //10.128.0.22/data /sharedata -t cifs -o username=toto,password=tata,uid=1004,gid=1005
```


