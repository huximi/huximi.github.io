---
layout:     post
title:      Spring源码解析之register——register(annotatedClasses)注册配置类
subtitle:   Spring源码解析之register——register(annotatedClasses)注册配置类
date:       2022-03-11
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring源码解析之register——register(annotatedClasses)注册配置类

 上一篇我们讲到执行完`this()`方法 

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
        	// 1. 首先初始化Spring的6个内置Bean后置处理器，并放到 DefaultListableBeanFactory 类型的对象 beanFactory 中
			// 2. 创建Spring的注解解析器 Component
            this(); 
        	// 传入的配置类annotatedClasses，生成BeanDefinition，然后将BeanDefinition注册到DefaultListableBeanFactory 类型的对象 beanFactory 中
            register(annotatedClasses);
            refresh();
    }
}
```

### 1. `AnnotationConfigApplicationContext`类的`register()`方法

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;
    @Override
    public void register(Class<?>... componentClasses) {
        Assert.notEmpty(componentClasses, "At least one component class must be specified");
        this.reader.register(componentClasses);
    }
}
```

 从上述代码的参数中可以看出该方法可以传入多个配置类。
该方法的功能：注册一个或者多个带有注解的配置类 

```java
public class AnnotatedBeanDefinitionReader {
	public void register(Class<?>... componentClasses) {
		for (Class<?> componentClass : componentClasses) {
			registerBean(componentClass);
		}
	}

	public void registerBean(Class<?> beanClass) {
		doRegisterBean(beanClass, null, null, null, null);
	}
}
```

 下面进入了关键的代码段： 

```java
// 根据给定的class来注册一个bean
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {
	// 根据传入的annotatedClass生成AnnotatedGenericBeanDefinition。AnnotatedGenericBeanDefinition是BeanDefinition的一个实现类。类图在4.补充图片中的图1中。
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
    // 根据 @cONditional注解来判断是否要跳过解析
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    abd.setInstanceSupplier(supplier);
    // 获取类的作用域，默认为Singleton
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    // 将类的作用域添加到AnnotatedGenericBeanDefinition的数据结构中
    abd.setScope(scopeMetadata.getScopeName());
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
	// 【1】1.1中补充介绍
    // 处理配置类中的通用注解，即Lazy、DependsOn、Primary和Role等，将处理的结果放到AnnotatedGenericBeanDefinition的数据结构中
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    // 依次判断了注解当中是否包含了Primary、Lazy、qualifier，如果包含就设置AnnotatedGenericBeanDefinition对应的属性
    if (qualifiers != null) {
        for (Class<? extends Annotation> qualifier : qualifiers) {
            if (Primary.class == qualifier) {
                abd.setPrimary(true);
            }
            else if (Lazy.class == qualifier) {
                abd.setLazyInit(true);
            }
            else {
                abd.addQualifier(new AutowireCandidateQualifier(qualifier));
            }
        }
    }
    if (customizers != null) {
        for (BeanDefinitionCustomizer customizer : customizers) {
            customizer.customize(abd);
        }
    }
	// 将AnnotatedGenericBeanDefinition和他对应的beanName存储到BeanDefinitionHolder中的对应属性中
    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    // 将 BeanDefinitionHolder 注册给 this.registry 即 AnnotationConfigApplicationContext。
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```

#### 1.1 `processCommonDefinitionAnnotations`方法

```java
public abstract class AnnotationConfigUtils {
	public static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd) {
		processCommonDefinitionAnnotations(abd, abd.getMetadata());
	}

    // 检查通用的注解，将存在的注解添加到AnnotatedBeanDefinition中
	static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd, AnnotatedTypeMetadata metadata) {
		AnnotationAttributes lazy = attributesFor(metadata, Lazy.class);
		// 判断Lazy
        if (lazy != null) {
			abd.setLazyInit(lazy.getBoolean("value"));
		}
        // 判断metadata 和 Lazy
		else if (abd.getMetadata() != metadata) {
			lazy = attributesFor(abd.getMetadata(), Lazy.class);
			if (lazy != null) {
				abd.setLazyInit(lazy.getBoolean("value"));
			}
		}
		// 判断Primary
		if (metadata.isAnnotated(Primary.class.getName())) {
			abd.setPrimary(true);
		}
        // 判断DependsOn
		AnnotationAttributes dependsOn = attributesFor(metadata, DependsOn.class);
		if (dependsOn != null) {
			abd.setDependsOn(dependsOn.getStringArray("value"));
		}

		AnnotationAttributes role = attributesFor(metadata, Role.class);
		if (role != null) {
			abd.setRole(role.getNumber("value").intValue());
		}
		AnnotationAttributes description = attributesFor(metadata, Description.class);
		if (description != null) {
			abd.setDescription(description.getString("value"));
		}
	}

}
```

### 2. `registerBeanDefinition`方法

```java
public abstract class BeanDefinitionReaderUtils {
	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
        // 获取到的是配置类的BeanDefinition
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
}    
```

### 3. `register`的执行结果

执行完以上的步骤后，`register`方法就执行完成了，执行完成后，Spring容器的`beanFactory`中除了有在`this()`构造方法中增加的6个（目前为5个，因为没有引入`jpa`依赖）还增加了配置类的`Bean` 。

![1646894090138](/img/spring/1646894090138.png)

其中最后一个`mainConfig`就是我们传入的配置类。可以追溯到最初的代码： 

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
```

### 4. 补充图片

1.  图1 


![1646894827557](/img/spring/1646894827557.png)

### 5. 总结

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
        // 1. 首先初始化Spring的6个内置Bean后置处理器，并放到 DefaultListableBeanFactory 类型的对象 beanFactory 中
        // 2. 创建Spring的注解解析器 Component
        this(); 
        // 传入的配置类annotatedClasses，生成BeanDefinition，然后将BeanDefinition注册到DefaultListableBeanFactory 类型的对象 beanFactory 中
        register(annotatedClasses);
        refresh();
    }
}
```

1. 在执行完`register(annotatedClasses);`方法后，我们传入的配置类信息就会保存到Spring容器的`beanFactory`中。至此，Spring容器中包含了6个类信息。（5个内置后置处理器类，一个配置类）。