---
layout:     post
title:      Spring源码解析之【BeanFactory】与【BeanDefinitionRegistry】
subtitle:   Spring源码解析之【BeanFactory】与【BeanDefinitionRegistry】
date:       2022-03-17
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring源码解析之【BeanFactory】与【BeanDefinitionRegistry】

Spring的一个非常重要的概念的就是Bean，Spring容器需要通过扫描和注册来实现对Bean信息的管理。在分析了Spring的源码之后发现，Spring里面用了很多的BeanFactory，这个类非常重要，因为实际上在Spring中BeanFactory是IOC容器的实际代表者，它用来创建和管理Bean。它为其他具体的IOC容器提供了最基本的规范，它是一个接口类型。

 `BeanFactory`是最典型的工厂方法模式。不同类型的工厂会生产不同类型的商品。它的子接口和对应的实现类结构如下： 

![1647506137913](/img/spring/1647506137913.png)

我们可以看到在`BeanFactory`接口下面有三个子接口： 

- HierarchicalBeanFactory：提供父容器的访问功能
  - ConfigurableBeanFactory 接口

- AutowireCapableBeanFactory：在BeanFactory基础上是实现了对已存在实例的管理

- ListableBeanFactory：提供了批量获取Bean的方法

接口ConfigurableListableBeanFactory继承了上述的三个接口，增加了一些其他的功能：如类加载器、类型转换，属性编辑器，BeanPostProcessor，Bean定义等

实现类DefaultListableBeanFactory继承ConfigurableListableBeanFactory接口，实现了所有的BeanFactory功能，且可以注册BeanDefinition。

#### 1.1 `ListableBeanFactory`接口

 上面的第一个图片没有截全，遗漏了`ListableBeanFactory`的部分实现类信息，且这个实现类还比较重要，因此再增加一个图片。 

![1647506386032](/img/spring/1647506386032.png)

我们可以看到AnnotataionConfigApplicationContext实现了ListableBeanFactory接口。AnnotataionConfigApplicationContext类是注解驱动去扫描Bean的定义信息。这个类我们很熟悉，因为在Spring启动的时候就是利用这个类：

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
```

 在使用`BeanFactory`工厂创建`Bean`的过程中，我们需要利用`BeanDefinitionRegistry`将这些`Bean`注册到`DefaultListableBeanFactory`中。那么接下来就介绍一下`BeanDefinitionRegistry` 。

### 2. `BeanDefinitionRegistry`接口

```java
public interface BeanDefinitionRegistry extends AliasRegistry{
    
}
```

 从上述代码可以看出，`BeanDefinitionRegistry`接口实现了`AliasRegistry`接口，它主要的工作就是定义`Bean`的常规操作。 

它有三个子接口：

- `SimpleBeanDefinitionRegistry`实现类

- `DefaultListableBeanFactory`  实现类

  - 基本实现类 ，拥有`registryBeanDefinition`功能

- `GenericApplicationContext`实现类

![1647506586856](/img/spring/1647506586856.png)

#### 2.1 `BeanDefinitionRegistry`接口的功能

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	// 向beanFactory中注册一个BeanDefinition
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	// 根据beanName从beanFactory中移除一个已经注册的BeanDefinition
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	// 根据beanName从beanFactory中获取一个BeanDefinition
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	// 判断beanFactory中是否有beanName的BeanDefinition
	boolean containsBeanDefinition(String beanName);

	// 获取beanFactory中的所有BeanDefinition的beanName
	String[] getBeanDefinitionNames();

	// 获取beanFactory中的BeanDefinition的数量
	int getBeanDefinitionCount();

	// 判断beanFactory中的beanName是否被占用
	boolean isBeanNameInUse(String beanName);
}
```

#### 2.2 `SimpleBeanDefinitionRegistry`类

 是`BeanDefinitionRegistry`的一个简单实现方式，没有内置的beanFactory，这个类是用来留给我们测试用的。开发情况下Spring不会使用该类。
这个类的功能很简单，这里就不做详细介绍了，根据方法的名称就可以看出对应的方法所实现的功能。 

```java
public class SimpleBeanDefinitionRegistry extends SimpleAliasRegistry implements BeanDefinitionRegistry {

	// 使用ConcurrentHashMap来存储beanName和BeanDefinition
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(64);


	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "'beanName' must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");
		this.beanDefinitionMap.put(beanName, beanDefinition);
	}

	@Override
	public void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		if (this.beanDefinitionMap.remove(beanName) == null) {
			throw new NoSuchBeanDefinitionException(beanName);
		}
	}

	@Override
	public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		BeanDefinition bd = this.beanDefinitionMap.get(beanName);
		if (bd == null) {
			throw new NoSuchBeanDefinitionException(beanName);
		}
		return bd;
	}

	@Override
	public boolean containsBeanDefinition(String beanName) {
		return this.beanDefinitionMap.containsKey(beanName);
	}

	@Override
	public String[] getBeanDefinitionNames() {
		return StringUtils.toStringArray(this.beanDefinitionMap.keySet());
	}

	@Override
	public int getBeanDefinitionCount() {
		return this.beanDefinitionMap.size();
	}

	@Override
	public boolean isBeanNameInUse(String beanName) {
		return isAlias(beanName) || containsBeanDefinition(beanName);
	}
}
```

#### 2.3 `DefaultListableBeanFactory`类

这个类非常重要，它是BeanDefinitionResgitry接口的基本实现，实际上很多beanFactory最后都是调用DefaultListableBeanFactory类的 registerBeanDefinition 来注册beanDefinition.

这个类里面有非常多的方法，这里只介绍比较重要的  registerBeanDefinition 方法的实现。

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
    
    // 以Map的数据结构保存beanName和BeanDefinition
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
    // 保存所有beanDefinition的名字
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
    
    // 注册BeanDefinition的信息
    @Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}
		// 先判断当前的beanName对应的bean信息是否已经存在
		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
            // 是否允许覆盖
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
            // 若允许覆盖  那还得比较下role  如果新进来的这个Bean的role更大	
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				//...
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				//...
			}
			else {
				//...
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// hasBeanCreationStarted：说明已经存在bean已经在创建了
				synchronized (this.beanDefinitionMap) {
                    // 将beanName和beanDefinition放进去
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// 仍然处在Spring启动阶段
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
		else if (isConfigurationFrozen()) {
			clearByTypeCache();
		}
	}    
} 
```

#### 2.4`AnnotationConfigApplicationContext`类

 这个类是`GenericApplicationContext`的子类，但是通过分析这个类的方法，发现并没有`registryBeanDefinition`方法。 

![1647507505235](/img/spring/1647507505235.png)

虽然没有registryBeanDefinition方法，但是又register方法，而register方法又调用了AnnotatedBeanDefinitionReader类的register方法，经过一系列的操作，最后调用了DefaultListableBeanFactory类的registerBeanDefinition 方法。

### 3 总结

1. Spring使用`BeanFactory`作为创建`Bean`的工厂，利用`BeanDefinitionRegistry`来注册`BeanDefinition`，实际上都是使用`DefaultListableBeanFactory`类的`registerBeanDefinition`。


















