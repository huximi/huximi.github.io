---
layout:     post
title:      Spring源码解析之this()——ClassPathBeanDefinitionScanner
subtitle:   Spring源码解析之this()——ClassPathBeanDefinitionScanner
date:       2022-03-10
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring源码解析之this()——ClassPathBeanDefinitionScanner

上一篇我们讲到

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;

	public AnnotationConfigApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
}
```

我们将this.reader = new AnnotatedBeanDefinitionReader(this);分析完后，reader得到了Spring的6个内置后置处理器。
那么下面，我们来分析第二步，获取Spring的类扫描器。我们进入ClassPathBeanDefinitionScanner类的构造器中。

### 1. `ClassPathBeanDefinitionScanner`类构造器

ClassPathBeanDefinitionScanner是一个从指定包内扫描所有Bean的Spring工具类。它在给定的包中进行扫描，找到其中标有注解且符合过滤规则的类，然后将这些类定义拿到将其注册到Spring容器中得到Bean组件。

在缺省的情况下，ClassPathBeanDefinitionScanner类只注册带有下面几个注解的类。

- 
  `@Component` 

  - `@Repository`

  - `@Service`

  - `
    @Controller
    `
    - `@RestController`

- `@ManagedBean(Java EE 6)`

- `@Named(JSR-330)`

```java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
    this(registry, true);
}

public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
    this(registry, useDefaultFilters, getOrCreateEnvironment(registry));
}

public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
			Environment environment) {

    this(registry, useDefaultFilters, environment,
         (registry instanceof ResourceLoader ? (ResourceLoader) registry : null));
}

public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
			Environment environment, @Nullable ResourceLoader resourceLoader) {

    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    this.registry = registry;

    if (useDefaultFilters) {
        registerDefaultFilters();
    }
    setEnvironment(environment);
    setResourceLoader(resourceLoader);
}
```

我们可以看到上面一直在调用`ClassPathBeanDefinitionScanner`类的构造器，我们关注后面倒数第三个调用方法`registerDefaultFilters()` 。 

### 2. `registerDefaultFilters()`

通过分析`ClassPathBeanDefinitionScanner`类中的方法可以直到，`ClassPathBeanDefinitionScanner`中并没有该方法，因此找到这个类的类结构。 

![1646891689783](/img/spring/1646891689783.png)

发现它的父类`ClassPathScanningCandidateComponentProvider`类中有`registerDefaultFilters()`方法，我们进入该方法 

```java
public class ClassPathScanningCandidateComponentProvider implements EnvironmentCapable, ResourceLoaderAware {
	protected void registerDefaultFilters() {
        // 增加Component类，即注解过滤器类
        // 在上面我给出了注解的结构层级，因为Controoler、Service、Repotory注解都是Component的子注解，因此他们也可以起作用。
		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
		try {
            // 添加ManagedBean 注解过滤器
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
			logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
		}
		try {
            // 添加Named 注解过滤器
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
			logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-330 API not available - simply skip.
		}
	}
}    
```

 以上运行结束后我们`this()`方法就执行结束了。 

### 3.总结

1. ClassPathBeanDefinitionScanner类的父类的registerDefaultFilters方法主要是给Spring容器中注册了注解过滤器类Component
2. 实际上ClassPathBeanDefinitionScanner类的父类ClassPathScanningCandidateComponentProvider还有一个非常重要的方法：scan()，该方法才是真正进行包扫描的方法，但是因为文章的连续性，程序debug到哪里就叙述到哪里吧！