# 使用PAM加强账号安全

PAM(Pluggable Authentication Modules)是认证模块，在login，ssh,sudo等命令后面都起着认证的作用。

关于基本概念的理解，推荐阅读 - 鸟哥的Linux的私房菜，[第十四章、Linux 账号管理与 ACL 权限配置](http://cn.linux.vbird.org/linux_basic/0410accountmanager_5.php)。

## 练习

使用`pam_tally2`模块实现用户密码输错5次就锁住20分钟不允许login。

提示：

1. 该PAM在`/etc/pam.d/login`中配置。
2. 安全模块在`lib64/security/`中。
3. `man pam_tally2`之后就可以找到如下一个样例.
```
auth     required       pam_tally2.so deny=4 even_deny_root unlock_time=1200`
```

