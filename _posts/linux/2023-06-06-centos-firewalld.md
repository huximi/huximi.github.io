---
layout:     post
title:      CentOS 7 firewalld-防火墙
subtitle:   CentOS 7 firewalld-防火墙
date:       2023-06-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - centos7
    - firewalld
---

# firewalld-防火墙

## 开启开机启动：
```
systemctl enable firewalld.service
```
## 关闭开机启动：
```
systemctl disable firewalld.service
```
## 关闭防火墙：
```
systemctl stop firewalld.service
```
## 开启防火墙
```
systemctl start firewalld.service
```
若遇到无法开启
```
先用：systemctl unmask firewalld.service 
然后：systemctl start firewalld.service
```
## 查看防火墙状态
```
systemctl status firewalld 
# 或
firewall-cmd --stat
```
## 查看已经开放的端口
```
firewall-cmd --list-ports
```
## 开放指定端口号
```
#（--permanent永久生效，没有此参数重启后失效）
#注：可以是一个端口范围，如1000-2000/tcp
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
## 重启防火墙（重新载入，更新配置）
```
firewall-cmd --reload
```
## 验证80端口是否开放
```
firewall-cmd --zone=public --query-port=80/tcp
```
## 移除端口
```
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 
firewall-cmd --permanent --remove-port=123/tcp
```

## 配置firewall-cmd
```
查看版本： firewall-cmd --version

查看帮助： firewall-cmd --help

显示状态： firewall-cmd --state

查看所有打开的端口： firewall-cmd --zone=public --list-ports

更新防火墙规则： firewall-cmd --reload

查看区域信息:  firewall-cmd --get-active-zones

查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0

拒绝所有包：firewall-cmd --panic-on

取消拒绝状态： firewall-cmd --panic-off

查看是否拒绝： firewall-cmd --query-panic

```