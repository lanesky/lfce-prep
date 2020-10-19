# 配置邮件服务器

这一节，我们主要是来看如何使用postfix来配置邮件服务器。

## 所需工具

邮件服务器需要安装`postfix`(CentOS7已经预装),收邮件需要安装`mailx`, 发邮件需要安装`telnet`。 

## postfix配置命令

`postfix`配置时候最常用的命令时`postconf`。`postconf -n`是查看当前配置。`postconf -e`则是更改配置。

## 收发邮件常用命令

发邮件需要使用`telnet`，连接到邮件服务器之后，发邮件常用的指令包括如下：

```
helo/ehlo
mail from:
rcpt to:
data
.
quit
```

收邮件是使用`mail`指令就可以。注意`&n`代表看编号为n的邮件，`&dn`代表删除编号为n的邮件，而`q`代表推出。

## 练习

本节主要练习如何安装postfix邮件服务器以及验证通过。视频请看：https://youtu.be/s5JmsjGH1jQ

本视频中所使用的配置如下：

```
postconf -e "inet_interfaces = all"
postconf -e "mydestination = mail.example.tt, localhost.locahostdomain, localhost"
postconf -e "mynetworks = 10.128.0.0/16, 127.0.0.0/8"
```

配置完成后需要注意重启`postfix`服务。







