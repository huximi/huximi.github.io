---
layout:     post
title:      macOS 11 Big Sur镜像制作与安装
subtitle:   如何制作macOS镜像及在ESXi7上安装macOS 11 Big Sur
author:     GuYang
header-img: "img/post-bg-2018.jpg"
tags:
    - ESXi7
    - macOS 11 Big Sur
---

# 如何在ESXi7上安装macOS 11 Big Sur

### Installing macOS 11 Big Sur on ESXi 7

使用vmware esxi安装macos能达到白苹果的使用感受。本文适用于情景：喜欢macos，又想省点钱，同时又不想折腾黑苹果。

- 文章作者由[panjie](https://www.codedemo.club/author/panjie/)
- 发布日期[2021-06-18](https://www.codedemo.club/install-macos-11-big-sur-on-esxi7/)

软件需求：

1. [macOS 11 Big Sur ISO](https://apps.apple.com/us/app/macos-big-sur/id1526878132)
2. [VMware ESXi7 Update 2A](https://my.vmware.com/en/web/vmware/downloads/details?downloadGroup=ESXI70U2A&productId=974&rPId=55457)
3. [macOS Unlocker](https://github.com/erickdimalanta/esxi-unlocker/releases)

# macOS 11 Big Sur ISO

首先我们需要一个用于安装macOS系统的iso文件。如果你直接从相关网站上下载了相关文件，在下载后则需要确认该文件的sha1和md5值，以免文件在下载过程损坏影响安装。

## 下载 macOS 11 Big Sur ISO

如果你使用的是macOS操作系统，则可以打开[Big Sur官方发布站点](https://apps.apple.com/us/app/macos-big-sur/id1526878132?mt=12)

进行界面后点击获取按钮。点击继续后便开始进行下载。至本文发布时，系统的版本为11.4 ，大小为12.4G。

下载完成后，如果系统自动打开了安装程序，则切**不**可点下一步，需要按`Command + Q `退出。

此时便表明已成功的下载了安装文件。

## 将安装文件转换为ISO文件

接下来，我们使用的一系列操作均需要root权限。我们打开shell，并输入`sudo -i`来切换至root用户。

```bash
sudo -i
```

输入密码后，便切换到了root权限。

接下来我们运行如下命令：

```bash
hdiutil create -o /tmp/bigsur -size 12900.1m -volname bigsur -layout SPUD -fs HFS+J
```

上述命令的作用是在`/tmp`文件夹下创建一个`dmg`格式的镜像文件，该文件的大小为12.9G。

接下来，我们使用命令来挂载这个dmg镜像文件：

```bash
hdiutil attach /tmp/bigsur.dmg -noverify -mountpoint /Volumes/bigsur
```

上述命令将刚刚创建的镜像文件挂载为`/Volumes/bigsur`。

接着使用如下命令将此镜像设置为可启动macos的启动介质。

```bash
sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/bigsur --nointeraction
```

这需要一定的时间，在过程中会显示进度信息，请耐心等待。完成后将在`/Volumes`中生成`Install macOS Big Sur`。接着弹出它：

```bash
hdiutil eject -force /volumes/Install\ macOS\ Big\ Sur
```

此时`dmg`文件写入完成。下一步将其转换为`cdr`文件。`cdr`文件即是`macOS`系统上的`iso`文件。

```bash
hdiutil convert /tmp/bigsur.dmg -format UDTO -o /Users/hyy/Desktop/install-macOS-Big-Sur.cdr
```

**注意：**不要照抄上面的命令，你需要将上述命令的`yourname`替换为你自己的用户名。

执行该命令同样需要一定的时候，这取决于我们的硬盘速度，整个过程会有进度提示，请耐心等待。

上述命令完成后，我们将在桌面得到一个名为`install-macOS-Big-Sur.cdr`的文件，接着将其重命名为`iso`文件。

```bash
mv /Users/hyy/Desktop/install-macOS-Big-Sur.cdr /Users/hyy/Desktop/install-macOS-Big-Sur.iso
```

同样还是注意把`yourname`换成你的用户名，或者不使用命令也是可以的：

1. 选择桌面上的bigsur.cdr文件
2. 重命名为install-macOS-Big-Sur.iso文件

## 清除

得到iso文件后，历史在`/tmp`文件夹的`dmg`文件就已经完成历史使命了。

```bash
rm -rf /tmp/bigsur*
```

## 将ISO文件上传到数据存储

有了ISO文件后，我们接下来将其上传到VMWare ESXi的数据存储上。

1. 登录ESXi 主机
2. 选择左侧的存储后，选择一个预上传文件的存储。
3. 点击右侧的Datastroe brower(数据浏览)
4. 选择一个预上传的文件夹
5. 点击上传

上传过程需要一定的时候，可以在下侧的最近任务状态中查看上传结果。

## 安装macOS Unlocker

经过上面的操作后，生成的用于安装macos的iso文件就已经被上传至vmware的数据存储中了。接下来我们需要为vmware安装一个[unlocker](https://github.com/shanyungyang/esxi-unlocker)。它的作用是防止在vmware上安装macos时出现的死循环（听说如果我们在是mac主机上安装的esxi，然后再在此exsi上安装macos的话，就不需要将unlocker）.

## 下载Unlocker

点击[下载链接](https://github.com/shanyungyang/esxi-unlocker/archive/refs/heads/master.zip)将得到一个压缩包，解压被进入文件夹后执行`./esxi-build.py`命令：

```bash
./esxi-build.py
```

**注意：**执行过程需要调用系统中的`tar`，如果未找到则会终止执行。解决方法是安装`gnu-tar`。使用brew的话，执行：`brew install gnu-tar`。

执行成功后，将得到一个名为`esxi-unlocker-xxx.tgz`的文件，该文件即我们需要的`unlokcer`文件。

## 上传至数据存储

与上传`iso`文件的方法相同，将`esxi-unlocker-xxx.tgz`上传至`vmware`的存储上。

## 安装Unlocker

vmware本质也是一个liunx操作系统，进行vmware并在服务上开启ssh登录后，使用ssh登录vmware。

存储位于`root`文件夹下`vmfs/volumes`文件夹下，进行上传的存储文件夹并执行：

```bash
tar xzvf esxi-unlocker-xxx.tgz
```

完成压缩包的解压。

### 安装

解压后进入触压文件夹，首先执行:

```bash
./esxi-smctest.sh
```

确认安装状态为`fasle`，即未安装。

接着执行：

```bash
./esxi-install.sh
```

完成安装后，将得到重新启动服务的提示。

```bash
reboot
```

注意：如果在安装过程中提示`Permissin denied`权限错误，则在执行上述命令前可以将整个文件的权限设置为`775`，比如：

```bash
chmod 775 -R esxi-unlocker-xxx/
```

### 验证

机器重新启动后，继续使用`ssh`登录vmware，进行相应的文件夹，重新执行以下命令查看安装结果：

```bash
./esxi-smctest.sh
```

此时将得到一个返回值为`true`的结果，说明安装成功。

## 安装macOS 11 Big Sur虚拟机

登录vmware后，点击管理虚拟机 -> 创建注册虚拟机 -> 创建一个全新的虚拟机, 然后点击next。

填写虚拟机的名称，兼容性选择最新，OS选择macos，版本选择11.4。接着点击下一步
然后选择存储的位置，下一步。
选择CPU核心数、内存、硬盘大小等，最后在CD/DVD中选择使用iso文件，并选择我们前面上传的install-macOS-Big-Sur.iso。点击下一步、完成。

再然后，开机，选择磁盘工具、下一步、格式化磁盘，格式选择`APFS` --`GUID Partition Map`，最后点击擦除，完成后点击完成，最后点击左上方的Disk Utility菜单，退出。

再根据情况选择第二项（一般用户）或是第一项（有时间机想直接恢复的）-> 继续 -> 同意 -> 选择磁盘 -> 下一步

再然后就是等待安装成功，选择语言。

然后，就没有然后了。iCloud、iMessage等一切正常！

参考原文：https://vmscrub.com/installing-macos-11-big-sur-on-esxi-7-update-1/#