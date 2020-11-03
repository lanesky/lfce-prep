# 安装NFS服务器

## NFS的概念理解

关于NFS的基本概念和操作，我推荐阅读`鸟哥的Linux私房菜`的[第十三章、文件服务器之一：NFS 服务器](http://cn.linux.vbird.org/linux_server/0330nfs.php)一文。
基本上看这一篇就够了。

## 场景练习

### 场景描述

你需要安装在server上通过NFSv4共享出来一个目录`/nfs/share1`，要求如下：
1. 来自10.128.0.0/16的访问为只读，并且为insecure共享。
2. 来自test.org这一domain的访问可以读写，并为secure共享。如果客户端是root，那么在服务端显示为匿名访问。
3. 在服务端共享目录下，所有的新建文件的权限为664,所有新建目录的共享权限为775。


### 实现场景所需命令

1. Server端配置

安装rpcbind和nfs服务。

```
yum install -y nfs-utils
systemctl enable rpcbind
systemctl enable nfs
systemctl start rpcbind
systemctl start nfs
```

设置exports，之后重新启动nfs。
```
vim /etc/exports
/nfs/share1/    10.128.0.0/16(ro,insecure) *.test.org(rw,root_squash,secure)
systemctl restart nfs
```

设定访问权限。
```
umask 002
```

查看mount是否成功。
```
showmount -e localhost
tail /var/lib/nfs/etab
```

2. Client端配置

安装rpcbind服务。
```
yum install -y nfs-utils
systemctl enable rpcbind
systemctl start rpcbind
```

查看服务端mount是否成功。
```
showmount -e 10.128.0.22
```

mount到服务端。
```
mkdir /data
mount -t nfs 10.128.0.22:/nfs/share1 /data

```

查看是否mount成功。
```
mount
```

### TODO:视频演示

### 延伸阅读

关于文件权限，用户及组权限的设定等，推荐阅读`鸟哥的Linux私房菜`的以下章节。

- [第六章、Linux 的文件权限与目录配置](http://cn.linux.vbird.org/linux_basic/0210filepermission.php)
- [第十四章、Linux 账号管理与 ACL 权限配置](http://cn.linux.vbird.org/linux_basic/0410accountmanager_3.php)
- [第八章、Linux 磁盘与文件系统管理](http://cn.linux.vbird.org/linux_basic/0230filesystem.php)












