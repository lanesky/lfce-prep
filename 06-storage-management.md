# 存储管理

## 基本概念

关于存储管理，我推荐阅读`鸟哥的Linux私房菜`的以下章节。

- [第八章、Linux 磁盘与文件系统管理](http://cn.linux.vbird.org/linux_basic/0230filesystem.php)
- [第十五章、磁碟配额(Quota)与进阶文件系统管理](http://cn.linux.vbird.org/linux_basic/0420quota.php#lvm)

## 场景练习

### 内容

使用LVM练习如下内容:
1. 创建一个名为`vg1`的Volume Group，对象是 `/dev/xvdw5`, `/dev/xvdw6`, `/dev/xvdw7` 这几块磁盘，定义PE为5MB
2. 在`vg1`上创建一个镜像Volume `mirrorvol`，大小为64MB
3. 在`vg1`上创建一个Striped Volume `stripedvol`，大小为64MB, block size为64K
4. 继续把`stripedvol`的大小扩展64MB


### 使用命令

创建Volume Group。
```
vgcreate -s 5M vg1 /dev/xvdw5 /dev/xvdw6 /dev/xvdw7
```
创建镜像卷。
```
lvcreate -L 64M -m1 -n mirrorvol vg1
```
创建Striped Volume。
```
lvcreate -L 64M -i3 -I64  -n stripedvol vg1
```
扩展Striped Volume大小。
```
lvextend -L +64M /dev/vg1/stripedvol
```
