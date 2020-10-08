# 包过滤，转发以及网络地址转换

关于用于包过滤，转发以及网络地址转换的工具，老一代的工具有iptables，近一点的有firewalld，更新的是nftables。
但是不管工具怎么变，都会围绕着表(table),链(chain)和规则(rule)三个方面来实现。

这些工具本身并不是防火墙，真正实施防火墙功能的是内核Netfilter。下图解释了Netfilter中包是如何被过滤的。

![](https://upload.wikimedia.org/wikipedia/commons/3/37/Netfilter-packet-flow.svg)


# 参考资料


- [netfilter位于内核的那一层](https://blog.csdn.net/MAOTIANWANG/article/details/17410069)
- [iptables 介绍](https://zh.wikipedia.org/wiki/Iptables)
- [iptables 详解](https://juejin.im/post/6844904057241337864)
- [Iptables与Firewalld防火墙](https://www.linuxprobe.com/chapter-08.html)
- [firewalld 与 iptables](https://www.jianshu.com/p/70f7efe3a227)
- [CentOS 7之FirewallD与iptables的区别](https://my.oschina.net/deanzhao/blog/3058904)
- [CentOS 7.x でのIPマスカレード設定](https://qiita.com/gurere/items/3584df990c98965e785b)
- [Linux ip_forward 数据包转发](https://www.jianshu.com/p/134eeae69281)
- [nftables 简明教程](https://www.hi-linux.com/posts/29206.html)
- [继iptables之后的新一代包过滤框架是nftables](https://blog.51cto.com/dog250/1583015)


## 练习

在基本理解了以上参考资料以后，请尝试完成以下练习。

1. 你有两台机器Server和Client。Server有两块网卡哦eth0, eth1。eth0同时具有公网和内网IP，eth1仅有内网IP。Client仅有一块网卡eth0绑定内网IP。你需要实现NAT，使得你可以直接从Client上网运行比如"ping google.com"或者"yum install httpd"这样的命令。

提示： 
- 有两种方法可以实现这个要求。一种方法是使用firewalld里面的zone概念。另外一种方法是在firewalld里面直接插入iptables rule。
- 不要忘记ipv4 forward的开关是否打开。

2. 你有一个用户"foo"，你不想让他访问一台机器的9000/tcp端口。如何实现？

3. 你想允许从某机器发过来的指向1234/tcp的新建连接，但是需要拒绝任何其他和这台机器相关的连接，如何实现？

提示： 考虑tcp连接有四种状态：New, related, established,invalid。

4. 你想将所有到达eth0的443/tcp的流量转向另外一台机器的9001端口。如何实现？

5. 你想拒绝某台机器访问你本机的Http服务，但是允许这台机器Ping你的本机。如何实现？

