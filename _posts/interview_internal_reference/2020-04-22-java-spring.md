---
layout: post
title: "Spring常见面试题"
subtitle: 'Spring常见面试题'
author: "Jian Shen"
header-img: "img/post-bg-2018.jpg"
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

 - *singleton* 容器中只有一个对应bean实例
 - *prototype* 一个bean的定义有多个实例
 - 以下四种基于web模块下的webApplicationContext
 
        request
        session
        globalSession
        application
    
#### 5、Spring框架中的单例bean是线程安全的吗？

不是

#### 6、解释Spring框架中bean的生命周期

 - Spring容器从XML文件中读取bean的定义，并实例化bean
 - Spring根据bean的定义填充所有属性
 - 如果bean实现了BeanNameAware接口，Spring传递bean的Id到setBeanName方法
 - 如果bean实现了BeanFactoryAware接口，Spring传递beanFactory到setBeanFactory方法
 - 如果bean实现了BeanPostProcessor接口，postProcessorBeforeInitialization方法将被调用
 - 如果bean实现了InitializingBean接口，afterPropertiesSet方法被调用；如果bean声明了初始化方法，则调用该初始化方法
 - 如果bean实现了BeanPostProcessor接口，postProcessorAfterInitialization方法将被调用
 - 如果bean实现了DisposableBean接口，destroy方法将被调用
 
#### 7、Spring团队为什么推荐使用构造器注入，而非变量注入

Java类会先执行构造方法，然后再给注解了@Autowired的对象注入值，如下会报错：

```
@Autowired
private User user;
private String school;

public UserAccountServiceImpl(){
    this.school = user.getSchool();
}

```
改为推荐的注入方式即可：

```
private User user;
private String school;

@Autowired
public UserAccountServiceImpl(User user){
    this.user = user;
    this.school = user.getSchool();
}

```

