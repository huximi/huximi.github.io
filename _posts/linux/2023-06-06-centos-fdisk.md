---
layout:     post
title:      在 CentOS 7 上新增硬盘挂载
subtitle:   在 CentOS 7 上新增硬盘挂载
date:       2023-06-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - centos7
    - fdisk
---

# 在 CentOS 7 上新增硬盘挂载

1. 查看磁盘信息
   在系统上插入新硬盘后，可以使用以下命令查看磁盘信息：

   ```
   fdisk -l
   ```

   该命令可以列出系统中所有硬盘的信息，包括已经挂载的和未挂载的。

2. 分区和格式化
   如果新硬盘还没有被分区和格式化，需要进行如下操作：

   ```
   fdisk /dev/sdb
   ```

   然后根据提示输入相关命令进行分区。
   在使用 `fdisk /dev/sdb` 命令后，会进入磁盘分区工具的命令行模式。您需要进行以下步骤来挂载新硬盘：

    - 输入 n，创建一个新分区。
    - 输入 p，选择主分区。
    - 根据提示输入分区编号、起始位置、结束位置等信息，如果您想使用整个硬盘作为一个分区，可以直接按回车键使用默认值。
    - 输入 w，保存分区表。

   此时您已经成功创建了一个新分区，但仍需要进行以下步骤来格式化分区和挂载分区：

   使用 pvcreate 命令创建物理卷，例如：
   ```
   pvcreate /dev/sdb1
   ```   
   这将创建一个新的物理卷 `/dev/sdb1`，可以使用 pvs 命令查看该物理卷的信息。

   使用 `vgcreate` 命令创建卷组：
   ```
   # 创建
   vgcreate centos /dev/sdb1
   # 扩展
   vgextend centos /dev/sdb1
   # 删除
   vgremove centos   
   ```
   这将创建一个名为 `centos` 的卷组，使用 `/dev/sdb1` 物理卷作为存储空间，可以使用 `vgs` 命令查看该卷组的信息。

   划分逻辑卷
   ```
   lvcreate -l 100%VG -n home centos
   ```

   格式化新的逻辑卷,此处以 xfs 文件系统为例。
   ```
   mkfs.xfs /dev/centos/home
   ``` 

3. 创建挂载点
   需要为新硬盘创建挂载点。可以选择一个空目录作为挂载点：

   ```
   mkdir /home
   ```

4. 修改 `/etc/fstab` 文件

   ```
   vi /etc/fstab
   ```
   在 `/etc/fstab` 文件中添加一行，以自动挂载新硬盘：

   ```
   /dev/mapper/centos-home /home xfs defaults 0 0
   ```

   这里的 `/dev/mapper/centos-home` 是逻辑卷，`/home` 是挂载点，`xfs` 是文件系统类型，`defaults` 是挂载选项，`0 0` 是文件系统检查选项。

5. 挂载新硬盘
   运行以下命令挂载新硬盘：

   ```
   mount -a
   ```

   然后可以使用以下命令查看新硬盘是否已经挂载：

   ```
   df -h
   ```

   如果 `/home` 出现在列表中，说明新硬盘已经成功挂载。


