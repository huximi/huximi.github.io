---
layout:     post
title:      Centos7服务器固定IP
subtitle:   Centos7服务器固定IP
date:       2023-06-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - centos7
    - network
---

# Centos7服务器固定IP：

## 服务器
```
ip: 192.168.2.11
```
## 修改固定IP

输入
```
vi /etc/sysconfig/network-scripts/ifcfg-ens160
```

配置IP：
```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
NAME="ens160"
UUID="8d37c26f-b37d-4215-916d-b66215a1d611"
DEVICE="ens160"
ONBOOT="yes"
IPADDR="192.168.2.11"
NETMASK="225.225.225.0"
GATEWAY="192.168.2.1"
DNS1="114.114.114.114"

```

> 手动修改
> BOOTPROTO=static静态获取IP，
> ONBOOT=yes开机自动启动网卡,
> 在末尾加入:
> IPADDR=192.168.2.*系统IP地址配置，
> 子网掩码：NETMASK=255.255.255.0，
> 网关：GATEWAY=192.168.2.1,
> 域名解析服务器：DNS1=114.114.114.114
> 输入ZZ保存且退出：

## 重启网络服务
```
service network restart
```

## 验证是否可以连接外网
```
ping www.baidu.com
```

## 修改主机名
方法1：`hostnamectl`

在 Linux 中使用 `hostnamectl` 来改变主机名
```
hostnamectl set-hostname myname
```
然后使用 `hostnamectl` 查看

方法2：修改`/etc/hostname`

修改配置文件`/etc/hostname`来实现主机名的修改。把该文件内容`hostname name`中的`name`替换成自己想要的主机名重启即可。
```
vim /etc/hostname 
hostname  myname
```

方法3：`nmtui`:

通过`nmtui`修改，之后重启`hostnamed`
```
nmcli general hostname myname
systemctl restart systemd-hostnamed
```