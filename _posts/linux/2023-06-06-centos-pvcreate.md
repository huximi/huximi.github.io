---
layout:     post
title:      Linux 逻辑卷的创建、扩展、删除
subtitle:   Linux 逻辑卷的创建、扩展、删除
date:       2023-06-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - centos7
    - linux
    - pvcreate
---

# Linux 逻辑卷的创建、扩展、删除

## 测试环境

操作系统：`centos7`

## 原始环境的卷组状态：

|硬盘         |物理卷PV  |卷组VG|逻辑卷LV                           |大小|挂载点|
|------------|---------|------|---------------------------------|----|-----|
|/dev/sda 20G|/dev/sda1|      |                                 |500m|/boot|
|            |/dev/sda2|vg_zg1|/dev/mapper/vg_zg1-lv_swap       |8G  |Swap |
|            |                |vg_zg1|/dev/mapper/vg_zg1-lv_root|11G |/    |

## 测试目标：

现在添加两块新的硬盘`/dev/sdb 20G`  `/dev/sdc 10G`。
需完成如下测试：
- 1、新创建一个`VG（vg2）`，使用`/dev/sdb`;
- 2、在原始`VG(vg2)`上添加新的硬盘`/dev/sdc`;

|硬盘         |物理卷PV  |卷组VG|逻辑卷LV                           |大小|挂载点|
|------------|---------|------|---------------------------------|----|-----|
|/dev/sda 20G|/dev/sda1|      |                                 |500m|/boot|
|            |/dev/sda2|vg_zg1|/dev/mapper/vg_zg1-lv_swap       |8G  |Swap |
|            |                |vg_zg1|/dev/mapper/vg_zg1-lv_root|11G |/    |
|/dev/sdb 20G|/dev/sdb1|vg2   |/dev/mapper/vg2-data             |10G |/data|
|/dev/sdc 10G|/dev/sdc1|vg2   |/dev/mapper/vg2-data             |10G |/data|

## 一、逻辑卷的创建

### 1、`/dev/sdb`磁盘分区`/dev/sdb1`
```
[root@zg1 ~]# fdisk /dev/sdb

n

p

1

回车

回车

t

8e

w
```

### 2、创建物理卷`/dev/sdb1`
```
[root@zg1 ~]# pvcreate /dev/sdb1

[root@zg1 ~]# pvs

[root@zg1 ~]# pvdisplay
```

### 3、创建卷组`vg2`，并将`/dev/sdb1`物理卷添加到卷组
```
[root@zg1 ~]# vgcreate vg2 /dev/sdb1

[root@zg1 ~]# vgs

[root@zg1 ~]# vgdisplay
```
可以看出默认`PE`大小为`4MB`,`PE`是卷组的最小存储单元.可以通过 **–s参数修改大小**。

### 4、划分逻辑卷
```
[root@zg1 ~]# lvcreate -l 100%VG -n data vg2

[root@zg1 ~]# lvs

[root@zg1 ~]# lvdisplay
```

### 5、在逻辑卷上创建文件系统
```
[root@zg1 ~]# mkfs.ext4 /dev/vg2/data
```

### 6、将文件系统挂载到`/data`上，将挂载信息添加到`/etc/fstab`
```
[root@zg1 ~]# mkdir /data

[root@zg1 ~]# blkid  --查看lv的uuid

[root@zg1 ~]# vi /etc/fstab

--添加

UUID=a7041bfe-4adb-4e5c-bc9f-400f9ac4ba95 /data ext4 defaults   1 1

[root@zg1 ~]# mount -a

[root@zg1 ~]# df -h
```

## 二、逻辑卷的扩展
```
[root@zg1 ~]# fdisk /dev/sdc

n

p

1

回车

回车

t

8e

w

# 创建物理卷/dev/sdc1
[root@zg1 ~]# pvcreate /dev/sdc1

# 将/dev/sdc1扩展到卷组vg2
[root@zg1 ~]# vgextend vg2 /dev/sdc1

# 划分逻辑卷/dev/vg2/data
[root@zg1 ~]# lvextend -l +100%FREE /dev/vg2/data

[root@zg1 ~]# lvdisplay /dev/vg2/data

[root@zg1 ~]# resize2fs /dev/vg2/data   
```

