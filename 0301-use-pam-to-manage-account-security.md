# 使用PAM加强账号安全

PAM(Pluggable Authentication Modules)是认证模块，在login，ssh,sudo等命令后面都起着认证的作用。

关于基本概念的理解，推荐阅读 - 鸟哥的Linux的私房菜，[第十四章、Linux 账号管理与 ACL 权限配置](http://cn.linux.vbird.org/linux_basic/0410accountmanager_5.php)。

## 练习一

使用`pam_tally2`模块实现用户密码输错5次就锁住20分钟不允许login。

提示：

1. 该PAM在`/etc/pam.d/login`中配置。
2. 安全模块在`lib64/security/`中。
3. `man pam_tally2`之后就可以找到如下一个样例.
```
auth     required       pam_tally2.so deny=4 even_deny_root unlock_time=1200`
```

## 练习二

任意用户ssh login后，系统都自动的赋予一个环境变量`WHRE_I_AM_FROM`, 其内容是`<用户名>@<远程主机>`， 如果取不到这个信息，就显示缺省值`nobody@localhost`。

实现方法：

查看`/etc/pam.d/postlogin`是否已经配置了`pam_env.so`。

```
vim /etc/pam.d/postlogin
```

如果没有配置，就加上如下一条。
```
session     required      pam_env.so
```

然后编辑以下文件。
```
vim /etc/security/pam_env.conf
```

以下一条如果是注释掉了，则反注释。
```
REMOTEHOST      DEFAULT=localhost OVERRIDE=@{PAM_RHOST}
```

然后加上下面这一条。
```
WHRE_I_AM_FROM          DEFAULT=nobody@localhost        OVERRIDE=@{PAM_RUSER}@${REMOTEHOST}
```

使用任意user测试login。然后使用以下指令查看是否有error。
```
tail /var/log/secure
```