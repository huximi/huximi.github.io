---
layout:     post
title:      Spring源码解析之this()——AnnotatedBeanDefinitionReader
subtitle:   Spring源码解析之this()——AnnotatedBeanDefinitionReader
date:       2022-03-09
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring源码解析之this()——AnnotatedBeanDefinitionReader

### 1. 程序入口

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
```

### 2. `AnnotationConfigApplicationContext`类有参构造器

 其中`MainConfig`类是自己写的配置类，我们从这一行代码来开始分析。Spring容器先从这一行代码开始来创建`AnnotationConfigApplicationContext`类型的容器，利用类型构造器来创建对象。下面来看一下这个有参构造器方法执行了哪些操作。 

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
            this();  // 1.
            register(annotatedClasses);  // 2.
            refresh(); // 3.
    }
}
```

第一行：`this();`：调用`AnnotationConfigApplicationContext`的无参构造器
第二行：`register(annotatedClasses)`：对传入的配置类进行预处理与解析
第三行：`refresh()`：容器的创建与刷新 

### 3. `AnnotationConfigApplicationContext`类的`this()`方法

#### 3.1 依次调用父类的构造方法

 我们先看一下`AnnotationConfigApplicationContext`类的类结构 

![1646817547766](1646817547766.png)

由于AnnotationConfigApplicationContext类的父类是GenericApplicationContext类，因此在有参构造器中调用this()时，首先执行父类GenericApplicationContext类的构造器，那么我们先看看GenericApplicationContext类的构造器。

```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {

	 private final DefaultListableBeanFactory beanFactory;
    
 	 public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	 }  
}
```

在这个构造器中，首先初始化一个`DefaultListableBeanFactory`类型的对象,**Spring加载的所有Bean都会放到`DefaultListableBeanFactory`中**。

当然`DefaultListableBeanFactory`类在执行构造方法的时候先调用其父类`AbstractApplicationContext`的构造器，`AbstractApplicationContext`在执行构造方法的时候先调用父类`DefaultResourceLoader`类的构造器。整个顺序就是上面类图的顺序。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
    
    /** ResourcePatternResolver used by this context. */
	private ResourcePatternResolver resourcePatternResolver;
    
	public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
	}
}
```

```java
public class DefaultResourceLoader implements ResourceLoader {
    
	@Nullable
	private ClassLoader classLoader;
    
    public DefaultResourceLoader() {
		this.classLoader = ClassUtils.getDefaultClassLoader();
	}
}
```

#### 3.2 `AnnotationConfigApplicationContext`类的`this()`方法

当父类方法全部调用对应的无参构造器后，我们来返回看`AnnotationConfigApplicationContext`类的`this()`方法。

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    
	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;

    public AnnotationConfigApplicationContext() {
        // BeanDefinition读取器. BeanDefinition是描述bean注册的信息
        this.reader = new AnnotatedBeanDefinitionReader(this);
        // 创建BeanDefinition扫描器
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
}
```

 在`this.reader = new AnnotatedBeanDefinitionReader(this)`中，传入的参数为`this`，而`this`的类型即当前类的类型，即：`AnnotationConfigApplicationContext`类型。 

#### 3.3 `this()` 中调用的`AnnotatedBeanDefinitionReader`构造器

```java
public class AnnotatedBeanDefinitionReader {
    private final BeanDefinitionRegistry registry;
    private ConditionEvaluator conditionEvaluator;
	public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
		this(registry, getOrCreateEnvironment(registry));
	}

	private static Environment getOrCreateEnvironment(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		if (registry instanceof EnvironmentCapable) {
			return ((EnvironmentCapable) registry).getEnvironment();
		}
		return new StandardEnvironment();
	}
    
    public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		Assert.notNull(environment, "Environment must not be null");
		this.registry = registry;
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
        // 注册注解配置的处理器。即项目中我们标注注解的类由下面注册的处理器解析
		// 3.4 分析该方法
		AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
	}
}
```

我们可以看到在3.2中传入的this为AnnotationConfigApplicationContext类型，而上面的方法参数类型就变为BeanDefinitionRegistry类型了。说明AnnotationConfigApplicationContext类为BeanDefinitionRegistry接口的实现类。

![1646818782289](1646818782289.png)

#### 3.4 `AnnotationConfigUtils.registerAnnotationConfigProcessors()`方法

BeanDefinitionHolder是指： Holder for a BeanDefinition with name and aliases. Can be registered as a placeholder for an inner bean. 拥有BeanDefinition的名字和别名，可以为一个内置Bean注册。

