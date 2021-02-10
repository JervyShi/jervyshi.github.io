---
layout: post
title: "Mac安装分区无损扩容"
description: "Mac安装分区无损扩容-双硬盘无损扩容实现记录"
category: mac
tags: [mac, hackintosh]
---

### 一、背景
第一次装mac os x的时候考虑到双系统都使用SSD就仅为mac划分了30G的空间，渐渐的不怎么使用win7的时候发现30G不够用了，此时考虑重装的话又要耗费好多时间显然是不值得了。在网上寻觅了好久也却是没有太多关于mac分区无损扩容的文章，最后在远景上找到一个方案[扩容Mac系统盘方法之一](http://bbs.pcbeta.com/viewthread-1156969-1-1.html)，作者写的比较剪短但还是很感谢提供了这样的思路，故我把自己的无损分区过程记录下来，留给后人分享！

注：此无损分区仅讨论MBR引导下的无损分区扩容方案，GPT分区表本身支持无损调整分区大小但不在本文的讨论范围之内。

HDD我的笔记本有两块硬盘，一块120G的SSD，一块500G的HDD。SSD上有三个分区分别装着WIN7、MAC、软件，HDD留有最初安装的WIN7主分区及一些逻辑分区。此次需要解决的问题是把WIN7无损迁移至HDD，并把整个SSD合并为一个分区给MAC使用，当然MAC的迁移也需要是无损的。

### 二、简易解决步骤
#### 必备工具：
安装有WIN PE的U盘一个，如没有可下载[老毛桃pe](http://www.laomaotao.net/)自行制作
##### 步骤：
1. 通过PE将SSD上的WIN7分区通过Ghost工具的Partition to Partition完整copy到HDD上的主分区
2. 把SSD上的软件分区完整合并到HDD的第一个逻辑分区，清空HDD最后一个逻辑分区数据
3. 通过PE重建HDD主分区引导
4. 进入HDD主分区的WIN7，并安装WINDOWS版变色龙
5. 使用DiskGenius把HDD最后一个逻辑分区删除并重建为NTFS格式分区（未格式化状态）
6. 进入MAC使用磁盘工具把HDD最后一个逻辑分区抹掉为*Mac OS 扩展（日志式）*
7. 使用 *Super Dumper!* 把Mac整个分区完整copy至第6步建立的mac分区
8. 重启进入PE用DiskGenius把HDD主分区设置为*活动*分区
9. 重启通过第4部安装的变色龙选择HDD最后一个分区进入copy后的mac系统（确保此分区mac可启动）
10. 重启进入PE使用DiskGenius把SSD上面的三个分区全部删除并新建唯一NTFS格式主分区（未格式化状态）
11. 重启进入HDD上的mac系统并使用磁盘工具把SSD上的未格式化分区抹掉为*Mac OS 扩展（日志式）*
12. 使用 *Super Dumper!* 把当前Mac分区完整copy至SSD上的新mac分区
13. 重启通过HDD上变色龙进入SSD的mac系统
14. 重装mac版变色龙对SSD上的mac进行引导
15. 重启直接把SSD做为第一启动项启动进入Mac，至此已经完成无损迁移

### 三、详细图文步骤解说
1、通过PE将SSD上的WIN7分区通过Ghost工具的Partition to Partition完整copy到HDD上的主分区：

此步骤主要为无损迁移WIN7系统，通过Ghost工具的分区对分区的对拷可轻松实现整个windows系统的迁移，但是如此迁移后会有引导问题，后续会修复，见下图从SSD_MAC至HDD_MAIN

![partition-info](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58o121lfkj206p05pdg7.jpg)


2、把SSD上的软件分区完整合并到HDD的第一个逻辑分区，清空HDD最后一个逻辑分区数据：

此步骤采用直接复制文件的方式，因SSD_SOFT中安装了常用的windows软件，在windows主分区的注册表表中会有部分软件的路径信息，把SSD_SOFT中的全部文件copy到HDD_1中，再进入HDD_MAIN的win7系统后可修改HDD_1的盘符为D盘即可无损使系统及软件全部迁移。

3、通过PE重建HDD主分区引导：

老毛桃PE中桌面上就有重建windows引导的小工具，双击根据提示即可轻松重建引导，如果重建完引导还是无法开机就使用DiskGenius重建下HDD上的MBR引导（此操作不会破坏分区表）

4、进入HDD主分区的WIN7，并安装WINDOWS版变色龙：

没有变色龙将无法引导Mac系统，所以如果WIN7主分区上没有变色龙一定要安装变色龙，下载及安装方法见远景论坛-[10.8 的变色龙 Chameleon_2.2svn_r2235 Mac版+Win版+EFI_Tools](http://bbs.pcbeta.com/viewthread-971434-1-1.html)

5、使用DiskGenius把HDD最后一个逻辑分区删除并重建为NTFS格式分区（未格式化状态）：

格式化的分区Mac下是无法直接对其进行抹掉操作的，所以需要在win下把一个分区删除并重建，但是不能对重建的分区进行格式化操作，新建完如步骤1中的H盘

6、进入MAC使用磁盘工具把HDD最后一个逻辑分区抹掉为*Mac OS 扩展（日志式）*：

Mac只能对Mac下的分区格式进行操作，所以此步骤就是建立一个可以让Mac把本身的所有文件copy过去的分区

![partition-format](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58psc0q9lj20kk0htac5.jpg)

7、使用 *Super Dumper!* 把Mac整个分区完整copy至第6步建立的mac分区：

工具*Super Dumper!*在[扩容Mac系统盘方法之一](http://bbs.pcbeta.com/viewthread-1156969-1-1.html)中有下载，打开软件选择 Copy SSD_MAC to disk1s8 ，using Backup - all files 然后就点*Copy now*

![ssdmactodisk1s8](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58o16oiu0j20ey08e3zl.jpg) 

整个过程根据所需要copy分区的大小需要的时间也不同，我30G的分区33分左右copy完成

![copy success](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58o13glmtj20ey0coq48.jpg)

8、重启进入PE用DiskGenius把HDD主分区设置为*活动*分区：

经过第7步，HDD主分区的活动状态将会被转移到HDD最后一个分区的mac系统上，所以需要进入PE把HDD主分区设置为活动，这样就可以启动win7或者是windows版的变色龙

9、重启通过第4部安装的变色龙选择HDD最后一个分区进入copy后的mac系统（确保此分区mac可启动）：

此步骤十分重要，通过变色龙启动HDD分区上的mac很有可能不成功，如不成功，请在变色龙启动HDD mac分区之前加上*-v -f*参数自行观察错误信息并解决后继续，如果不能确保此分区Mac无问题就把SSD上的Mac干掉后果……

10、重启进入PE使用DiskGenius把SSD上面的三个分区全部删除并新建唯一NTFS格式主分区（未格式化状态）：

此步骤同第5步

![partition-format2](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58o11cvepj206m04bgls.jpg)

11、重启进入HDD上的mac系统并使用磁盘工具把SSD上的未格式化分区抹掉为*Mac OS 扩展（日志式）*：

此步骤同第6步

12、使用 *Super Dumper!* 把当前Mac分区完整copy至SSD上的新mac分区：

此步骤同第7步

13、重启通过HDD上变色龙进入SSD的mac系统：

直接启动SSD上的mac是成功不了的，因为mac版的变色龙失效了，所以先从HDD上的windows版变色龙启动SSD上的mac

14、重装mac版变色龙对SSD上的mac进行引导

15、重启直接把SSD做为第一启动项启动进入Mac，至此已经完成无损迁移：

扩容成功，此思路同样适合单硬盘mac分区无损扩容，扩容后效果

![disk-info](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/713d9449jw1e58o15f1j2j20g90a6myu.jpg)
