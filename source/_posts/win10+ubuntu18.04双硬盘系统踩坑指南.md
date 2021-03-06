---
layout: post
title: "win10+ubuntu18.04双硬盘系统踩坑指南"
date: 2019-4-22 13:53
comments: true
tags: 
	- 技术
---

因为学习需要，打算在家里的台式机原有的win10系统上再安装一个ubuntu系统，于是马上下单了一块固态，本以为安装双系统应该分分钟的事，可是事实证明没这么简单。

### 1.加装固态硬盘
以前刚工作时比较拮据，配电脑的时候就装了一个250g固态，虽然日常需求已经可以满足了，但是如果要在原来已经分区的情况下再去压缩几十g给ubuntu,那就远远不够了，毕竟电脑上还是会储存一些私人的资料，总不能都格式化或者迁移，于是就有了加装一个固态硬盘的想法
```
准备材料：
1.SATA3接口固态硬盘
2.SATA3硬盘数据线(买固态一般是不赠送数据线，所以这个要自己单独买，网上顺道买就是，
千万不要像我一样跑去实体店里，结果给了一g跟烂线，耽误了好些时间)
3.硬盘电源线(一般我们配置的主机都会留有多余的电源线接口，在机箱里找找就行)
其它注意事项：有些教程上说要开启ACH模式，4k对齐，我这里主板默认就是开启的，这步略过
```
### 2.制作U盘启动盘
```
准备工具：
1.refus (https://rufus.ie/)
2.ubuntu iso(https://www.ubuntu.com/download/desktop)
制作：
1.安装refus,一直点next.
2.选择U盘(最好是低速盘)，ubuntu.iso镜像选择
3.分区类型选择(可以右键计算机->管理->磁盘管理->属性->卷查看自己磁盘是GPT还是MBR.我这里选择MBR)
4.上面配置完成，点击开始等待即可

其它注意事项：
1.如果磁盘是MBR类型，则我们需要使用传统BIOS引导，而不是UEFI，不然安装过程中会出错或者无法启动
(refus这里选择MBR类型会自动生成传统BIOS启动和UEFI启动两种方式,到时u盘启动会有两个启动项，
我们直接选择哪个没有UEFI标识的项目启动安装就行)
```
参考链接：

1.[refus制作启动盘](https://blog.csdn.net/ifreewolf_csdn/article/details/81330921)
2.[UEFI和BIOS](https://www.cnblogs.com/phyking/p/4456603.html)
### 3.硬盘准备
如果是新装的硬盘，一般不需要格式化等操作，但是如果我们第一次安装失败等情况，我们最好是将磁盘进行一次格式化还原操作，这里我们可以下载分区大师，将磁盘进行格式化，删除分区操作，做完这些我们可以进入我的电脑的磁盘管理界面查看我们刚安装的硬盘是否都变成未分配了，如果状态没有及时更新，我们可以右键及时刷新一下。

```
注意事项：我们这里是在win10的环境下安装ubuntu,有很多博客都会讲到在一个引导问题，
即新安装的ubuntu在另一块硬盘，导致开机的时候自动进入win10,无法进入ubuntu系统
有些博主的做法是在Windows所在盘中分一个区，然后将ubuntu系统的/boot分区安装到Windows分的这个区中，
即可解决问题，但是我尝试了这种做法没有成功，可能跟安装的系统版本或者uefi启动安装的类型有关，所以本文未采用上述的这种方法
```


### 4.安装系统

```
1.u盘启动，我这里是直接F12就会提示启动项，会出现两个u盘启动项，一个标识的是uefi，
一个没有标识，我们这里选择没有标识的启动项(部分主板可能不支持直接选择，这个时候需要你到dos里直接改成上面说的这个启动项)
2.选择语言(这里暂时选中文，因为出错了好歹有个提示能看得懂)，等一会儿，我们直接选择 try install ubutu,
进入到ubuntu系统直接点击桌面的install ubuntu
3.选择 normal installation （可选安装时获取更新）
4.分区，点击+号依次添加分区(怎么分区可以看后面问题解决附带的分区指导)，然后选择 启动引导分区为 挂载点/boot的分区
```

### 4.引导修复
不出意外的话，我们安装系统成功，并重启进入了我们原生的win10系统，这个时候我们是没办法直接进入ubuntu系统的，我们可以下载一个easyBCD的软件修复引导

```

1.下载easyBCD安装
2.添加条目，选择GRUB2类型，选择安装时的/boot分区
3.编辑引导菜单->保存
4.然后重启电脑就可以进入ubuntu系统了
```


### 问题解决
* 无法将grub-efi-amd64-signed软件包安装到/target

```
1.这个是因为ubuntu自己的bug，我们可以在安装的时候选择 “安装时更新” 进行修复

2.如果选择了“安装时更新”还是会出现这个问题，则应该是分区的问题，在uefi启动的引导里需要我们分配一个efi的区，下面列出分区信息
[
1. 容量:20480   主分区    起始位置 ext4日志文件系统   挂载点：/
2. 容量：10480  逻辑分区  起始位置 交换文件系统swap   挂载点：无
3. 容量：20480  逻辑分区  起始位置  ext4日志文件系统  挂载点：/boot
4. 容量：10480  逻辑分区  起始位置  efi文件系统       挂载点：无
5. 容量：全部   逻辑分区  起始位置  ext4日志文件系统  挂载点：/home
] 注意：如果不是uefi启动的，则应该分区这里是没用efi文件系统的，
而且这里还有一个大坑，就是你的机器可能不支持ext4文件系统，此时就需要换成ext3日志文件系统

```


* 安装完成启动ubuntu，卡黑屏而且左上角光标一直在闪烁

```
这个一般是在装双系统，自己设置的启动引导的问题，我们一般采用的easyBCD软件来设置，这里注意，我们选择类型的时候应该选择GRUB2，然后驱动器选择自己安装系统分区 /boot 的这个分区
```

* 安装完全启动ubuntu,卡在GRUB命令行

```
这个问题就比较复杂了，我先说一下我的解决办法。
1.我的硬盘是MBR类型的，所以我选的BIOS+MBR来安装系统
2.我的电脑不支持ext4日志文件系统，我在分区的时候 /,/home,/boot都选择ext3日志文件系统(ext3分区的时候需要把 挂载点“/”放在最后分)
3.easyBCD选择GRUB2方式
```

* 安装过程中提示 还原应用失败...

```
虽然这个错误不影响全局，但是强迫症不能忍，
发生这个错误的原因可能是下面其中的一个
1.安装的硬盘没有格式化(没有恢复到“未分配”这种状态)
2.制作的启动硬盘有问题(尽量用refus来制作，并且尽量用低速磁盘即usb2.0)
```



