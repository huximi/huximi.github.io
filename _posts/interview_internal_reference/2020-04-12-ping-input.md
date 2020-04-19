---
layout: post
title: "ping IP 理解"
subtitle: '输入 ping IP 后敲回车，发包前会发生什么'
author: "GuYang"
header-img: "img/post-bg-2018.jpg"
tags:    
    - 面试题
---

#### 输入 ping IP 后敲回车，发包前会发生什么？


首先根据目的IP和路由表决定走哪个网卡，再根据网卡的子网掩码地址判断目的IP是否在子网内。如果不在则会通过arp缓存查询IP的网卡地址，不存在的话会通过广播询问目的IP的mac地址，得到后就开始发包了，同时mac地址也会被arp缓存起来。
