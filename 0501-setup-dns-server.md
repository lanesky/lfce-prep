# 安装DNS服务器

## 概要

安装DNS服务器一般使用bind。 关于DNS的基础知识以及bind的用法，我严重推荐阅读骏马金龙的博客，[第7章 DNS & bind从基础到深入](https://www.cnblogs.com/f-ck-need-u/p/7367503.html)。 该博客写的非常的清楚，基本上涵盖了所有关于DNS的基础知识。


## 练习1

我制作了以下视频以演示如何使用bind来安装一个dns服务器。

https://youtu.be/CXSp4wlVJBY


## 练习2

使用Bind View可以实现相同域名可以根据不同的客户端有不同的域名解析结果。比如说，有一个域名"dbserver.example.tt"。使用Bind View可以做到： 你从Client1和Client2有不同的解析结果。 我制作了下面这个视频来演示如何实现。

https://youtu.be/nScgwIfjrVc