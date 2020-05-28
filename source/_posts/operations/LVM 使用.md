---
title: LVM 使用
author: wanghuagang
date: 2017-02-14 9:11:00
updated: 2020-05-28 11:50:00
tags: 
- centos
- lvm
categories: 运维
---

本文主要介绍 LVM 的概念和基本使用。环境在 Centos 7　系统上做演示，旨在记录 LVM 基本概念，和常用操作命令，方便以后回忆。

<!--more-->

> LVM是 Logical Volume Manager（逻辑卷管理）的简写，它是Linux环境下对磁盘分区进行管理的一种机制，它由Heinz Mauelshagen在Linux 2.4内核上实现，目前最新版本为：稳定版1.0.5，开发版 1.1.0-rc2，以及LVM2开发版。

## 简介
LVM是逻辑盘卷管理（Logical Volume Manager）的简称，它是Linux环境下对磁盘分区进行管理的一种机制，LVM是建立在硬盘和 分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。通过LVM系统管理员可以轻松管理磁盘分区，如：将若干个磁盘分区连接为一个整块的卷组 （volumegroup），形成一个存储池。 管理员可以在卷组上随意创建逻辑卷组 （logical volumes），并进一步在逻辑卷组上创建文件系统。管理员通过LVM可以方便的调整存储卷组的大小，并且可以对磁盘存储按照组的方式进行命名、管理和分配，例如按照使用用途进行定义：“development”和“sales”，而不是使用物理磁盘名“sda”和“sdb”。而且当系统添加了新的磁盘，通过LVM管理员就不必将磁盘的 文件移动到新的磁盘上以充分利用新的存储空间，而是直接扩展文件系统跨越磁盘即可。

## 基本术语

### 物理存储介质（Physical StorageMedia）

指系统的物理存储设备：磁盘、分区，如：/dev/hda、/dev/sda等，是存储系统最底层的存储单元。

### PV：物理卷（Physical Volume）

指磁盘分区或从逻辑上与磁盘分区具有同样功能的设备（如RAID），是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。

### VG：卷组（Volume Group，）

类似于非LVM系统中的物理磁盘，其由一个或多个物理卷PV组成。可以在卷组上创建一个或多个LV（逻辑卷）。

### LV：逻辑卷（Logical Volume）

类似于非LVM系统中的磁盘分区，逻辑卷建立在卷组VG之上。在逻辑卷LV之上可以建立文件系统（比如/home或者/usr等）。

### PE：物理块（Physical Extent）

每一个物理卷PV被划分为称为PE（Physical Extents）的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。所以物理卷（PV）由大小等同的基本单元PE组成。

### LE：逻辑块（Logical Extent）

逻辑卷LV也被划分为可被寻址的基本单位，称为LE。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

> 系统启动LVM时激活VG，并将VGDA加载至内存，来识别LV的实际物理存储位置。当系统进行I/O操作时，就会根据VGDA建立的映射机制来访问实际的物理位置。

## 安装

    本次使用 Centos 7 操作系统，
    [root@localhost ~]# uname -a 
    Linux localhost.localdomain 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

首先确定系统中是否安装了lvm工具：

```
[root@localhost ~]# rpm -qa | grep lvm 
mesa-private-llvm-3.6.2-2.el7.x86_64
lvm2-2.02.130-5.el7.x86_64
lvm2-libs-2.02.130-5.el7.x86_64
```

如果命令结果输入类似于上例，那么说明系统已经安装了LVM管理工具；如果命令没有输出则说明没有安装LVM管理工具，则需要从网络下载或者从光盘装LVM rpm工具包。

## 创建管理

### 创建分区

使用分区工具（如：fdisk等）创建分区，如果不创建分区也可以直接使用整个磁盘。

**查看磁盘**

```
[root@localhost ~]# fdisk -l

磁盘 /dev/sdc：5368 MB, 5368709120 字节，10485760 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/sdb：5368 MB, 5368709120 字节，10485760 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/sda：85.9 GB, 85899345920 字节，167772160 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0005e789

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   167772159    83373056   8e  Linux LVM

磁盘 /dev/sdd：5368 MB, 5368709120 字节，10485760 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-root：81.1 GB, 81075896320 字节，158351360 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：4294 MB, 4294967296 字节，8388608 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
```

**创建分区** 使用 /dev/sdb 磁盘创建一个 2G 的分区

