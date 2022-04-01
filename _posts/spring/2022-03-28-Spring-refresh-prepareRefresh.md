---
layout:     post
title:      Spring源码解析之refresh(2)
subtitle:  【prepareRefresh】与【ConfigurableListableBeanFactory】
date:       2022-03-28
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring源码解析之refresh(2)——【prepareRefresh】与【ConfigurableListableBeanFactory】

上一篇介绍了refresh方法的概览，其中包含12个方法，接下来我们就依照顺序依次聊一下每个方法所实现的功能以及如何实现的。这篇文章先介绍【prepareRefresh】与【ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()】。

### 1. `prepareRefresh`方法

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
	// ...
    protected void prepareRefresh() {
		// 记录容器启动的时间.
		this.startupDate = System.currentTimeMillis();
        // 设置对应的标志位
		this.closed.set(false);
		this.active.set(true);

		// ...

		// Initialize any placeholder property sources in the context environment.
        // 扩展方法：
		initPropertySources();

		// Validate that all properties marked as required are resolvable:
		// see ConfigurablePropertyResolver#setRequiredProperties
        // 这里我没有用到，所以暂时也没有具体看是做什么的，后续再补充吧！
		getEnvironment().validateRequiredProperties();

		// Store pre-refresh ApplicationListeners...
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}    
    // ...
}
```

#### 1.1 `AbstractRefreshableWebApplicationContext#initPropertySources`

```java
public abstract class AbstractRefreshableWebApplicationContext extends AbstractRefreshableConfigApplicationContext
		implements ConfigurableWebApplicationContext, ThemeSource {
    // ...
	@Override
	protected void initPropertySources() {
		ConfigurableEnvironment env = getEnvironment();
		if (env instanceof ConfigurableWebEnvironment) {
			((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, this.servletConfig);
		}
	}
    // ...
}
```

 在类`AbstractApplicationContext`中该方法是空实现，而因为该工程是MVC工程，因此这里面使用的是调用其子类`AbstractRefreshableApplicationContext`中的方法实现的。（待补充mvc相关知识。） 

### 2. `ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()`

这个方法很重要，从字面上理解是获取BeanFactory。以包扫描的方式使用Spring时，走到该方法的类是：AbstractApplicationContext，从下面的图中可以看出，AbstractApplicationContext实现了BeanFacotory接口。因此它不但包含BeanFacotory接口的所有功能，还在其基础上增加了大量的扩展应用。

![1648519330402](/img/spring/1648519330402.png)

经过方法 `obtainFreshBeanFactory()` 之后，`ApplicationContext`就拥有了`BeanFactory`的全部功能。

实际上该方法的主要功能是：会解析所有Spring的配置文件，这些配置文件我们一般都放在`resources`目录下面。将所有的配置文件中的bean都转换为`BeanDefinition`.

我们有2种方式获取bean信息：

- 方式一：通过配置文件的`<bean>`。

- 方式二：通过配置文件的扫描包的形式：`<context:component-scan base-package="com.suning"></context:component-scan>`

执行完该方法后，主要关注以下两个变量中的值：

- `beanDefinitionName`
  - 存储`beanFactory`中的所有`bean`的`beanName`

- `beanDefinitionMap`
  - 存储`beanFactory`中的所有`bean`的`<beanName,beanDefinition>`

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
 	// ...    
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}
    // ...
}
```

- 【1】初始化`BeanFactory`,并进行xml文件读取，并将得到的`BeanFactory`记录在当前实体的属性中

- 【2】返回当前实体的`beanFactory`属性

#### 2.1 `refreshBeanFactory()`

`AbstractRefreshableApplicationContext#refreshBeanFactory`

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
 	// ...
    @Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
            // 创建 DefaultListableBeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
            // 1.是否覆盖同名称不同定义的对象
            // 2.是否允许bean之间存在循环依赖
			customizeBeanFactory(beanFactory);
            // 初始化DocumentReader，并进行xml文件读取及解析
			loadBeanDefinitions(beanFactory);
			this.beanFactory = beanFactory;
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
    // ...
}
```

```java
protected DefaultListableBeanFactory createBeanFactory() {
    return new DefaultListableBeanFactory(getInternalParentBeanFactory());
}

