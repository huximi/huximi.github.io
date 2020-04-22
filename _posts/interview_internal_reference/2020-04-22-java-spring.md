---
layout: post
title: "Spring常见面试题"
subtitle: 'Spring常见面试题'
author: "Jian Shen"
header-style: text
tags:
  - Java
  - Spring
---

### Spring常见面试题

#### 1、使用Spring框架的好处是？

 - **轻量**：Spring是轻量的，基本版本约为2M
 - **控制反转(IOC)**: Spring通过控制反转实现松耦合，对象们给出了依赖，而不是创建与查找依赖的对象们
 - **面向切面(AOP)**：Spring支持AOP编程，把业务逻辑与系统服务分开
 - **容器**：Spring管理应用中对象的生命周期与配置
 - **MVC框架**
 - **事务管理**：Spring提供事务管理接口，上至本地事务，下至全局事务
 - **异常处理**：Spring提供API将具体技术相关的异常转化为一致的unchecked异常
 
#### 2、Spring由哪些模块组成

    spring-core 
    spring-beans
    spring-context
    spring-aop
    spring-expression
    spring-jdbc
    spring-oxm
    spring-tx
    spring-web
    spring-webmvc
    
#### 3、ApplicationContext通常的实现是什么？

    ClassPathXmlApplicationContext
    FileSystemXmlApplicationContext
    WebXmlApplicationContext
        
#### 4、解释Spring支持的几种bean的作用域