```
[root@localhost ~]# fdisk /dev/sdb          #使用 fdisk 分区工具对 sdb 分区
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

Device does not contain a recognized partition table
使用磁盘标识符 0x2285b981 创建新的 DOS 磁盘标签。

命令(输入 m 获取帮助)：m          # 显示菜单
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition          # 删除一个分区
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu          # 显示菜单
   n   add a new partition          # 创建一个新的分区
   o   create a new empty DOS partition table
   p   print the partition table          # 显示分区列表
   q   quit without saving changes          # 不保存退出
   s   create a new empty Sun disklabel
   t   change a partition's system id          # 更改分区类型
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit          # 写入磁盘信息并退出
   x   extra functionality (experts only)

命令(输入 m 获取帮助)：n          # 新建一个分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p):          # 回车使用默认，创建主分区
Using default response p
分区号 (1-4，默认 1)：          # 回车使用默认，创建分区号为1
起始 扇区 (2048-10485759，默认为 2048)：           # 回车使用默认起始扇区
将使用默认值 2048
Last 扇区, +扇区 or +size{K,M,G} (2048-10485759，默认为 10485759)：+2G       # 创建大小为 2G
分区 1 已设置为 Linux 类型，大小设为 2 GiB

命令(输入 m 获取帮助)：p          # 显示分区列表

磁盘 /dev/sdb：5368 MB, 5368709120 字节，10485760 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x2285b981

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     4196351     2097152   83  Linux

命令(输入 m 获取帮助)：n          # 创建一个分区大小为 1G 的主分区
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): 
Using default response p
分区号 (2-4，默认 2)：
起始 扇区 (4196352-10485759，默认为 4196352)：
将使用默认值 4196352
Last 扇区, +扇区 or +size{K,M,G} (4196352-10485759，默认为 10485759)：+1G
分区 2 已设置为 Linux 类型，大小设为 1 GiB

命令(输入 m 获取帮助)：n          创建分区，使用磁盘剩余空间作为此分区大小
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): 
Using default response p
分区号 (3,4，默认 3)：
起始 扇区 (6293504-10485759，默认为 6293504)：
将使用默认值 6293504
Last 扇区, +扇区 or +size{K,M,G} (6293504-10485759，默认为 10485759)：        # 不填写数值，默认使用剩余空间
将使用默认值 10485759
分区 3 已设置为 Linux 类型，大小设为 2 GiB

命令(输入 m 获取帮助)：p

磁盘 /dev/sdb：5368 MB, 5368709120 字节，10485760 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x2285b981

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     4196351     2097152   83  Linux
/dev/sdb2         4196352     6293503     1048576   83  Linux
/dev/sdb3         6293504    10485759     2096128   83  Linux

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.
正在同步磁盘。
```

### 创建物理卷

创建物理卷的命令为 pvcreate，利用该命令将希望添加到卷组的所有 **分区** 或者 **磁盘** 创建为物理卷。将整个磁盘创建为物理卷的命令为：

将 **分区** 创建为物理卷

```
[root@localhost ~]# pvcreate /dev/sdb1           # 使用sdb1 分区创建物理卷
  Physical volume "/dev/sdb1" successfully created
[root@localhost ~]# pvcreate /dev/sdb2
  Physical volume "/dev/sdb2" successfully created
[root@localhost ~]# pvcreate /dev/sdb3
  Physical volume "/dev/sdb3" successfully created
```

将 **磁盘**创建为物理卷

```
[root@localhost ~]# pvcreate /dev/sdc          # 使用 sdc 磁盘创建物理卷
  Physical volume "/dev/sdc" successfully created
```

### 查看物理卷

查看物理卷的命令是 pvdisplay，可以查看当前系统的物理卷详情。

```
[root@localhost ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               79.51 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              20354
  Free PE               0
  Allocated PE          20354
  PV UUID               Gv6gjF-lcaF-1KOP-mJK1-ejyA-80fc-FUuf0f
   
  "/dev/sdc" is a new physical volume of "5.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  VG Name               
  PV Size               5.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               8x7NNq-FK15-1GLo-UeyK-6nIl-fLr2-FAs2ZH
   
  "/dev/sdb2" is a new physical volume of "1.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb2
  VG Name               
  PV Size               1.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               XA4A2B-W52C-7EoE-lsuv-5KxJ-dLdJ-QGPv1H
   
  "/dev/sdb3" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb3
  VG Name               
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               9DKGiX-RdYb-re1b-P7dq-ZtkX-OGHe-0sqeB8
   
  "/dev/sdb1" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               6ep0dc-VUtR-SdJ8-ZzG6-n1n4-ndkO-ufPXZy
```

> 上面 sda2 是我系统的物理卷，后面为我们前面创建的。

### 创建卷组

创建卷组的命令为vgcreate，将使用pvcreate建立的物理卷创建为一个完整的卷组：

    vgcreate VolumeGroupName PhysicalDevicePath [PhysicalDevicePath...]