public DefaultListableBeanFactory(@Nullable BeanFactory parentBeanFactory) {
    super(parentBeanFactory);
}

public AbstractAutowireCapableBeanFactory(@Nullable BeanFactory parentBeanFactory) {
    this();
    setParentBeanFactory(parentBeanFactory);
}

public AbstractAutowireCapableBeanFactory() {
    super();
    ignoreDependencyInterface(BeanNameAware.class);
    ignoreDependencyInterface(BeanFactoryAware.class);
    ignoreDependencyInterface(BeanClassLoaderAware.class);
}
```

##### 2.1.1 `loadBeanDefinitions`

 该步操作是在`XmlWebApplicationContext`类中。 

```java
public class XmlWebApplicationContext extends AbstractRefreshableWebApplicationContext {
	// ...
    @Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
        // 为beanFactory创建 读取xml配置文件的类
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
		initBeanDefinitionReader(beanDefinitionReader);
        // 在执行了创建beanFactory：createBeanFactory(); 和创建xml的读取器后new XmlBeanDefinitionReader(beanFactory);就可以进行配置文件的读取了。
		loadBeanDefinitions(beanDefinitionReader);
	}
    // ...
} 
```

1. 创建读取xml配置文件的读取器

```java
public XmlBeanDefinitionReader(BeanDefinitionRegistry registry) {
    super(registry);
}

protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    this.registry = registry;

    // Determine ResourceLoader to use.
    if (this.registry instanceof ResourceLoader) {
        this.resourceLoader = (ResourceLoader) this.registry;
    }
    else {
        this.resourceLoader = new PathMatchingResourcePatternResolver();
    }

    // Inherit Environment if possible
    if (this.registry instanceof EnvironmentCapable) {
        this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
    }
    else {
        this.environment = new StandardEnvironment();
    }
}
```

我们先看一下`beanFactory`在使用`xmlBeanDefinitionReader`读取xml解析配置之前(即执行`loadBeanDefinition`之前)，`beanFactory`中`beanDefinitionMap`的情况。

2. 读取`xml`配置文件生成`beanDefinition`放到`beanFactory`中

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        for (String configLocation : configLocations) {
            reader.loadBeanDefinitions(configLocation);
        }
    }
}
```

这个配置文件的位置在`web.xml`中被`org.springframework.web.context.ContextLoaderListener`读取到位置，然后放到`ConfigurableWebApplicationContext`中的。

接下来讲解，`XmlBeanDefinitionReader` 是如何读取到xml中的bean信息的

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
    // 1. 获取配置文件的路径，这里是 applicationContext
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        for (String configLocation : configLocations) {
            // 根据xml配置文件的信息来解析
            reader.loadBeanDefinitions(configLocation);
        }
    }
}

@Override
public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
    // 下面会具体讲解如何解析xml的, 见【详解-1】
    return loadBeanDefinitions(location, null);
}
```

 **【详解-1】** 

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
    // 获取资源加载器，因为得到了资源的位置，需要利用加载器将该资源加载到内存中
    // 当前资源加载器的类型为：XmlWebApplicationContext，该类型属于ResourcePatternResolver
    ResourceLoader resourceLoader = getResourceLoader();
    if (resourceLoader == null) {
        throw new BeanDefinitionStoreException(
            "Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
    }

    if (resourceLoader instanceof ResourcePatternResolver) {
        // Resource pattern matching available.
        try {
            Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
            int count = loadBeanDefinitions(resources);
            if (actualResources != null) {
                Collections.addAll(actualResources, resources);
            }
            if (logger.isTraceEnabled()) {
                logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
            }
            return count;
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException(
                "Could not resolve bean definition resource pattern [" + location + "]", ex);
        }
    }
    else {
        // Can only load single resources by absolute URL.
        Resource resource = resourceLoader.getResource(location);
        int count = loadBeanDefinitions(resource);
        if (actualResources != null) {
            actualResources.add(resource);
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
        }
        return count;
    }
}
```

