## 三、减少逻辑卷空间
```
[root@zg1 ~]# umount /data   --卸载逻辑卷

[root@zg1 ~]# e2fsck -f /dev/vg2/data  --检测逻辑卷的剩余空间

[root@zg1 ~]# resize2fs /dev/vg2/data 20G  --将文件系统减少到20G

[root@zg1 ~]# lvreduce -L 20G /dev/vg2/data --将逻辑卷减少到20G

[root@zg1 ~]# mount -a  --重新挂载使用
```

## 四、逻辑卷的删除
```
[root@zg1 ~]# lvdisplay

[root@zg1 ~]#umount /dev/vg2/data

[root@zg1 ~]#vi /etc/fstab

[root@zg1 ~]# lvremove /dev/vg2/data
```
卷组的删除
```
[root@zg1 ~]# vgs

[root@zg1 ~]# vgdisplay

[root@zg1 ~]# vgremove vg1
```
将物理卷转换成普通分区
```
[root@zg1 ~]# pvs

[root@zg1 ~]# pvdisplay

[root@zg1 ~]# pvremove /dev/sdc1

[root@zg1 ~]# pvremove /dev/sdb1
```
修改分区`id`标识为普通分区
```
[root@zg1 ~]# fdisk /dev/sdc

t

1

83

w

[root@zg1 ~]# fdisk /dev/sdb

t

1

83

w
```

## 五、逻辑卷的移动

例子：卷组`vg2`有两个物理卷`/dev/sdb1(20G)`，`/dev/sdc1(10G)`。
```
[root@zg1 /]# pvs

  PV         VG     Fmt  Attr PSize  PFree

  /dev/sda2  vg_zg1 lvm2 a--  19.51g     0

  /dev/sdb1  vg2    lvm2 a--  19.99g 19.99g

  /dev/sdc1  vg2    lvm2 a--   9.99g  9.99g

[root@zg1 /]# vgs

  VG     #PV #LV #SN Attr   VSize  VFree

  vg2      2   0   0 wz--n- 29.98g 29.98g

  vg_zg1   1   2   0 wz--n- 19.51g     0
```
在卷组下创建一个逻辑卷`data`占用`8G`空间。
```
[root@zg1 /]# lvcreate -L +8G -n data vg2

  Logical volume "data" created

[root@zg1 /]# pvs

  PV         VG     Fmt  Attr PSize  PFree

  /dev/sda2  vg_zg1 lvm2 a--  19.51g     0

  /dev/sdb1  vg2    lvm2 a--  19.99g 11.99g

  /dev/sdc1  vg2    lvm2 a--   9.99g  9.99g

[root@zg1 /]# mkfs.ext4 /dev/vg2/data

[root@zg1 ~]# blkid  

[root@zg1 ~]# vi /etc/fstab

[root@zg1 ~]# mount -a
```
这个时候如果想把`sdb1`数据转移到`sdbc1`空间
```
[root@zg1 /]# pvmove /dev/sdb1 /dev/sdc1  --转移空间数据

  /dev/sdb1: Moved: 0.3%

  /dev/sdb1: Moved: 100.0%

[root@zg1 /]# pvs                              --查看空间剩余，可以看到数据被转移

  PV         VG     Fmt  Attr PSize  PFree

  /dev/sda2  vg_zg1 lvm2 a--  19.51g     0

  /dev/sdb1  vg2    lvm2 a--  19.99g 19.99g

  /dev/sdc1  vg2    lvm2 a--   9.99g  1.99g

[root@zg1 /]# vgreduce vg2 /dev/sdb1               --从卷组中移除不需要的硬盘

  Removed "/dev/sdb1" from volume group "vg2"

[root@zg1 /]# pvs

  PV         VG     Fmt  Attr PSize  PFree

  /dev/sda2  vg_zg1 lvm2 a--  19.51g     0

  /dev/sdb1         lvm2 a--  19.99g 19.99g

  /dev/sdc1  vg2    lvm2 a--   9.99g  1.99g

 [root@zg1 ~]# pvremove /dev/sdb1  --将sdb1从物理卷中删除
```
然后就可以将硬盘手工拆除了