vgcreate命令第一个参数是指定该卷组的逻辑名：web_document。后面参数是指定希望添加到该卷组的所有分区和磁盘。vgcreate 在创建卷组web_document以外，还设置使用大小为4MB的PE（默认为4MB），这表示卷组上创建的所有逻辑卷都以4MB为增量单位来进行扩充 或缩减。由于内核原因，PE大小决定了逻辑卷的最大大小，4MB的PE决定了单个逻辑卷最大容量为256GB，若希望使用大于256G的逻辑卷则创建卷组 时指定更大的PE。PE大小范围为8KB到512MB，并且必须总是2的倍数（使用-s指定，具体请参考manvgcreate）。（centos 6.2系统已发现没有这种限制）

```
[root@localhost ~]# vgcreate test_volume /dev/sdb1 /dev/sdc         # 后面指定添加的物理卷
  Volume group "test_volume" successfully created
```

### 查看卷组

查看卷组的命令为 vgdisplay，可以查看某个卷组的详细信息。

因为前面分区 sdb1 大小为 2G，所以创建的物理卷 sdb1 大小为 2G 。sdc 磁盘为5G。 上面把它们分给 test_volume 卷组。所以此时卷组的剩余空间为 2G，即有 2G 空间可以再分配。

```
[root@localhost ~]# vgdisplay           # 不指定参数时，会查看所有卷组
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               6.99 GiB
  PE Size               4.00 MiB
  Total PE              1790
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1790 / 6.99 GiB
  VG UUID               jzKmzc-IJbU-cQR8-clm3-dWIC-Nvzb-pdCFPf
   
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               79.51 GiB
  PE Size               4.00 MiB
  Total PE              20354
  Alloc PE / Size       20354 / 79.51 GiB
  Free  PE / Size       0 / 0   
  VG UUID               Abfqd4-N3hG-BJR1-2j2a-VYFQ-cRXI-OtmJmR

[root@localhost ~]# vgdisplay test_volume          # 指定查看某个卷组
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               6.99 GiB
  PE Size               4.00 MiB
  Total PE              1790
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1790 / 6.99 GiB          # 剩余空间为 7G
  VG UUID               M54Oct-igCs-FOwl-kI0S-Bf4f-sXOi-nIw5Yw
```

### 激活卷组

为了立即使用卷组而不是重新启动系统，可以使用vgchange来激活卷组：

```
[root@localhost ~]# vgchange -ay test_volume 
  0 logical volume(s) in volume group "test_volume" now active
```

### 扩展卷组

当系统安装了新的磁盘并创建了新的物理卷，而要将其添加到已有卷组时，就需要使用vgextend 命令：
```
[root@localhost ~]# pvcreate /dev/sdd
  Physical volume "/dev/sdd" successfully created
[root@localhost ~]# vgextend test_volume /dev/sdd
  Volume group "test_volume" successfully extended
[root@localhost ~]# vgdisplay test_volume        
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               11.99 GiB
  PE Size               4.00 MiB
  Total PE              3069
  Alloc PE / Size       0 / 0   
  Free  PE / Size       3069 / 11.99 GiB          # 7G + 5G 
  VG UUID               M54Oct-igCs-FOwl-kI0S-Bf4f-sXOi-nIw5Yw
```

### 减小卷组

要从一个卷组中删除一个物理卷，首先要确认要删除的物理卷没有被任何逻辑卷正在使用，就要使用pvdisplay命令察看一个该物理卷信息：
如果某个物理卷正在被逻辑卷所使用，就需要将该物理卷的数据备份到其他地方，然后再删除。删除物理卷的命令为vgreduce：

```
[root@localhost ~]# vgreduce test_volume /dev/sdc
  Removed "/dev/sdc" from volume group "test_volume"
[root@localhost ~]# vgdisplay test_volume        
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               6.99 GiB
  PE Size               4.00 MiB
  Total PE              1790
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1790 / 6.99 GiB
  VG UUID               M54Oct-igCs-FOwl-kI0S-Bf4f-sXOi-nIw5Yw
```

### 删除卷组

当不再使用某个卷组，可以删除它。删除前，请务必备份好资料！

```
[root@localhost ~]# vgremove test_volume 
  Volume group "test_volume" successfully removed
[root@localhost ~]# vgdisplay           # 查看所有卷组， test_volume 消失
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               79.51 GiB
  PE Size               4.00 MiB
  Total PE              20354
  Alloc PE / Size       20354 / 79.51 GiB
  Free  PE / Size       0 / 0   
  VG UUID               Abfqd4-N3hG-BJR1-2j2a-VYFQ-cRXI-OtmJmR
```

### 创建逻辑卷

使用 lvcreate 创建逻辑卷。该命令就在卷组 test_volume 上创建名字为 testLV，大小为 3G 的逻辑卷，并且设备入口为 /dev/test_volume/testLV （test_volume 为卷组名， testLV 为逻辑卷名）。 创建之前，请先查看卷组的大小。

```
[root@localhost ~]# lvcreate -L 3G -n testLV test_volume 
  Logical volume "testLV" created.
```

### 查看逻辑卷

