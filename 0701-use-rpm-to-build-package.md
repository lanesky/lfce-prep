# 使用RPM打包

## 使用一个RPM来打一个hello word的包。

本节直接使用我制作的一个视频来说明如何打包。包括使用什么工具来打包，怎么来写SPEC，怎么使用mock来模拟安装。
视频所使用的内容来自[How to create a GNU Hello RPM package](https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package).


[![use rpm to build package](http://img.youtube.com/vi/wncPnBQn7I4/0.jpg)](http://www.youtube.com/watch?v=wncPnBQn7I4 "Awesome")


## 参考
RPM的功能实际上相当多的，RPM SPEC的各种语法，macro也是相当的繁多。具体的内容可以参考:
- [Maximum RPM](http://ftp.rpm.org/max-rpm/).
- `鸟哥的Linux私房菜` [第二十三章、软件安装： RPM, SRPM 与 YUM 功能](http://cn.linux.vbird.org/linux_basic/0520rpm_and_srpm.php)

## Cheat sheet

安装所需有的工具。
```
yuminstall @"Development Tools" mock rpmdevtools rpmlint
```

建立空目录。
```
rpmdev-setuptree
```

建立新的SPEC文件。
```
rpmdev-newspec hello
```

打包。
```
rpmbuild -ba 
```

使用列出已有Key Pair。
```
gpg --list-keys
```

使用GPG给包签名。
```
gpg --add-sign xxx.rpm
```
