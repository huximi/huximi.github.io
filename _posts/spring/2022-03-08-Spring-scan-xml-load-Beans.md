---
layout:     post
title:      Spring解析xml并加载Beans
subtitle:   Spring解析xml并加载Beans
date:       2022-03-08
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - Spring  
---

# Spring解析xml并加载Beans

##  一、Spring之AnnotationConfigApplicationContext的2种构造方法

### 1、App类- 第一种方式

```java
public class App
{
    public static void main( String[] args ) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);

        String[] names = applicationContext.getBeanNamesForType(Person.class);
        for(String a: names)
            System.out.println(a);

    }
}
```

上面这个例子是向AnnotationConfigApplicationContext构造函数中传入一个注解类来注册组件的。以上是第一种方式ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class); -> 即通过传入一个或者多个注解类来注册组件，传入的类会作为组件被spring容器管理。

### 2、App类- 第二种方式

```java
public class App
{
    public static void main( String[] args ) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.huximi");
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);

        String[] names = applicationContext.getBeanNamesForType(Person.class);
        for(String a: names)
            System.out.println(a);

    }
}
```

第二种方式是：ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.huximi"); -> 传入一个包名，该包下的所有带有注解的类都会创建bean，这些Bean会被Spring容器管理

## 二、Spring之ApplicationContext配置文件的方式

 通常，我们有2种方式来利用Spring， 第一种方式是使用配置的方式，第二种方式是：注解的方式。 

- 配置方式-1

```java
BeanFactory bf = new XmlBeanFactory(new ClasswPthResource("applicationContext.xml"));
```

- 配置方式-2

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
```

- 包扫描注解的方式

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.huximi");
```

本篇文章主要讲解配置方式-2的实现方式。

在讲解之前，有必要说明一下配置方式的两种区别，实际上就是 BeanFactory 和 ApplicationContext 的区别。 **两者都是用来加载 bean 的，但ApplicationContext 提供了更多的功能。即ApplicationContext 包含了BeanFactory 的所有功能。实际上从继承关系也可以看出来。**

 我们从解析`xml`作为切入点，来对`Spring`的整体进行分析 

###  `ClassPathXmlApplicationContext`类

```java
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
	this(new String[] {configLocation}, true, null);
}
```

我们可以将配置文件路径以数组的方式传递进去，因为他也提供下面这个方法。

```java
public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
	this(configLocations, true, null);
}

public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {		
    super(parent);
    // 设置配置路径
    setConfigLocations(configLocations);
    if (refresh) {
        // 解析和加载
        refresh();
    }
}
```



## 三、Spring之解析xml文件与BeanDefinition

 Spring中有2个很重要的类，在分析Spring源码之前，我们先来了解几个Spring中的核心类。 

#### 1、`DefaultListableBeanFactory`

DefaultListableBeanFactory是整个Bean 加载的核心部分，是Spring注册和加载 bean的默认实现

#### 2、`XmlBeanDefinitionReader`  XML读取

- EnvironmentCapable
  - 定于获取 Environment方法
- BeanDefinitionReader
  - 主要定义资源文件读取并转换为BeanDefinition 的各个功能
- AbstractBeanDefinitionReader
  - 对EnvironmentCapable、BeanDefinitionReader类定义的功能实现

#### 3、**xml 配置文件的读取流程** 

【1】通过 继承自 AbstractBeanDefinitionReader 中的方法，来使用 ResoureLoader将资源文件路径转换为对应的 Resource 文件。

【2】通过 DocumentLoader 对 Resource 文件进行转换，将 Resource文件转换为 Document 文件。

【3】通过实现接口 BeanDefinitionDocumentReader 的 DefaultBeanDefinitionDocumentReader 类对 Document 进行解析，并使用 BeanDefinitionParserDelegate 对 Element 进行解析。

 以上配置文件的读取过程是单纯使用Spring的方式去加载web.xml文件。如果我们使用spring-mvc的方式去读取配置文件，那么我们的调用过程如下。 

![](/img/spring/spring-xml-load.png)

