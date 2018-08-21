---
layout: post
title: vdb or vdb1 how to find file system partition start sector 
date: 2018-08-19 10:00:00.000000000 +08:00
tags: partition start sector
---


# vdb or vdb1 how to find file system partition start sector ?

There are some issues in kvm public cloud virtual machines, for incorrect partition table or file system corruption, which leads to the disk can not be mounted or the host will be not booted and entered into rescue model.  The root case is always misconfiguration of fstab,  bewildered with vdb or vdb1。 This article will give a general method to solve it.

在KVM云主机中，很多人因为/dev/vdb或者/dev/vdb1不区分造成主机没法启动或者磁盘分区没法挂载的情况，本文给出一种ext3文件系统下的一般的处理方法供参考。解释清楚这个问题之前先看磁盘分区结构和ext3数据分区格式。ubuntu和CentOS系统有差异，分别介绍。

## 磁盘头引导部分
引导部分包括1个MBR引导扇区和0填充信息，不同的版本的Linux fdisk工具依据不同，分区的时候开始位置不同，造成0填充的长度不一样。

### ubuntu默认分区开始Sector是2048
#### winhex下的磁盘分区结构

![image]({{site.baseurl}}/assets/img/0819/disk_structure.jpg)

可以看到一个Linux下的文件系统分区前存在1MiB的空间。
#### 对应fdisk下查看关系
![image]({{site.baseurl}}/assets/img/0819/disk_structure1.jpg)

可以计算一下

```
sector = 512 bytes

2048 sector * 512 bytes/sectors = 1 * 1024 * 1024 bytes = 1MiB
``` 

从winhex直接查看磁盘或者fdisk查看可以知道**磁盘分区前有1MiB空间**。这1MiB的空间，除了前512MiB的MBR，剩余是0填充的。
![image]({{site.baseurl}}/assets/img/0819/ubuntu.jpg)


### CentOS默认分区开始Sector是63

同样分析CentOS可以得到如下：


- 0-1 sector为MBR
- 2-63 sector为0填充 
![image]({{site.baseurl}}/assets/img/0819/centos.jpg)

## Ext3 磁盘数据结构
![image]({{site.baseurl}}/assets/img/0819/disk_data_structure.jpg)

这里先看第0个Block group，因为我们要分析分区引导位置，所以从第一个开始。
根据ext4文件系统数据格式（https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout）知道（同样适合ext3）, 第0个block group中开头存在1024B的预留用来安装x86的引导和填充。具体可以见：
'''

**For the special case of block group 0**, the first 1024 bytes are unused, to allow for the installation of x86 boot sectors and other oddities. The superblock will start at offset 1024 bytes, whichever block that happens to be (usually 0). However, if for some reason the block size = 1024, then block 0 is marked in use and the superblock goes in block 1.** For all other block groups, there is no padding.**
'''
通过这句话可以，市面上或者网络上的磁盘数据结构都忽略了第0个group的前1024B，预留这里也懒得重新绘图了。

## 超级块数据结构
Total size is 1024 bytes. 很长自行查看，这里关注特征码，magic相对Magic signature位置。

![image]({{site.baseurl}}/assets/img/0819/magic.jpg)


### ubuntu Magic signature位置计算


![image]({{site.baseurl}}/assets/img/0819/ubuntu_magic.jpg)

根据之前分析

2048磁盘头引导Sector + group 0开头2个Sector填充 + 0超级块Magic Signature偏移 438

= 2048 * 512 + 2 * 512 + 0x38

= 1049656

= 0x100438


### CentOS Magic signature位置计算
![image]({{site.baseurl}}/assets/img/0819/centos_magic.jpg)


63磁盘头引导Sector + group 0开头2个Sector填充 + 0超级块Magic Signature偏移 438
= 63 * 512 + 2 * 512 + 0x38
= 33336
= 0x8238
hexdump看到地址8230
P.S.这里为了数据安全匹配的时候使用了：ef53 000

- 0x38	__le16	s_magic	Magic signature, 0xEF53


- 0x3A	__le16	s_state	File system state. Valid values are:

       0x0001	Cleanly umounted,0x0002	Errors detected,0x0004	Orphans being recovered

所以可以使用ef53 000进行匹配。

## CentOS上的一个分区开始位置计算题

![image]({{site.baseurl}}/assets/img/0819/probems.jpg)

如何计算/dev/vdb1的分区开始位置是1234？

![image]({{site.baseurl}}/assets/img/0819/answer.png)

0x009a830-0x00430 = 0x0091400 (hexdump计算的16进制，这里使用)

0x0091400 = 十进制 631808

631808 / 512 bytes/sector = 1234 

## 最后一个问题vdb和vdb1的问题
对于直接整体格式化的磁盘，实际上是没有分区表的，不建议这么使用磁盘，在扩容或者其他情况下问题比较多不直观，不建议这么使用。

![image]({{site.baseurl}}/assets/img/0819/entire.jpg)

这里赫然有提示，是个整体的磁盘，不是分区，当然也就没有分区表，继续hexdump可见。

![image]({{site.baseurl}}/assets/img/0819/entrie2.jpg)

这里直接grep到的位置就是0x438,完全没有包括计算分区表位置。

也就是说以后对于CentOS匹配grep "ef53 000"或"ef53"的时候是0x430说明分区是整体，如果匹配到其他值，减去0x430,除以block size就可以计算出分区起始位置。

写道这里已经8/19/2018 1:15:31 AM 了，问一声“你幸福吗？”