```java
public abstract class AnnotationConfigUtils {    
    // 在DefaultListableBeanFactory中注册所有相关的注解后置处理器
    public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
		registerAnnotationConfigProcessors(registry, null);
	}
    
    public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {
		// 【1】获取DefaultListableBeanFactory类型的BeanFactory ，去掉DefaultListableBeanFactory包装
		// 3.4.1介绍
		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
		if (beanFactory != null) {
			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
			}
			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
			}
		}

		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
	   /**
		* 下面注册Spring的内置Bean，主要是根据Bean的类型进行创建。
		*/
		
		// 注册【ConfigurationAnnotationProcessor】类型的Bean 
		// ConfigurationAnnotationProcessor是工厂后置处理器
		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
            // 【2行】 3.4.2介绍
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
		// 注册【AutowiredAnnotationBeanPostProcessor】
		// 处理@Autowired的Bean后置处理器
		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// 注册【CommonAnnotationBeanPostProcessor】
		// 检查是否支持JSR-250。处理一些公共注解的Bean后置处理器，可以处理@PostConstruct、@PreDestroy和@Resource
		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// 检查是否支持JPA, 注册【PersistenceAnnotationBeanPostProcessor】
		// 只有pom中导入了spring-orm后才会注册这个类型的Bean后置处理器
		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				...
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
		// 注册【EventListenerMethodProcessor】
		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}
		// 注册【DefaultEventListenerFactory】
		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
	}
    
}
```

##### 3.4.1 `unwrapDefaultListableBeanFactory`方法

 回溯到3.4.1中的【1】行代码 

```java
@Nullable
private static DefaultListableBeanFactory unwrapDefaultListableBeanFactory(BeanDefinitionRegistry registry) {
    if (registry instanceof DefaultListableBeanFactory) {
        return (DefaultListableBeanFactory) registry;
    }
    else if (registry instanceof GenericApplicationContext) {
        return ((GenericApplicationContext) registry).getDefaultListableBeanFactory();
    }
    else {
        return null;
    }
}
```

如果是DefaultListableBeanFactory类型的直接返回，如果是GenericApplicationContext类型的，通过调用方法获取DefaultListableBeanFactory。因为GenericApplicationContext类中有DefaultListableBeanFactory。

```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {

	 private final DefaultListableBeanFactory beanFactory;
    
 	 public final DefaultListableBeanFactory getDefaultListableBeanFactory() {
		return this.beanFactory;
	 } 
}
```

##### 3.4.2 `registerPostProcessor`方法

 回溯到3.4.1中的【2】行代码 

```java
private static BeanDefinitionHolder registerPostProcessor(
			BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {

    definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    // 将Spring中默认的BeanDefinition注册到Spring容器中
    registry.registerBeanDefinition(beanName, definition);
    return new BeanDefinitionHolder(definition, beanName);
}
```

```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {

	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		this.beanFactory.registerBeanDefinition(beanName, beanDefinition);
	}
}
```

由于上面代码中的beanFactory是DefaultListableBeanFactory，因此调用的是DefaultListableBeanFactory类的registerBeanDefinition方法。下面给出该方法，该方法的内容很多，但是主要在最后，即将Bean存储到registerBeanDefinition和beanDefinitionNames中。

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
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
				...
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				...
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				...
			}
			else {
				...
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// Still in startup registration phase
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

 以上代码执行完成后，整个`this()`方法就执行完了 

### 4. `AnnotatedBeanDefinitionReader`的执行结果

 下面我们看看`AnnotatedBeanDefinitionReader`执行完成后，Spring容器得到了什么。它得到了下面5个Bean后置处理器。 

![1646820518155](1646820518155.png)

我们注意到在3.4中的registerAnnotationConfigProcessors方法的中实际上注册了6个Bean，但是实际上为什么只注册了5个呢？主要是因为PersistenceAnnotationBeanPostProcessor类型的后置处理器需要导入spring-orm依赖（先检查jpaPresent，若不存在jpa，则不会注册PersistenceAnnotationBeanPostProcessor类型的后置处理器 ），而我并没有导入的原因.

 在`AnnotatedBeanDefinitionReader`执行完后，我们返回3.2中，即 

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

 我们看到将得到的Bean后置处理器放到了当前类的`reader`中。以上只是对Spring的6种（这里是5种）内置处理器进行注册。后面在`register`的时候使用了这个`reader`。 

### 5. 补充【`BeanDefinitaion`】

 在Spring源码中会出现很多次BeanDefinition，那么他究竟是什么东西呢？我们现在就来揭秘一下。

![1646821699567](1646821699567.png)

![1646894827557](1646894827557.png)

从上面的类图中我们可以看出，BeanDefinition是一个接口，在Spring中存在三种实现：GenericBeanDefinition、ChildBeanDefinition和RootBeenDefinition。

三种实现均继承自AbstractBeanDefinition，其中BeanDefinition是配置文件<bean>元素标签或者@Bean注解在Spring容器中的内部表示形式。<bean>元素标签或者@Bean注解拥有class、scope、lazy-init等属性，BeanDefinition则提供了相应的beanClass、scope和lazyInit属性。<bean>元素标签或者@Bean注解和BeanDefinition是一一对应的。

Spring通过BeanDefinition将<bean>或者@Bean中的信息转换为容器的内部表示，并将这些BeanDefinition注册到BeanDefinitionRegistry中。Spring容器的BeanDefinitionRegistry(接口)就像是Spring内存数据库，主要是以Map形式存储，后续操作直接从BeanDefinitionRegistry中读取Bean信息，实际上这里的BeanDefinitionRegistry是DefaultListableBeanFactory类型，他们之间的关系如下类图所示：

![1646822036584](1646822036584.png)

### 6. 总结

实际上，我们在执行`this.reader = new AnnotatedBeanDefinitionReader(this);`的时候，只做了一件事情，就是将Spring的内置后置处理器注册到Spring容器中。