在讲解 Spring功能扩展时，我会详细讲解Spring是如何找到的配置文件。这里继续分析他是如何解析配置文件applicationContext.xml的。

本文使用的是Spring-mvc的方式启动应用程序。因此这里解析配置文件时，使用的是AbstractBeanDefinitionReader类

##### 3.1、AbstractBeanDefinitionReader

```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader, EnvironmentCapable {
    ......
    public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		// 解析location文件路径			
		Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
		int count = loadBeanDefinitions(resources);					
		return count;					
    }
    @Override
	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {		
		int count = 0;
        // 一个一个解析资源文件
		for (Resource resource : resources) {
            // 这个方法是调用子类XmlBeanDefinitionReader的方法
			count += loadBeanDefinitions(resource);
		}
		return count;
	}
    ......
}
```

数据准备阶段的逻辑

【步骤一】
通过属性来记录已经加载的资源
【步骤二】
对传入的 resource 参数做封装
【步骤三】
通过 SAX 读取 XML 文件的方式来准备 InputSource对象
【步骤四】
最后将准备的数据通过参数传入真正的核心处理部分

##### 3.2、XmlBeanDefinitionReader

```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
	......
    @Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException 	  {
        // 这一步是对 resource 资源文件使用EncodedResource类进行封装。
		return loadBeanDefinitions(new EncodedResource(resource));
	}
    public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
        // 通过属性来记录已经加载的资源
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		// 从encodeResource中获取已经封装的Resource对象并再次从Rescource中获取其中的inputStream
        try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
			InputSource inputSource = new InputSource(inputStream);
			if (encodedResource.getEncoding() != null) {
				inputSource.setEncoding(encodedResource.getEncoding());
			}
            // 真正进入了逻辑核心部分
			return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
		}
		catch (IOException ex) {
            ......
		}
		finally {
			......
		}
	}    
    
    protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
             // 加载 XML 文件，并得到对应的 Document
			Document doc = doLoadDocument(inputSource, resource);
            // 根据返回的 Document 注册 bean 信息
			int count = registerBeanDefinitions(doc, resource);			
			return count;
		}
        ......
    }
    
    protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
        // 加载 XML 文件，并得到对应的 Document
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
                // 获取对 XML 文件的验证模式
				getValidationModeForResource(resource), isNamespaceAware());
	}   
    ......        
}
```

```java
public class EncodedResource implements InputStreamSource {
    public EncodedResource(Resource resource) {
		this(resource, null, null);
    }
    private EncodedResource(Resource resource, @Nullable String encoding, @Nullable Charset charset) {
		super();
		Assert.notNull(resource, "Resource must not be null");
		this.resource = resource;
		this.encoding = encoding;
		this.charset = charset;
	}
}
```

##### 3.3、`DefaultBeanDefinitionDocumentReader` 

```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
    ......
    public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
            BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
            int countBefore = getRegistry().getBeanDefinitionCount();
            // 重点
            documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
            return getRegistry().getBeanDefinitionCount() - countBefore;
    }	
    ......
}
```

- 使用 `DefaultBeanDefinitionDocumentReader` 实例化 `BeanDefinitionDocumentReader`
- 记录统计前 BeanDefinition 的加载个数
- 加载及注册Bean
- 记录本次加载的 BeanDefinition 个数

```java
public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {
	@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
        // 核心
		doRegisterBeanDefinitions(doc.getDocumentElement());
	}
    
    protected void doRegisterBeanDefinitions(Element root) {	
        // 专门处理解析
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);
		......
        // 解析前处理，留给子类实现
		preProcessXml(root);
        // 核心
		parseBeanDefinitions(root, this.delegate);
        // 解析后处理，留给子类实现
		postProcessXml(root);

		this.delegate = parent;
	}    
    
    protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
        // 对beans的处理
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);
					}
					else {
                        // Spring中使用包扫描的方式时，走这个方法
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
}
```
我们可以看到，parseBeanDefinitions 方法中有两种解析方法。因为Spring 的 XML配置里面有2大类 Bean 声明。一种是默认的，另一种是自定义的。

