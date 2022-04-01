---
layout:     post
title:      Spring源码解析之refresh(3)
subtitle:  【prepareBeanFactory】与【postProcessBeanFactory】
date:       2022-04-01
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---


# Spring源码解析之refresh(3)——【prepareBeanFactory】与【postProcessBeanFactory】

上一篇文章介绍了 `refresh` 方法中的前两个方法，本篇文章我们继续介绍`refresh`方法中的第三个方法`prepareBeanFactory`和第四个方法`postProcessBeanFactory`。

## 一、 `prepareBeanFactory`

这个方法设置`beanFactory`的标准上下文特征，为`beanFactory`设置类加载器、后置处理器等。其中主要注意一下这里面增加了2个`beanFactory`的后置处理器，由于`Bean`和`beanFactory`在Spring中都有后置处理器，所以要优先注意一下。这里需要强调一点：`beanFactory`在想要拥有后置处理器时，必须通过显示的调用`addBeanPostProcessor` 才能获得。

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置beanFactory的类加载器classLoader为当前context的类加载器.
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置SpEL表达式解析器（在Bean初始化完成之后填充属性的时候用到）
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    // 添加属性编辑注册器（注册属性编辑器），属性编辑器实际上是属性的类型转换器
    // 因为bean的属性配置都是字符串类型的，实例化的时候要将这些属性转换为实际类型
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // 添加beanFactory的后置处理器
    // 在Bean初始化之前，调用ApplicationContextAwareProcessor的postProcessBeforeInitialization
    // 处理所有的Aware接口，进行如下操作：
    // 1. 如果bean实现了EnvironmentAware接口，调用bean.setEnvironment方法
    // 2. 如果bean实现了EmbeddedValueResolverAware接口，调用bean.setEmbeddedValueResolver方法
    // 3. 如果bean实现了ResourceLoaderAware接口，调用bean.setResourceLoader方法
    // 4. 如果bean实现了ApplicationEventPublisherAware接口，调用bean.setApplicationEventPublisher方法
    // 5. 如果bean实现了MessageSourceAware接口，调用bean.setMessageSource方法
    // 6. 如果bean实现了ApplicationContextAware接口，调用bean.setApplicationContext方法
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    
    // 忽略自动装配，在注入的时候忽略此方法指定的接口类，也就是指定的接口不会被注入进去
    // 可以忽略的原因是：ApplicationContextAwareProcessor把这五个接口的实现工作做完了
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    // 下面这4个class会被加入到beanFactory的resolvableDependencies字段里面缓存着，
    // 为后面处理依赖注入的时候使用DefaultlistableBeanFactory#resolveDependency方法处理依赖关系
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Register early post-processor for detecting inner beans as ApplicationListeners.
    // 添加beanFactory的后置处理器 事件监听器
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // Register default environment beans.
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

### 1.1 增加`SpEL`语言支持

SpEL表达式语言全称为“Spring Expression Language”.  SpEL是单独模块， 只依赖于core模块，不依赖其他模块，可以单独使用。

SpEL 使用 `#{}`,作为定界符，所有在大括号中的字符串都将被认为是SpEL.

```java
<bean id="rdsDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<property name="url" value="${spring.datasource.druid.url}"/>
 </bean>
```

在`prepareBeanFactory`中方法中主要通过`beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.geBeanclass)); `注册语言解析器，就可以对SpEL进行解析了。

解析的动作是在bean进行初始化的时候会有属性填充的一步，而在这一步中Spring会调用`AbstractAutowireCapableBeanFactory`类的`applyPropertyValue`函数来完成功能。

### 1.2 添加 `ApplicationContextAwareProcessor`处理器

`beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this))` 方法

我们先看一下类 `ApplicationContextAwareProcessor`的结构。

![1648793538011](/img/spring/1648793538011.png)

该类是`BeanPostProcessor`的实现类。我们知道`BeanPostProcessor`类，在bean的init方法执行前后会调用`postProcessBeforeInitialization`和`postProcessAfterinitialization`方法。

因此Spring增加了这个类，就是为了在bean的init方法执行时调用`postProcessBeforeInitialization`方法。那么我们就来具体看看这个方法的实现情况。

#### 1.2.1 `postProcessBeforeInitialization`方法

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor {
	@Override
	@Nullable
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
				bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
				bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)){
			return bean;
		}

		AccessControlContext acc = null;

		if (System.getSecurityManager() != null) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                // 调用这个方法主要是为了调用该类中的invokeAwareInterfaces 方法
				invokeAwareInterfaces(bean);
				return null;
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}
}
```

```java
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```

我们从上面的`invokeAwareInterfaces`方法中可以看出，实现了对应类型的Bean，在初始化之前会根据bean实现的类来获取对应的资源。

执行完上面语句后，`beanFactory`中便有了`ApplicationContextAwareProcessor`后置处理器。我们从下面的图片可以看出，此时`beanFactory`中仅有一个后置处理器。

![1648795928209](/img/spring/1648795928209.png)

### 1.3 设置忽略依赖

当Spring将`ApplicationContextAwareProcessor`注册之后，在`invokeAwareInterfaces`方法中调用的Aware类已经不是普通的bean了。如类已经实现了接口`ApplicationContextAware`时，该类就有`applicationContext`属性。因此当然需要在Spring做bean的依赖注入的时候忽略它们。而`ignoreDependencyInterface`功能便是如此。

```java
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
```

### 1.4 注册依赖

当注册依赖之后，比如当注册了对`BeanFactory`的解析依赖之后，当bean的属性注入的时候，一旦检测到属性为`BeanFactory`类型便会将`BeanFactory`的实例注入进去。

![1648800225299](/img/spring/1648800225299.png)

1.5 事件监听后置处理器`ApplicationListenerDetector`

`beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));`

![1648800779896](/img/spring/1648800779896.png)

## 二、`postProcessBeanFactory`方法

该方法是一个留给子类的空方法。子类通过重写这个方法来在`BeanFactory`创建并预准备完成以后做进一步的设置。

## 三、总结

以上是`refresh`方法中调用的第三个方法`prepareBeanFactory`和第四个方法`postProcessBeanFactory`。































