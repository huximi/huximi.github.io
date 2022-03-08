---
layout:     post
title:      Spring的@ComponentScan注解
subtitle:   @ComponentScan注解
date:       2022-03-07
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

#  @ComponentScan注解

### 1. `@ComponentScan` 注解

在配置类中只要标注了`@ComponentScan` 注解，Spring就可以自动扫描Value对应的包了

#### 1.1. 配置类

```java

import com.huximi.beans.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(value = "com.huximi")
public class MainConfig {

    @Bean("person")
    public Person person() {
        return new Person("huximi", 31);
    }

}

```

 以上扫描com.huximi包中的所有组件 

#### 1.2 单元测试类

```java
import com.huximi.config.MainConfig;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class IOCTest_1 {
    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        // 获取容器中的所有组件
        String[] names = applicationContext.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
    }
}
```

 输出的结果 

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookController
bookDao
bookServiceImpl
person
```

前面五个都是Spring自己的组件，其中mainConfig类是因为增加了@Configuration注解，bookController类是因为增加了@Controller注解，bookService类是因为增加了@Servide注解，bookDao类是因为增加了@Repository注解，pereson类是因为在mainConfig类的person方法中增加了@Bean注解

#### 1.3 `excludeFilters` 排除

```java
import com.huximi.beans.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;

@Configuration
@ComponentScan(value = "com.huximi", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = { Controller.class })
})
public class MainConfig {

    @Bean("person")
    public Person person() {
        return new Person("huximi", 31);
    }

}
```

 根据注解类型进行排除 

#### 1.4 `includeFilters` 仅包含

```java
import com.huximi.beans.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;

@Configuration
@ComponentScan(value = "com.huximi", includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = { Controller.class })
}, useDefaultFilters = false)
public class MainConfig {

    @Bean("person")
    public Person person() {
        return new Person("huximi", 31);
    }

}
```

 使用仅包含`includeFilters`需要禁用默认的过滤规则，即将`useDefaultFilters`设置为false 

#### 1.5 `FilterType.CUSTOM`自定义过滤规则

```java
import org.springframework.core.io.Resource;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

public class MyTypeFilter implements TypeFilter {

    /**
     * @param metadataReader        读取到的当前正在扫描的信息
     * @param metadataReaderFactory 可以获取到其他任何类的信息
     * @return
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) {

        // 获取当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        System.out.println("获取当前类注解的信息：" + annotationMetadata.getAnnotationTypes());

        // 获取到当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();

        // 获取当前类资源（类的路径）
        Resource resource = metadataReader.getResource();
        System.out.println("获取当前类资源（类的路径）：" + resource);

        String className = classMetadata.getClassName();
        System.out.println("类名：" + className);

        if (className.contains("dao")) {
            return true;
        }

        return false;
    }
}
```

##### 1.5.1 自定义过滤，使用自定义的MyTypeFilter.class：

```java
import com.huximi.beans.Person;
import com.huximi.filter.MyTypeFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(value = "com.huximi", includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class }) }, useDefaultFilters = false
)
public class MainConfig {

    @Bean("person")
    public Person person() {
        return new Person("huximi", 31);
    }

}
```

### 总结

1. excludeFilers = Filter[], 指定扫描的时候按照什么规则排除哪些组件
2. includeFilers = Filter[], 指定扫描的时候按照什么规则包含哪些组件
3. FilterType.ANNOTATION， 按照注解
4. FilterType.ASSIGNABLE_TYPE，按照给定的类型
5. FilterType.ASPECTJ，使用ASPECTJ表达式
6. FilterType.REGEX，使用正则规则
7. FilterType.CUSTOM，使用自定义规则
8. @ComponentScan注解还可以叠加使用