例子：

- 默认
  ```<bean id="test" class="com.test"/>```

- 自定义
  ```<context:component-scan base-package="com.huximi"/>```
  判断是否默认命名空间主要是使用 delegate.isDefaultNamespace(ele) 方法。
##### 3.4、`BeanDefinitionParserDelegate`

```java
public class BeanDefinitionParserDelegate {

	public static final String BEANS_NAMESPACE_URI = "http://www.springframework.org/schema/beans";

public boolean isDefaultNamespace(Node node) {
		return isDefaultNamespace(getNamespaceURI(node));
	}

	public boolean isDefaultNamespace(@Nullable String namespaceUri) {
		return !StringUtils.hasLength(namespaceUri) || BEANS_NAMESPACE_URI.equals(namespaceUri);
	}
}
```

 与Spring 中固定的命名空间`"http://www.springframework.org/schema/beans"` 进行比对，如果一致就认为是默认，否则就认为是自定义。 

**自定义标签解析**

```
org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseCustomElement
```

```java
public class BeanDefinitionParserDelegate {
    public BeanDefinition parseCustomElement(Element ele) {
        return parseCustomElement(ele, null);
    }
    
    @Nullable
	public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
        // 获取命名空间
		String namespaceUri = getNamespaceURI(ele);	
        // 根据命名空间找到对应的 NamespaceHandler
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);		
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}
}
```

 根据对应的bean获取对应的命名空间，根据命名空间解析对应的处理器，然后根据用户自定义的处理器进行解析。 

标签的解析是从命名空间的获取开始的，无论是区分 Spring 中默认标签和自定义标签还是区分自定义标签中不同标签的处理器都是以标签所提供的命名空间为基础。

`String namespaceUri = getNamespaceURI(ele);`中提供获取标签的命名空间的方法。

```java
@Nullable
public String getNamespaceURI(Node node) {
    return node.getNamespaceURI();
}
```

 获取命名空间的动作是由`org.w3c.dom`提供的。我们只需要直接调用即可。 

 有了命名空间，就可以进行 `NamespaceHandler` 的提取了。 

##### 3.5、`DefaultNamespaceHandlerResolver`

```java
public class DefaultNamespaceHandlerResolver implements NamespaceHandlerResolver {
	@Override
	@Nullable
	public NamespaceHandler resolve(String namespaceUri) {
        // 获取所有已经配置的 handler 映射
		Map<String, Object> handlerMappings = getHandlerMappings();
        // 根据命名空间找到对应的信息
		Object handlerOrClassName = handlerMappings.get(namespaceUri);
		if (handlerOrClassName == null) {
			return null;
		}
		else if (handlerOrClassName instanceof NamespaceHandler) {
            // 已经做过解析的情况，直接从缓存读取
			return (NamespaceHandler) handlerOrClassName;
		}
		else {
            // 没有做过解析，则返回的是类路径
			String className = (String) handlerOrClassName;
			try {
                // 使用反射，将类路径转化为类
				Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
				// 初始化类
				NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
                // 调用自定义的 NamespaceHandler 的初始化方法
				namespaceHandler.init();
                // 记录在缓存
				handlerMappings.put(namespaceUri, namespaceHandler);
				return namespaceHandler;
			}
			......
		}
	}
}
```

 得到了解析器以及要分析的元素后，Spring就可以将解析工作委托给自定义解析器去解析了。 

##### 3.6、`ClassPathBeanDefinitionScanner`包扫描

在IOC 容器初始化阶段（调用beanFactoryPostProcessor阶段），会采用ClassPathBeanDefinitionScanner进行扫描包(<context:component-scan base-package="com.huximi"/>)下 所有类，扫描时有过滤条件，将符合过滤条件的类的class信息包装为 BeanDefinition 然后注册到IOC 容器内

###### 3.6.1、NamespaceHandlerSupport