```
[root@localhost ~]# lvdisplay /dev/test_volume/testLV     # 不加参数查看全部逻辑卷
  --- Logical volume ---
  LV Path                /dev/test_volume/testLV
  LV Name                testLV
  VG Name                test_volume
  LV UUID                dmtbfb-XxQK-WnL1-K5BB-FdWA-FZ1z-aeexLn
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-02-11 22:00:00 +0800
  LV Status              available
  # open                 0
  LV Size                3.00 GiB
  Current LE             768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

### 创建文件系统，并使用

**创建文件系统**

```
[root@localhost ~]# mkfs.ext4 /dev/test_volume/testLV    
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
196608 inodes, 786432 blocks
39321 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=805306368
24 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (16384 blocks): 完成
Writing superblocks and filesystem accounting information: 完成 

```

**挂载到目录**

> *注意：* 挂载到目录之前，请备份好目录内的文件！

```
[root@localhost ~]# mount /dev/test_volume/testLV /home/MyVolume/
```

**配置开机自动挂载文件系统**

编辑 /etc/fstab 文件，追加文件系统信息
```
[root@localhost ~]# vi /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Fri Feb 10 16:38:19 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=5725d92e-05b1-4f52-bc62-4de75b503be0 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/test_volume/testLV         /home/MyVolume/ ext4    default         1 2
```

### 删除一个逻辑卷

删除逻辑卷之前需要将其卸载后才能删除。请备份文件系统内文件！

```
[root@localhost ~]# umount /home/MyVolume/
[root@localhost ~]# lvremove /dev/test_volume/testLV 
Do you really want to remove active logical volume testLV? [y/n]: y
  Logical volume "testLV" successfully removed
```

### 扩展逻辑卷

LVM提供了方便调整逻辑卷大小的能力，扩展逻辑卷大小的命令是lvextend。扩展之前请先查看卷组可用空间大小。如果可用空间不足，可用使用新的磁盘创建物理卷后添加至卷组。

**查看卷组大小**   可用空间为 3G

```
[root@localhost ~]# vgdisplay test_volume 
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               6.99 GiB
  PE Size               4.00 MiB
  Total PE              1790
  Alloc PE / Size       768 / 3.00 GiB
  Free  PE / Size       1022 / 3.99 GiB          # 可用空间为 3G 
  VG UUID               WDFqaI-Qopl-fDJM-eeMl-3U20-aCpc-umR44I
```

**扩展卷组**   扩展后可用为 6G

```
[root@localhost ~]# vgextend test_volume /dev/sdb3
  Volume group "test_volume" successfully extended
[root@localhost ~]# vgdisplay test_volume         
  --- Volume group ---
  VG Name               test_volume
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               8.99 GiB
  PE Size               4.00 MiB
  Total PE              2301
  Alloc PE / Size       768 / 3.00 GiB
  Free  PE / Size       1533 / 5.99 GiB          # 可用空间为 6G
  VG UUID               WDFqaI-Qopl-fDJM-eeMl-3U20-aCpc-umR44I
```

**查看当前逻辑卷**  逻辑卷大小为 3G

```
[root@localhost ~]# lvdisplay /dev/test_volume/testLV 
  --- Logical volume ---
  LV Path                /dev/test_volume/testLV
  LV Name                testLV
  VG Name                test_volume
  LV UUID                k3WITF-SIK1-MVhz-MdNr-exdX-5qIm-2d2IOr
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-02-11 22:17:13 +0800
  LV Status              available
  # open                 0
  LV Size                3.00 GiB
  Current LE             768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

**扩展逻辑卷**

```
[root@localhost ~]# lvextend -L 6G /dev/test_volume/testLV 
  Size of logical volume test_volume/testLV changed from 3.00 GiB (768 extents) to 6.00 GiB (1536 extents).
  Logical volume testLV successfully resized.
```

**查看扩展后的逻辑卷**  逻辑卷大小为 6G

```
[root@localhost ~]# lvdisplay /dev/test_volume/testLV 
  --- Logical volume ---
  LV Path                /dev/test_volume/testLV
  LV Name                testLV
  VG Name                test_volume
  LV UUID                k3WITF-SIK1-MVhz-MdNr-exdX-5qIm-2d2IOr
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-02-11 22:17:13 +0800
  LV Status              available
  # open                 0
  LV Size                6.00 GiB
  Current LE             1536
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

### 注意

当向已经格式化文件系统的卷组中扩展物理卷的时候，请在创建物理卷之前将磁盘或分区格式化相应的系统。比如当我们想要向系统根目录扩展空间，因为 Centos 系统安装为 ext4 文件系统。所有需要将分区格式化为 ext4 后再扩展！！！

## 总结

LVM具有很好的可伸缩性，使用起来非常方便。可以方便地对卷组、逻辑卷的大小进行调整，更进一步调整文件系统的大小。














