# Operation of Running Systems

### 常用指令
- top, vmstat, lsof, tcpdump, netstat, ss, htop, iotop, and iostat commands.
- iftop, nethogs, and iptraf

ss可以认为是netstat的一个升级版本。

### 如何更新Kernel版本

grub2-mkconfig 脚本用于生成 grub 的配置。
/etc/default/grub 在大多数情况下，这是唯一应该直接修改的文件。
GRUB_DEFAULT 定义启动菜单中的默认操作系统选项。可以是数字索引、菜单标题或者 "saved"。
grub2-mkconfig 脚本用于生成 grub 的配置。它综合使用 /etc/grub.d/* 和 /etc/default/grub 中的相关配置文件生成最终的 /boot/grub/grub.cfg - GRUB2 所使用的唯一配置文件。


- 下载Kernel可以使用如下命令。
```
yum update kernel -y
```
- 查看GRUB2（GRand Unified Bootloader version 2)配置。 
```
[cloud_user@d0cc44ac1f1c ~]$ ll /etc/grub2.cfg
lrwxrwxrwx. 1 root root 22 Aug  5 16:00 /etc/grub2.cfg -> ../boot/grub2/grub.cfg
```
- 查看GRUB的通用配置。

`/boot/grub2/grub.cfg` 是 GRUB2 所使用的唯一的直接的配置文件。
`/etc/default/grub`是GRUB的通用配置，`/boot/grub2/grub.cfg`可以由`grub2-mkconfig`生成，
生成时所参考的对象之一就是`/etc/default/grub`。

注意下面的例子中打印出来的`GRUB_DEFAULT`。定义启动菜单中的默认操作系统选项。可以是数字索引、菜单标题或者 "saved"。

```
[cloud_user@d0cc44ac1f1c ~]$ cat /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_SERIAL_COMMAND="serial --speed=115200"
GRUB_CMDLINE_LINUX="console=ttyS0,115200 console=tty0 vconsole.font=latarcyrheb-sun16 crashkernel=auto  vconsole.keymap=us"
GRUB_DISABLE_RECOVERY="true"
```

- 查看启动菜单中的默认操作系统选项中所定义的"saved"的详细。这里需要查看另外一个文件`/boot/grub2/grubenv`。
其中包含了Kernel的详细版本。

```
[cloud_user@d0cc44ac1f1c ~]$ sudo cat /boot/grub2/grubenv
# GRUB Environment Block
saved_entry=CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)
```
- 查看可以选择的Kernel镜像。
```
[cloud_user@d0cc44ac1f1c ~]$ sudo awk -F\' /^menuentry/{print\$2} /etc/grub2.cfg
CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1127.18.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1127.10.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1127.8.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1062.18.1.el7.x86_64) 7 (Core)
CentOS Linux, with Linux 0-rescue-f9afeb75a5a382dce8269887a67fbf58
```

- 使用`uname -a` 查看当前的OS的Kernel版本。
```
[cloud_user@d0cc44ac1f1c ~]$ uname -a
Linux d0cc44ac1f1c.mylabserver.com 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

- 更改启动菜单中的默认操作系统选项。
```
[cloud_user@d0cc44ac1f1c ~]$ sudo grub2-set-default 2
```

更改完成后，确认看启动菜单中的默认操作系统选项已经完成更改。
```
[cloud_user@d0cc44ac1f1c ~]$ sudo grep saved /boot/grub2/grubenv
saved_entry=2
```

- 使用`grub2-mkconfig`来生成`/boot/grub2/grub.cfg` ，这是 GRUB2 所使用的唯一的直接的配置文件。
```
[cloud_user@d0cc44ac1f1c ~]$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1127.19.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.19.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.18.2.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.18.2.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.10.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.10.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.8.2.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.8.2.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1062.18.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1062.18.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-f9afeb75a5a382dce8269887a67fbf58
Found initrd image: /boot/initramfs-0-rescue-f9afeb75a5a382dce8269887a67fbf58.img
done
```

注意：尽管如下所示`/boot/grub2/grub.cfg`的symlink是`/etc/grub2.cfg`，但是一定不要直接更新Symlink。
原因是，更新Symlink时，不同的Program可能会有不同的处理，如果Program处理不当可能会引起不必要的麻烦。
所以，更新时请更新其原文件。
```
[cloud_user@d0cc44ac1f1c ~]$ ll /etc/grub2.cfg
lrwxrwxrwx. 1 root root 22 Aug  5 16:00 /etc/grub2.cfg -> ../boot/grub2/grub.cfg
```

- 更新完成以后再启动。
```
[cloud_user@d0cc44ac1f1c ~]$ reboot
```

- 启动后再确认系统Kernel版本。
```
[cloud_user@d0cc44ac1f1c ~]$ uname -a
Linux d0cc44ac1f1c.mylabserver.com 3.10.0-1127.10.1.el7.x86_64 #1 SMP Wed Jun 3 14:28:03 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

- 如果启动不成功
  这时可以从启动菜单中选择旧的版本，回到原始设定。
  ![例子](images/0201-boot-menu.png "启动菜单例")

-  一些参考资料
  - https://wiki.gentoo.org/wiki/GRUB2/zh-cn
  - https://wiki.centos.org/HowTos/Grub2
  
## 如何使用工具进行自动化

最常用的工具就是Shell，Python，Ruby这样的Script语言。比如Yum就是Python写的，sosreport也是。

## 如何制定一个具有现实意义的Plan

这一部分虽然和技术看起来无关，但是就如何制定个人或组织的长期的技术成长计划来说却是重要的。有一个SMART原则可以用于衡量您的Plan是否合适。
- Is it specific?
- Is it Measurable?
- Is it Achieveable?
- Is it Result-focused?
- Is it Time-bound?


## 如何管理已有系统的变化？
使用版本配置工具以及各种配置工具。 比如Git，Terraform，Ansible等等。这一点就可以和"Infrastructure as Code"这个概念联系起来了。

## 如何选择配置管理工具(Configuraiton Management Tool)
现在流行的配置管理工具有不少，名气比较大的有Terraform，Ansible，Chef，Puppet等等。
不同的管理工具有不同的特性和使用要求，你可以根据你所在的Team的特点来进行选择。

- Terraform: 不需安装代理，具有等幂性（不会重复操作）。 
- Ansible: 不需安装代理。内置tag属性可以实现类似于pipeline的功能。需要SSH到Server。
- Chef: 需要在Node上安装代理。多少需要对Ruby知识。
- Puppet:需要在Node上安装代理。具有等幂性。多少需要对Ruby知识。

## 如何确保硬件的完整性(Integrity)和可用性(Availability)

完整性(Integrity)主要是涉及到物理安全(Physical Security)。比如在构建data center时候需要考虑以下几点：
- 构建地点是否有灾难的可能性。（估计不会有人在核电站附近搞一个数据中心。）
- 人员进出时是怎么管理门禁的。（胸章，指纹，还是人脸识别？）
- 有没有对温度，湿度的监控？  （A**一次东京zone的当机据说就是和温度监控出了问题有关系。）
- 机柜有没有倾斜？ （不是开玩笑，这是有可能导致硬件产生问题的，事实上这样的事情曾经发生过。）

硬件可用性(Availability)主要是指Firmware的更新。母版有Firmware，NiC有Firmware，SAN有Firmware。
保护Firmware的可用也是保障硬件可用性重要的一环。

### 如何更新OS上的软件包
CentOS上有yum，Debian（比如Ubuntu）上有apt。 
注意yum是"YellowdogUpdater,Modified"的缩写。Apt是"AdvancedPackagingTool"的缩写。

Yum和Apt的用法非常相似。下面是Yum的几个例子：
```
yum install bash
yum update 
yum install http://some.path/bash.rpm
```
### 如何进行事故（Incident）管理

第一个是进行RCA。在RCA讨论中，需要着眼于分析下面几项,
- 发生了什么事情？
- 怎么发生的？
- 为什么会发生？
- 如果补救以防止再次发生？

RCA之后很重要的一点就是从长远考虑以防止相同事故再次发生。

### 如何使用sar来生成系统报告
sar用于生成系统快照，对象包括:Processor, Memory, Disk, and Network。sadf用于从系统快照中生成csv，json等格式的报告。

Cron的配置可以参见。
```
[cloud_user@d0cc44ac1f1c ~]$ sudo cat /etc/cron.d/sysstat 
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A
```

Sar的快照的存放地址在：
```
ls /var/log/sa/
```

每秒取3次快照。
sa -u 1 3
```
[cloud_user@d0cc44ac1f1c ~]$ sar -u 1 3
Linux 3.10.0-1127.10.1.el7.x86_64 (d0cc44ac1f1c.mylabserver.com) 	10/01/2020 	_x86_64_	(2 CPU)

09:23:49 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
09:23:50 PM     all      0.00      0.00      0.00      0.00      0.50     99.50
09:23:51 PM     all      0.00      0.00      0.50      0.00      0.00     99.50
09:23:52 PM     all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.00      0.00      0.17      0.00      0.17     99.67
```

取关于内存的信息。
```
[cloud_user@d0cc44ac1f1c ~]$ sar -r 1 3
Linux 3.10.0-1127.10.1.el7.x86_64 (d0cc44ac1f1c.mylabserver.com) 	10/01/2020 	_x86_64_	(2 CPU)

09:26:41 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
09:26:42 PM    686188   1163024     62.89      1036    567280   3050096     77.29    658216    342076         4
09:26:43 PM    682424   1166788     63.10      1036    567280   3067532     77.73    661172    342076         4
09:26:44 PM    682560   1166652     63.09      1036    567284   3067532     77.73    661136    342080        16
Average:       683724   1165488     63.03      1036    567281   3061720     77.58    660175    342077         8
```
直接查看Log的信息。注意Log本身二进制存放。
```
[cloud_user@d0cc44ac1f1c ~]$ sar -f /var/log/sa/sa01
Linux 3.10.0-1127.10.1.el7.x86_64 (d0cc44ac1f1c.mylabserver.com) 	10/01/2020 	_x86_64_	(2 CPU)
```

也可以使用sadf生成report（格式可以是csv，json等）。

例子1 （由于sar刚刚安装，所以没有log生成。这也导致outupt为空。）
```
[cloud_user@d0cc44ac1f1c ~]$ sadf -s 05:00:01 -e 07:00:01 -dT /var/log/sa/sa01 -- -A
```
例子2 （取当前数据。）

```
[cloud_user@d0cc44ac1f1c ~]$ sadf  -dT  -- -A
# hostname;interval;timestamp;CPU;%usr;%nice;%sys;%iowait;%steal;%irq;%soft;%guest;%gnice;%idle
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;-1;0.09;0.00;0.08;0.00;0.03;0.00;0.00;0.00;0.00;99.79
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0;0.10;0.00;0.10;0.00;0.01;0.00;0.00;0.00;0.00;99.79
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;1;0.08;0.00;0.07;0.00;0.04;0.00;0.00;0.00;0.00;99.80
# hostname;interval;timestamp;proc/s;cswch/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.54;100.27
# hostname;interval;timestamp;pswpin/s;pswpout/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.00;0.00
# hostname;interval;timestamp;pgpgin/s;pgpgout/s;fault/s;majflt/s;pgfree/s;pgscank/s;pgscand/s;pgsteal/s;%vmeff
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.28;1.81;197.87;0.00;93.78;0.00;0.00;0.00;0.00
# hostname;interval;timestamp;tps;rtps;wtps;bread/s;bwrtn/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.36;0.03;0.33;0.56;3.61
# hostname;interval;timestamp;frmpg/s;bufpg/s;campg/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;-6.55;0.00;0.05
# hostname;interval;timestamp;kbmemfree;kbmemused;%memused;kbbuffers;kbcached;kbcommit;%commit;kbactive;kbinact;kbdirty
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;681924;1167288;63.12;1036;567416;3058460;77.50;660576;342068;4
# hostname;interval;timestamp;kbswpfree;kbswpused;%swpused;kbswpcad;%swpcad
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;2097148;0;0.00;0;0.00
# hostname;interval;timestamp;kbhugfree;kbhugused;%hugused
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0;0;0.00
# hostname;interval;timestamp;dentunusd;file-nr;inode-nr;pty-nr
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;46573;6464;26099;1
# hostname;interval;timestamp;runq-sz;plist-sz;ldavg-1;ldavg-5;ldavg-15;blocked
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;2;406;0.00;0.01;0.05;0
# hostname;interval;timestamp;TTY;rcvin/s;txmtin/s;framerr/s;prtyerr/s;brk/s;ovrun/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0;0.00;0;0.00;0;0.00;0;0.00;0;0.00;0;0.00
# hostname;interval;timestamp;DEV;tps;rd_sec/s;wr_sec/s;avgrq-sz;avgqu-sz;await;svctm;%util
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;dev259-0;0.36;0.56;3.61;11.45;0.00;2.12;1.99;0.07
# hostname;interval;timestamp;IFACE;rxpck/s;txpck/s;rxkB/s;txkB/s;rxcmp/s;txcmp/s;rxmcst/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;ens5;2.68;2.28;0.51;0.40;0.00;0.00;0.00
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;lo;0.00;0.00;0.00;0.00;0.00;0.00;0.00
# hostname;interval;timestamp;IFACE;rxerr/s;txerr/s;coll/s;rxdrop/s;txdrop/s;txcarr/s;rxfram/s;rxfifo/s;txfifo/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;ens5;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;lo;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00
# hostname;interval;timestamp;call/s;retrans/s;read/s;write/s;access/s;getatt/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.00;0.00;0.00;0.00;0.00;0.00
# hostname;interval;timestamp;scall/s;badcall/s;packet/s;udp/s;tcp/s;hit/s;miss/s;sread/s;swrite/s;saccess/s;sgetatt/s
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00;0.00
# hostname;interval;timestamp;totsck;tcpsck;udpsck;rawsck;ip-frag;tcp-tw
d0cc44ac1f1c.mylabserver.com;601;2020-10-01 21:30:01;750;13;11;0;0;0
```

## 安全审计

对于网络端口的开放以及是否有service正在监听端口，可以使用ss和nmap来查看。

ss使用例如下：

```
[root@d0cc44ac1f1c ~]# ss -t -o
State      Recv-Q Send-Q                                                           Local Address:Port                                                                            Peer Address:Port                
ESTAB      0      0                                                                  172.31.44.0:54122                                                                          52.94.210.188:https                 timer:(keepalive,6.472ms,0)
ESTAB      0      36                                                                 172.31.44.0:ssh                                                                          153.174.145.143:50979                 timer:(on,464ms,0)
ESTAB      0      0                                                                  172.31.44.0:60052                                                                         52.119.168.133:https                 timer:(keepalive,1.800ms,0)
```

nmap可以取得更多的信息。比如，如果apache是web服务器，nmap可以查看apache是否暴露了版本信息等。

```
[root@d0cc44ac1f1c ~]# nmap -A -sS localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2020-10-01 22:32 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000012s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 993 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 2048 57:80:47:d1:b4:97:49:6f:4c:87:2b:c1:9d:06:06:1b (RSA)
|_256 55:15:41:aa:f5:99:71:67:8a:ec:33:50:49:24:7c:97 (ECDSA)
25/tcp   open  smtp    Postfix smtpd
|_smtp-commands: d0cc44ac1f1c.mylabserver.com, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS))
| http-methods: Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_http-title: Apache HTTP Server Test Page powered by CentOS
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  3,4         2049/tcp  nfs
|   100003  3,4         2049/udp  nfs
|   100005  1,2,3      20048/tcp  mountd
|   100005  1,2,3      20048/udp  mountd
|   100021  1,3,4      37110/tcp  nlockmgr
|   100021  1,3,4      59934/udp  nlockmgr
|   100024  1          34442/udp  status
|   100024  1          51673/tcp  status
|   100227  3           2049/tcp  nfs_acl
|_  100227  3           2049/udp  nfs_acl
2049/tcp open  nfs     3-4 (RPC #100003)
5901/tcp open  vnc     VNC (protocol 3.8)
| vnc-info:   
|_  ERROR: Too many security failures
6001/tcp open  X11     (access denied)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.7 - 3.9
Network Distance: 0 hops
Service Info: Host:  d0cc44ac1f1c.mylabserver.com; OS: Unix

OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.06 seconds

```

对于`/var/log/audit/audit.log`的审核可以使用aureport工具，简单方便。

```
[root@d0cc44ac1f1c ~]# aureport --failed

Failed Summary Report
======================
Range of time in logs: 01/01/70 00:00:00.000 - 10/01/20 22:34:01.500
Selected time for report: 01/01/70 00:00:00 - 10/01/20 22:34:01.500
Number of changes in configuration: 0
Number of changes to accounts, groups, or roles: 3
Number of logins: 0
Number of failed logins: 595
Number of authentications: 0
Number of failed authentications: 1129
Number of users: 2
Number of terminals: 4
Number of host names: 66
Number of executables: 7
Number of commands: 4
Number of files: 11
Number of AVC's: 66
Number of MAC events: 0
Number of failed syscalls: 66
Number of anomaly events: 0
Number of responses to anomaly events: 0
Number of crypto events: 0
Number of integrity events: 0
Number of virt events: 0
Number of keys: 0
Number of process IDs: 601
Number of events: 1825
```

`auditd` 可以用于对于特定文件的更新的审计。[auditdでLinuxのファイル改竄検知を行う](https://qiita.com/minamijoyo/items/0d77959a869e458d850a) 是一个很好的post，可以参考。 



















