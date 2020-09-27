# 基本指令（Essential Commands）

## Git
很多人能使用Git的命令进行操作，却对Git的基本原理不甚了解，甚至对于Git和SVN之间的区别也无法能够精准的说明。这样导致的后果就是，在解决问题时候缺乏信心，也没有理论基础。

理解Git的关键在于理解Git是如何存储数据，具体来说需要理解`.git`的文件目录中各个文件夹的作用，特别是`HEAD`,`refs`,`objects`中是如何存储数据的。

另外一点需要理解的是， Git的快照就是Commit，一次Commit其实就是当前Project的一个快照。
当使用`git cat-file -p COMMIT_SHA` 这样的命令来具体查看一个快照所包含的内容的话，那么就会更加了解这个快照的含义。 

```
.git/
├── HEAD
├── config
├── description
├── hooks
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       ├── heads
│       │   └── master
│       └── remotes
│           └── origin
│               └── HEAD
├── objects
└── refs
    ├── heads
    │   └── master
    ├── remotes
    │   └── origin
    │       └── HEAD
    └── tags
```

下面的几篇文章提供了非常简单明了的Git的原理的说明，非常适合入门。

- [Git 原理入门](http://www.ruanyifeng.com/blog/2018/10/git-internals.html)
- [这才是真正的Git——Git内部原理揭秘！](https://www.jiqizhixin.com/articles/2019-12-20)
- [图解Git](https://marklodato.github.io/visual-git-guide/index-zh-cn.html)

## diff, patch

diff和patch是基本上成双成对出现的。在很多项目中，`git diff`用来打包项目版本间的差异，而'patch'的目的就是把打包好的差异更新到另外一个需要同步的Git项目中去。

举个例子，比如你有一个外包Vendor，他们为你开发程序，打包好了一个Python的应用程序V1.0。你把这个程序V1.0Copy到了你的生产环境里面。
外包Vendor有自己的Git，你们公司也有自己的Git。当外包Vendor的Git把应用程序升级到V2.0以后，你们也需要把你们的git项目升级版本到V2.0。

怎么办？ 下面是一种解决方案。（注意：下面步骤中的指令仅是示意。）

- 第一步，在外包Vendor的Git项目中执行`git diff v1.0 v2.0 > patchfile`来打包差异。
- 第二步，在你的Git项目中执行`patch -p0 patchfile`来展开差异并更新你的Git。 

## sed

文字替换指令。

## ssh and pssh

ssh和pssh的区别在于pssh可以多台Machine一起ssh。