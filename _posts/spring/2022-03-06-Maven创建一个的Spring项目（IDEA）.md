---
layout:     post
title:      Maven创建一个的Spring项目（IDEA）
subtitle:   Maven创建一个的Spring项目（IDEA）
date:       2022-03-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Maven创建一个的Spring项目（IDEA）

准备工作：首先电脑里需要有IDEA和Maven。

### 1. 创建maven工程

首先打开IDEA，然后开始new一个新的project，具体如下

1. 选中：Maven

2. 勾选：Create from archetype

3. 选择：org.apache.maven.archetypes:maven-archetype-quickstart

4. 下一步后，输入GroupId、ArtifactId、Version

   ```
   GroupId: com.huximi.annotation
   ArtifactId: springAnnotation
   Version: 1.0-SNAPSHOT
   ```

5. 下一步后，选择maven版本和配置

6. 下一步后，输入项目名和项目地址

7. 结束

 以上一个Maven的Spring项目就创建出来了

  `com.huximi.annotation`包下只有一个APP.java文件是Spring的入口文件 

### 2. 导入Spring依赖

在pom.xml中的 dependencies 节点下添加Spring依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.12.RELEASE</version>
</dependency>
```

### 3. 小Demo

#### 3.1 POJO类

```java
package com.huximi.beans;

public class Person {
    private String name;
    private Integer age;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public Person() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### 3.2 `MainConfig`配置类

 原来是配置文件，现在是注解的方式所以使用的是配置类，只需要在配置类前面增加注解`@Configuration` 

```java
package com.huximi.config;

import com.huximi.beans.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MainConfig {

    @Bean("person")
    public Person person() {
        return new Person("huximi",31);
    }
}
```

#### 3.3 `App`入口类

```java
package com.huximi.annotation;

import com.huximi.beans.Person;
import com.huximi.config.MainConfig;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main( String[] args ) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean(Person.class);
        System.out.println(person);
    }
}
```

### 4. 总结

1. `@Configuration`：告诉Spring这是一个配置类
2. `@Bean`：给容器中注册一个Bean，类型为返回值的类型，id默认为方法名