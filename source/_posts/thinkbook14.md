---
title: Thinkbook14+锐龙版安装Linux避坑指南
author: chanoch
toc: true
mathjax: true
date: 2023-02-26 23:25:50
tags:
- Linux
- 6800H
- Thinkbook14+
category:
- Linux
description: Thinkbook14+锐龙版安装Ubuntu22.04 LTS所遇到的坑及解决方法，其它版本也可以参考
---

我在2022年10月在京东购买了Thinkbook14+锐龙版，具体配置参数如下：

Thinkbook14+锐龙版 R7 6800H，16G，512G镁光3400固态硬盘，Realtek 8852BE网卡(已经更换为Intel AX201)，2.8K 90Hz屏幕

到手后删除Windows，安装Ubuntu22.04 LTS版本，记录所遇到的问题

# 关于分区

Ubuntu默认分区一般为Ext4，带LVM的Ext4和ZFS，但我选择选择并推荐使用Btrfs，Btrfs的快照可以结合Timeshift生成备份，有效防止误操作和数据损坏，当然，也可以使用Rsync的方式进行备份，但占用空间较大。

Btrfs已经发展多年，稳定性上已经有较大的提升，并且拥有现代文件系统的特性，可以适用于个人工作站等场景，当然，如果追求绝对的稳定性，可以选用带LVM的Ext4文件系统。



# 关于Realtek网卡

在笔记本无线网卡领域，可以说是Intel一家独大，这几年，开始有一些厂商采用Realtek网卡，由于各种原因，一方面，即使在厂商提供驱动支持的Windows上，也经常有断流等问题发生，另一方面，厂商也无意积极为Linux提供驱动适配。

关于网卡有以下两种解决方法：

- 更换Intel网卡，例如AX201，AX211等，Linux内核中就包含相应的驱动，无需额外配置

- 对于小于6.1版本的内核，可以参考[https://github.com/lwfinger/rtw89](https://github.com/lwfinger/rtw89)，而内核版本大于6.1后，内核中直接包含相应驱动，可以先连接有线网更换内核



# 更换内核

Ubuntu22.04 LTS默认内核为5.19.XX，此版本的内核对于Thinkbook14+ 锐龙版有两个缺陷：

- 虽然包含针对680M核显的AMDGPU驱动，然而版本教老，时常会黑屏重启
- 针对Realtek的网卡也没有包含

可以使用如下命令更换针对OEM厂商的内核，会包含一些标准内核中未包含的专有驱动，并且已经更新到6.1.XX版本：

```
apt install linux-image-oem-22.04c
```

安装完成后，会自动调用脚本生成vmlinuz，initrd.img等文件，为之前的内核备份，并更新GRUB



# SSL相关错误

在Ubuntu22.04中，开始默认使用SSL3，而非之前的SSL1，尽管官方已经源中的大多数包，但如果使用仍旧使用SSL1的第三方软件，仍可能会有各种错误，包括但不限libssl.so.1，libcrypto.so.1.1未找到等

解决方法有两种：

- 安装SSL1的包

  虽然Ubuntu22.04默认不再提供SSL1的安装，但在[Debian软件包网站](https://packages.debian.org/stretch/libssl1.1)仍能下载SSL1的deb包，可以下载后进行安装

  ```
  dpkg -i libssl1.1*.deb
  ```

- 复制相关文件

  许多软件都自带SSL1的相关库，可以复制相关文件到系统目录下



# 使用Timeshift进行备份

可以使用如下命令安装timeshift

```
$ sudo apt install timeshift
```

打开timeshift，会对timeshift进行配置，依次选择备份方式，目录，定期计划等

如果在分区时使用Btrfs系统，可以选用Btrfs方式进行备份，其它系统可以选择Rsync方式，相对来说，利用Btrfs镜像进行备份更节省空间，在选择备份计划时，可以相对设置更频繁的备份

<img src="/home/chanoch/.config/Typora/typora-user-images/image-20230301095158471.png" alt="image-20230301095158471" style="zoom:33%;" />

下图中为备份计划，可以选择每月，每周，每日，每小时，每次启动等，并选择保留对应的镜像数量，超过限制的镜像会被删除

<img src="/home/chanoch/.config/Typora/typora-user-images/image-20230301095356182.png" alt="image-20230301095356182" style="zoom:33%;" />
