---
layout:     post
title:      VirtualBox6.1上安装Windows 11
subtitle:   如何在 VirtualBox 6.1 上安装 Microsoft Windows 11
author:     GuYang
header-img: "img/post-bg-2018.jpg"
tags:
    - VirtualBox 6.1
    - Windows 11
---

# 如何在 VirtualBox 6.1 上安装 Microsoft Windows 11！

> 原文 ：[如何在 VirtualBox 6.1 上安装 Microsoft Windows 11](https://blogs.oracle.com/virtualization/post/install-microsoft-windows-11-on-virtualbox)
>

![img](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT47A30F8C7A9045A1B868871EA6093DE7/Thumbnail?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)[西蒙·科特](https://blogs.oracle.com/virtualization/authors/Blog-Author/CORE663DB8EA17ED4B669937CC6C0E87F10D/simon-coter)| 2021 年 9 月 29 日Oracle Linux 和虚拟化产品管理总监

以下分步指南显示了如何在 VirtualBox 6.1 之上安装 Microsoft Windows 11（最新的 Insider Preview 版本 22463.1000）；本指南已在 macOS 和 Linux 主机上进行了测试和验证。这是为了解决报告的 Windows 11 无法安装为 VirtualBox VM 的问题。

注意：这些说明应该也适用于[Microsoft Windows 11 一般可用版本，实际上是针对 2021 年 10 月 5 日](https://blogs.windows.com/windowsexperience/2021/08/31/windows-11-available-on-october-5/)。

第一步是正确配置虚拟机，该虚拟机将作为“Microsoft Windows 11”安装来宾；虚拟机必须按照以下方式配置（最低要求），基于官方的“ [Windows 11 规格和系统要求](https://www.microsoft.com/en-us/windows/windows-11-specifications)”，并且您的系统需要有[适当更新的 x86 CPU](https://docs.microsoft.com/en-us/windows-hardware/design/minimum/windows-processor-requirements)：

- 系统 - 主板
  - RAM：4GB（最低）- 8GB（建议最低）
  - EFI（仅限特殊操作系统）已启用

![系统 - 主板](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT23F3D30F8AD640EB81CE89968AEB9BF2/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

- 系统 - 处理器
  - CPU：2（最低）

![系统 - 处理器](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT2AAC5923EF164E38B31506E4A862C4FE/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

- 贮存
  - 64 GB 虚拟磁盘（最小大小）

![贮存](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT555CDB201A0F455A947258FE1DBA8DB4/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

- 显示屏
  - 显存：256MB
  - 图形控制器：VBoxSVGA
  - 启用 3D 加速已启用

![显示屏](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT81277F53D435464CB3FA050A975124E4/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

使用这些配置选项并将“ **Microsoft Windows 11** ” ISO 正确添加为“**虚拟 cd-rom** ”后，我们就可以开始安装过程：

![Microsoft Windows 11 - 安装](https://blogs.oracle.com/content/published/api/v1.1/assets/CONTF139352542624DCBA4A6E9D48C9B796D/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

单击下一步显示“立即**安装**”按钮；当您看到安装按钮时，同时按下键盘上的“ **Shift+F10** ”以启动命令提示符。在此命令提示符下，键入“ **regedit** ”并按 Enter 键以启动 Windows 注册表编辑器。

![Windows 11 - 注册表编辑器](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT1B93C6DDBF2742F28AC7B2C2B680A60C/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

当注册表编辑器打开时，导航到“ **HKEY_LOCAL_MACHINE\SYSTEM\Setup** ”，右键单击“ **Setup** ”键并选择“ **New => Key** ”。

当提示为密钥命名时，输入“ **LabConfig** ”并按回车键。

现在右键单击“ **LabConfig** ”键并选择“ **New => DWORD (32-bit)** ”值并创建一个名为“ **BypassTPMCheck** ”的值，并将其数据设置为“ **1** ”。使用相同的步骤创建“ **BypassRAMCheck** ”和“ **BypassSecureBootCheck** ”值并将它们的数据设置为“ **1** ”，因此它看起来像下图。

![Windows 11 - 绕过检查](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT767EEF272AD841B1A9D54E950209879D/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

在“ **LabConfig** ”项下配置这三个值后，关闭“**注册表编辑器**”，然后在“**命令提示符**”中键入exit，然后按回车键关闭窗口。您现在可以单击“立即安装”按钮继续将“ **Microsoft Windows 11** ”作为虚拟机安装在 VirtualBox 之上。

![Windows 11 - 安装过程](https://blogs.oracle.com/content/published/api/v1.1/assets/CONTDEA424E3678D43609F6A3E30627422E1/Medium?cb=_cache_7a6f&format=jpg&channelToken=bdc0a0a82e064a5093403b37226ecf1e)

我将保持这篇文章的更新，以防同样不适用于更新的版本或 Microsoft Windows 11 的 GA 版本，显然，我愿意对此发表评论，因此请随时联系[推特在这里](https://twitter.com/scoter80)！

安装完成：

![Windows 11 - 安装完成](/img/vm/windows11-installed.png)

Windows 11 激活：
复制以下文本，另存为.bat文件，并右键选择【管理员身份运行】。
```
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.03k.org
slmgr /ato
```
![Windows 11 - 激活完成](/img/vm/windows11-slmgr.png)