```java
public abstract class NamespaceHandlerSupport implements NamespaceHandler {
	@Override
	@Nullable
	public BeanDefinition parse(Element element, ParserContext parserContext) {
        // 寻找解析器并进行解析操作
		BeanDefinitionParser parser = findParserForElement(element, parserContext);
		return (parser != null ? parser.parse(element, parserContext) : null);
	}
    
    @Nullable
	private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
		String localName = parserContext.getDelegate().getLocalName(element);
		BeanDefinitionParser parser = this.parsers.get(localName);		
		return parser;
	}
}
```

###### 3.6.2、ComponentScanBeanDefinitionParser

```java
public class ComponentScanBeanDefinitionParser implements BeanDefinitionParser {
    private static final String BASE_PACKAGE_ATTRIBUTE = "base-package";
	@Override
	@Nullable
	public BeanDefinition parse(Element element, ParserContext parserContext) {
        // 获取  <context:component-scan base-package="com.huximi"/> 中的base-package值
		String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
		basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
		String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
				ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

		// Actually scan for bean definitions and register them.
        // 扫描
		ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
        // 扫描base-package值对应的包中的 Bean 
		Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
        // 注册扫描到的bean
		registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

		return null;
	}
}
```

###### 3.6.3、ClassPathBeanDefinitionScanner

```java
public class ClassPathScanningCandidateComponentProvider implements EnvironmentCapable, ResourceLoaderAware {

	static final String DEFAULT_RESOURCE_PATTERN = "**/*.class";
    
	protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
		for (String basePackage : basePackages) {
            // 核心
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			for (BeanDefinition candidate : candidates) {
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				candidate.setScope(scopeMetadata.getScopeName());
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
				if (candidate instanceof AnnotatedBeanDefinition) {
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
				if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					definitionHolder =
							AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
                    // 注册bean
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}


	public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
			return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
		}
		else {
			return scanCandidateComponents(basePackage);
		}
	}
    
    private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
		Set<BeanDefinition> candidates = new LinkedHashSet<>();
		try {
            // 1.根据指定包名 生成包搜索路径
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + '/' + this.resourcePattern;
			//2. 资源加载器 加载搜索路径下的 所有class 转换为 Resource[]
            Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
            // 3. 循环 处理每一个 resource 
			for (Resource resource : resources) {
				...
				if (resource.isReadable()) {
					try {
                        // 读取类的 注解信息 和 类信息 ，信息储存到  MetadataReader
						MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
                        // 执行判断是否符合 过滤器规则，函数内部用过滤器 对metadataReader 过滤  
						if (isCandidateComponent(metadataReader)) {
                            //把符合条件的 类转换成 BeanDefinition
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							sbd.setSource(resource);
                            // 再次判断 如果是实体类 返回true,如果是抽象类，但是抽象方法 被 @Lookup 注解注释返回true 
							if (isCandidateComponent(sbd)) {
								...
								candidates.add(sbd);
							}
							else {
								...
							}
						}
						else {
							...
						}
					}
					catch (Throwable ex) {
						...
					}
				}
				else {
					...
				}
			}
		}
		catch (IOException ex) {
			...
		}
		return candidates;
	}
    
    protected void registerBeanDefinition(BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry) {
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, registry);
	}
}
```

###### 3.6.4、BeanDefinitionReaderUtils

```java
public abstract class BeanDefinitionReaderUtils {
	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
        // 核心，这里面会将对应的bean 放到beanFactory 的 beanDefinitionMap
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

###### 3.6.5、DefaultListableBeanFactory

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
				...
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
                // 核心
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

上面代码中，主要关注代码片段：this.beanDefinitionMap.put(beanName, beanDefinition); 和 this.beanDefinitionNames.add(beanName);。这段代码的意思是将xml中的所有bean信息放到 beanDefinitionMap 和 beanDefinitionNames 中。

 执行完以上步骤，所有的bean都会存储在beanFactory中。 

####  四、总结

本篇文章主要讲解的是如何将 `applicationContext.xml` 中的`bean` 转换为 `BeanDefinition`，并将其存储在`beanFactory`中。







