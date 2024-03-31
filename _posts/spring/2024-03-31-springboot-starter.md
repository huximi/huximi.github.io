---
layout:     post
title:      SpringBoot的Starter解析
subtitle:   天天用SpringBoot写代码，结果连Starter是什么都不知道？
date:       2024-03-31
author:     苏三呀
header-img: "img/post-bg-2018.jpg"
tags:    
- Spring
- SpringBoot
- Starter
---

## 前言
我们都知道，`Spring`的功能非常强大，但也有些弊端。比如：我们需要手动去配置大量的参数，没有默认值，需要我们管理大量的`jar`包和它们的依赖。
为了提升`Spring`项目的开发效率，简化一些配置，`Spring`官方引入了`SpringBoot`。

当然，引入`SpringBoot`还有其他原因，在这里就不过多描述了。

本文重点跟大家一起聊聊`SpringBoot`的`starter`机制，因为它太重要了。

![img.png](/img/spring/spring-boot-starter/img.png)

## 1 为什么要用starter？

在`SpringBoot`还没有出来之前，我们使用`Spring`开发项目。如果程序需要连接数据库，我们一般会使用`Hibernate`或`Mybatis`等`ORM`框架，这里我以`Mybatis`为例，具体的操作步骤如下：

 1. 到`maven`仓库去找需要引入的`mybatis jar`包，选取合适的版本。
 2. 到`maven`仓库去找`mybatis-spring`整合的`jar`包，选取合适的版本。
 3. 在`spring`的`applicationContext.xml`文件中配置`dataSource`和`mybatis`相关信息。

当然有些朋友可能会指正，不是还需要引入数据库驱动包吗？

确实需要引入，但数据库驱动有很多，比如：`mysql`、`oracle`、`sqlserver`，这不属于`mybatis`的范畴，使用者可以根据项目的实际情况单独引入。

如果程序只是需要连接数据库这一个功能还好，按上面的步骤做基本可以满足需求。但是，连接数据库可能只是庞大的项目体系中一个环节，实际项目中往往更复杂，需要引入更多的功能，比如：连接`redis`、连接`mongodb`、使用`rocketmq`、使用`excel`功能等等。

引入这些功能的话，需要再把上面的步骤再重复一次，工作量无形当中增加了不少，而且有很多重复的工作。

另外，还是有个问题，每次到要到maven中找合适的版本，如果哪次找的`mybatis.jar`包 和 `mybatis-spring.jar`包版本不兼容，程序不是会出现问题？

`SpringBoot`为了解决以上两个问题引入了`starter`机制。

## 2 starter有哪些要素？

我们首先一起看看`mybatis-spring-boot-starter.jar`是如何定义的。

![img_1.png](/img/spring/spring-boot-starter/img_1.png)

可以看到它的`META-INF`目录下只包含了：
 - `pom.protperties`  配置`maven`所需的项目`version`、`groupId`和`artifactId`。
 - `pom.xml`  配置所依赖的`jar`包。
 - `MANIFEST.MF` 这个文件描述了该`Jar`文件的很多信息。
 - `spring.provides` 配置所依赖的`artifactId`，给`IDE`使用的，没有其他的作用。

注意一下，没有一行代码。

我们重点看一下`pom.xml`，因为这个`jar`包里面除了这个没有啥重要的信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot</artifactId>
    <version>1.3.1</version>
  </parent>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <name>mybatis-spring-boot-starter</name>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
</project>
```
从上面可以看出，`pom.xml`文件中会引入一些`jar`包，其中除了引入`spring-boot-starter`，之外重点看一下：`mybatis-spring-boot-autoconfigure`。

我们找到`mybatis-spring-boot-autoconfigure.jar`文件，打开这个文件。

![img_2.png](/img/spring/spring-boot-starter/img_2.png)

里面包含如下文件：
 - `pom.properties`  配置`maven`所需的项目`version`、`groupId`和`artifactId`
 - `pom.xml`  配置所依赖的`jar`包
 - `additional-spring-configuration-metadata.json`  手动添加`IDE`提示功能
 - `MANIFEST.MF` 这个文件描述了该`Jar`文件的很多信息
 - `spring.factories` `SPI`会读取的文件
 - `spring-configuration-metadata.json` 系统自动生成的`IDE`提示功能
 - `ConfigurationCustomizer` 自定义`Configuration`回调接口
 - `MybatisAutoConfiguration` `mybatis`配置类
 - `MybatisProperties` `mybatis`属性类
 - `SpringBootVFS` 扫描嵌套的`jar`包中的类

`spring-configuration-metadata.json`和`additional-spring-configuration-metadata.json`的功能差不多，我们在`applicationContext.properties`文件中输入`spring`时，会自动出现下面的配置信息可供选择，就是这个功能了。

![img_3.png](/img/spring/spring-boot-starter/img_3.png)

来自灵魂的一问：这两个文件有什么区别?

答：如果`pom.xml`中引入了`spring-boot-configuration-processor`包，则会自动生成`spring-configuration-metadata.json`。

如果需要手动修改里面的**元数据**，则可以在`additional-spring-configuration-metadata.json`中编辑，最终两个文件中的元数据会合并到一起。

`MybatisProperties`类是属性实体类：

```java
@ConfigurationProperties(prefix = MybatisProperties.MYBATIS_PREFIX)
public class MybatisProperties {

    public static final String MYBATIS_PREFIX = "mybatis";
    private String configLocation;
    private String[] mapperLocations;
    private String typeAliasesPackage;
    private String typeHandlersPackage;
    private boolean checkConfigLocation = false;
    private ExecutorType executorType;
    private Properties configurationProperties;
    @NestedConfigurationProperty
    private Configuration configuration;

    public String getConfigLocation() {
        return this.configLocation;
    }

    public void setConfigLocation(String configLocation) {
        this.configLocation = configLocation;
    }

    @Deprecated
    public String getConfig() {
        return this.configLocation;
    }

    @Deprecated
    public void setConfig(String config) {
        this.configLocation = config;
    }

    public String[] getMapperLocations() {
        return this.mapperLocations;
    }

    public void setMapperLocations(String[] mapperLocations) {
        this.mapperLocations = mapperLocations;
    }

    public String getTypeHandlersPackage() {
        return this.typeHandlersPackage;
    }

    public void setTypeHandlersPackage(String typeHandlersPackage) {
        this.typeHandlersPackage = typeHandlersPackage;
    }

    public String getTypeAliasesPackage() {
        return this.typeAliasesPackage;
    }

    public void setTypeAliasesPackage(String typeAliasesPackage) {
        this.typeAliasesPackage = typeAliasesPackage;
    }

    public boolean isCheckConfigLocation() {
        return this.checkConfigLocation;
    }

    public void setCheckConfigLocation(boolean checkConfigLocation) {
        this.checkConfigLocation = checkConfigLocation;
    }

    public ExecutorType getExecutorType() {
        return this.executorType;
    }

    public void setExecutorType(ExecutorType executorType) {
        this.executorType = executorType;
    }

    public Properties getConfigurationProperties() {
        return configurationProperties;
    }

    public void setConfigurationProperties(Properties configurationProperties) {
        this.configurationProperties = configurationProperties;
    }

    public Configuration getConfiguration() {
        return configuration;
    }

    public void setConfiguration(Configuration configuration) {
        this.configuration = configuration;
    }

    public Resource[] resolveMapperLocations() {
        ResourcePatternResolver resourceResolver = new PathMatchingResourcePatternResolver();
        List<Resource> resources = new ArrayList<Resource>();
        if (this.mapperLocations != null) {
            for (String mapperLocation : this.mapperLocations) {
                try {
                    Resource[] mappers = resourceResolver.getResources(mapperLocation);
                    resources.addAll(Arrays.asList(mappers));
                } catch (IOException e) {
                    // ignore
                }
            }
        }
        return resources.toArray(new Resource[resources.size()]);
    }
}
```
可以看到`Mybatis`初始化所需要的很多属性都在这里，相当于一个`JavaBean`。

下面重点看一下`MybatisAutoConfiguration`的代码：

```java
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnBean(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration {

    private static final Logger logger = LoggerFactory.getLogger(MybatisAutoConfiguration.class);
    private final MybatisProperties properties;
    private final Interceptor[] interceptors;
    private final ResourceLoader resourceLoader;
    private final DatabaseIdProvider databaseIdProvider;
    private final List<ConfigurationCustomizer> configurationCustomizers;

    public MybatisAutoConfiguration(MybatisProperties properties,
            ObjectProvider<Interceptor[]> interceptorsProvider,
            ResourceLoader resourceLoader,
            ObjectProvider<DatabaseIdProvider> databaseIdProvider,
            ObjectProvider<List<ConfigurationCustomizer>> configurationCustomizersProvider) {
        this.properties = properties;
        this.interceptors = interceptorsProvider.getIfAvailable();
        this.resourceLoader = resourceLoader;
        this.databaseIdProvider = databaseIdProvider.getIfAvailable();
        this.configurationCustomizers = configurationCustomizersProvider.getIfAvailable();
    }

    @PostConstruct
    public void checkConfigFileExists() {
        if (this.properties.isCheckConfigLocation() && StringUtils.hasText(this.properties.getConfigLocation())) {
            Resource resource = this.resourceLoader.getResource(this.properties.getConfigLocation());
            Assert.state(resource.exists(), "Cannot find config location: " + resource
                    + " (please add config file or check your Mybatis configuration)");
        }
    }

    @Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setVfs(SpringBootVFS.class);
        if (StringUtils.hasText(this.properties.getConfigLocation())) {
            factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
        }
        Configuration configuration = this.properties.getConfiguration();
        if (configuration == null && !StringUtils.hasText(this.properties.getConfigLocation())) {
            configuration = new Configuration();
        }
        if (configuration != null && !CollectionUtils.isEmpty(this.configurationCustomizers)) {
            for (ConfigurationCustomizer customizer : this.configurationCustomizers) {
                customizer.customize(configuration);
            }
        }
        factory.setConfiguration(configuration);
        if (this.properties.getConfigurationProperties() != null) {
            factory.setConfigurationProperties(this.properties.getConfigurationProperties());
        }
        if (!ObjectUtils.isEmpty(this.interceptors)) {
            factory.setPlugins(this.interceptors);
        }
        if (this.databaseIdProvider != null) {
            factory.setDatabaseIdProvider(this.databaseIdProvider);
        }
        if (StringUtils.hasLength(this.properties.getTypeAliasesPackage())) {
            factory.setTypeAliasesPackage(this.properties.getTypeAliasesPackage());
        }
        if (StringUtils.hasLength(this.properties.getTypeHandlersPackage())) {
            factory.setTypeHandlersPackage(this.properties.getTypeHandlersPackage());
        }
        if (!ObjectUtils.isEmpty(this.properties.resolveMapperLocations())) {
            factory.setMapperLocations(this.properties.resolveMapperLocations());
        }

        return factory.getObject();
    }

    @Bean
    @ConditionalOnMissingBean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        ExecutorType executorType = this.properties.getExecutorType();
        if (executorType != null) {
            return new SqlSessionTemplate(sqlSessionFactory, executorType);
        } else {
            return new SqlSessionTemplate(sqlSessionFactory);
        }
    }

    public static class AutoConfiguredMapperScannerRegistrar
            implements BeanFactoryAware, ImportBeanDefinitionRegistrar, ResourceLoaderAware {
        private BeanFactory beanFactory;
        private ResourceLoader resourceLoader;

        @Override
        public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
                BeanDefinitionRegistry registry) {

            ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
            try {
                if (this.resourceLoader != null) {
                    scanner.setResourceLoader(this.resourceLoader);
                }

                List<String> packages = AutoConfigurationPackages.get(this.beanFactory);
                if (logger.isDebugEnabled()) {
                    for (String pkg : packages) {
                        logger.debug("Using auto-configuration base package '{}'", pkg);
                    }
                }

                scanner.setAnnotationClass(Mapper.class);
                scanner.registerFilters();
                scanner.doScan(StringUtils.toStringArray(packages));
            } catch (IllegalStateException ex) {
                logger.debug("Could not determine auto-configuration package, automatic mapper scanning disabled.", ex);
            }
        }

        @Override
        public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
            this.beanFactory = beanFactory;
        }

        @Override
        public void setResourceLoader(ResourceLoader resourceLoader) {
            this.resourceLoader = resourceLoader;
        }
    }

    @org.springframework.context.annotation.Configuration
    @Import({ AutoConfiguredMapperScannerRegistrar.class })
    @ConditionalOnMissingBean(MapperFactoryBean.class)
    public static class MapperScannerRegistrarNotFoundConfiguration {

        @PostConstruct
        public void afterPropertiesSet() {
            logger.debug("No {} found.", MapperFactoryBean.class.getName());
        }
    }
}
```

这个类就是一个`Configuration`（配置类），它里面定义很多bean，其中最重要的就是`SqlSessionFactory`的bean实例，该实例是`Mybatis`的核心功能，用它创建`SqlSession`，对数据库进行CRUD操作。

除此之外，`MybatisAutoConfiguration`类还包含了：

 - `@ConditionalOnClass` 配置了只有包含`SqlSessionFactory.class`和`SqlSessionFactoryBean.class`，该配置类才生效。
 - `@ConditionalOnBean` 配置了只有包含`dataSource`实例时，该配置类才生效。
 - `@EnableConfigurationProperties` 该注解会自动填充`MybatisProperties`实例中的属性。
 - `AutoConfigureAfter` 配置了该配置类在`DataSourceAutoConfiguration`类之后自动配置。

这些注解都是一些辅助功能，决定`Configuration`是否生效，当然这些注解不是必须的。

接下来，重点看看`spring.factories`文件有啥内容：
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```
里面只有一行配置，即`key`为`EnableAutoConfiguration`，`value`为`MybatisAutoConfiguration`。

好了，介绍了这么多东西，现在我们来总结一下，

starter几个要素如下图所示：

![img_4.png](/img/spring/spring-boot-starter/img_4.png)

那么，编写starter需要哪些步骤?

 1. 需要定义一个名称为`xxx-spring-boot-starter`的空项目，里面不包含任何代码，可以有`pom.xml`和`pom.properties`文件。
 2. `pom.xml`文件中包含了名称为`xxx-spring-boot-autoconfigure`的项目。
 3. `xxx-spring-boot-autoconfigure`项目中包含了名称为`xxxAutoConfiguration`的类，该类可以定义一些`bean`实例。当然，`Configuration`类上可以打一些如：`ConditionalOnClass`、`ConditionalOnBean`、`EnableConfigurationProperties`等注解。
 4. 需要在`spring.factories`文件中增加`key`为`EnableAutoConfiguration`，`value`为`xxxAutoConfiguration`。

我们试着按照这四步，自己编写一个`starter`看看能否成功，验证一下总结的内容是否正确。

## 3 如何定义自己的starter？

### 3.1 先创建一个空项目

该项目名称为id-generate-starter，注意为了方便我把项目重命名了，原本应该是叫id-generate-spring-boot-starter的，如下图所示：

![img_5.png](/img/spring/spring-boot-starter/img_5.png)

`pom.xml`文件定义如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <version>1.3.1</version>
    <groupId>com.sue</groupId>
    <artifactId>id-generate-spring-boot-starter</artifactId>
    <name>id-generate-spring-boot-starter</name>
    <dependencies>
        <dependency>
            <groupId>com.sue</groupId>
            <artifactId>id-generate-spring-boot-autoconfigure</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
</project>
```

我们看到，它只引入了`id-generate-spring-boot-autoconfigure`。当然如果有需要这里还可以引入多个`autoconfigure`或者多个其他jar包或者。

### 3.2 创建id-generate-autoconfigure

同样为了方便我把项目重命名了，原本是叫`id-generate-spring-boot-autoconfigure`，如下图所示：

![img_6.png](/img/spring/spring-boot-starter/img_6.png)

该项目当中包含：`pom.xml`、`spring.factories`、`IdGenerateAutoConfiguration`、`IdGenerateService` 和 `IdProperties` 这5个关键文件，下面我们逐一看看。

先从`pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <version>1.3.1</version>
    <groupId>com.sue</groupId>
    <artifactId>id-generate-spring-boot-autoconfigure</artifactId>
    <name>id-generate-spring-boot-autoconfigure</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
我们可以看到，这个文件比较简单就引入了：
 - spring-boot-starter：springboot的相关jar包。
 - spring-boot-autoconfigure：springboot自动配置相关jar包。
 - spring-boot-configuration-processor：springboot生成IDE提示功能相关jar包。

重点看看`spring.factories`文件：
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.sue.IdGenerateAutoConfiguration
```

它里面只包含一行配置，其中`key`是`EnableAutoConfiguration`，`value`是`IdGenerateAutoConfiguration`。

再重点看一下`IdGenerateAutoConfiguration`
```java
@ConditionalOnClass(IdProperties.class)
@EnableConfigurationProperties(IdProperties.class)
@Configuration
public class IdGenerateAutoConfiguration {
    @Autowired
    private IdProperties properties;

    @Bean
    public IdGenerateService idGenerateService() {
        return new IdGenerateService(properties.getWorkId());
    }
}
```
该类是一个使用了`@Configuration`注解标记为了配置类，生效的条件是`@ConditionalOnClass`注解中检测到包含`IdProperties.class`。并且使用`@EnableConfigurationProperties`注解会自动注入`IdProperties`的实例。

此外，最关键的点是该类里面创建了`idGenerateService`的`bean`实例，这是自动配置的精髓。

再看看`IdGenerateService`
```java
public class IdGenerateService {
    private Long workId;

    public IdGenerateService(Long workId) {
        this.workId = workId;
    }

    public Long generate() {
        return new Random().nextInt(100) + this.workId;
    }
}
```
我们可以看到它是一个普通的类，甚至都没有使用`@Service`注解，里面有个`generate`方法，根据`workId`的值和随机数动态生成`id`。

最后看看`IdProperties`
```java
@ConfigurationProperties(prefix = IdProperties.PREFIX)
public class IdProperties {
    public static final String PREFIX = "sue";
    private Long workId;

    public Long getWorkId() {
        return workId;
    }

    public void setWorkId(Long workId) {
        this.workId = workId;
    }
}
```
它是一个配置实体类，里面包含了相关的配置文件。使用`@ConfigurationProperties`注解，会自动把`application.properties`文件中以`sue`开通的，参数名称跟`IdProperties`中一样的参数值，自动注入到`IdProperties`对象中。

### 3.3 创建`id-generate-test`

这个项目主要用于测试。

![img_7.png](/img/spring/spring-boot-starter/img_7.png)

该项目里面包含：`pom.xml`、`application.properties`、`Application` 和 `TestRunner` 文件。

先看看`pom.xml`文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <version>1.3.1</version>
    <groupId>com.sue</groupId>
    <artifactId>spring-boot-id-generate-test</artifactId>
    <name>spring-boot-id-generate-test</name>
    <dependencies>
        <dependency>
            <groupId>com.sue</groupId>
            <artifactId>id-generate-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
</project>
```

由于只测试刚刚定义的id生成功能，所以只引入的`id-generate-spring-boot-starter jar`包。

`application.properties`配置资源文件
```
sue.workId=123
```

只有一行配置，因为我们的`IdProperties`中目前只需要这一个参数。

`Application`是测试程序启动类
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
很简单，就是一个普通的`springboot`启动类

`TestRunner`是我们的测试类
```java
@Component
public class TestRunner implements ApplicationRunner {
    @Autowired
    private IdGenerateService idGenerateService;

    public void run(ApplicationArguments args) throws Exception {
        Long sysNo = idGenerateService.generate();
        System.out.println(sysNo);
    }
}
```
它实现了`ApplicationRunner`接口，所以在`springboot`启动的时候会调用该类的`run`方法。

好了，所有自定义`starter`的代码和测试代码都已经就绪。接下，运行一下`Application`类的`main`方法。

运行结果：
```
176
```

完美，验证成功了。

接下来，我们分析一下`starter`的底层实现原理。

## 4 starter的底层原理是什么？

通过上面编写自己的`starter`的例子，相信大家对`starter`的认识更进一步了，现在跟大家一起看看`starter`的底层是如何实现的。

`id-generate-starter.jar`其实是一个空项目，依赖于`id-generate-autoconfiguration.jar`。

`id-generate-starter.jar`是一个入口，我们给他取一个更优雅的名字：门面模式，其他业务系统想引入相应的功能，必须要通过这个门面。

我们重点分析一下 `id-generate-autoconfiguration.jar`

该`jar`包核心内容是：`IdGenerateConfiguration`，这个配置类中创建了`IdGenerateService`对象，`IdGenerateService`是我们所需要自动配置的具体功能。

接下来一个最重要的问题：`IdGenerateConfiguration`为什么会自动加载的呢？

还记得我们定义的`spring.factories`文件不？
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.sue.IdGenerateAutoConfiguration
```
它里面只包含一行配置，其中`key`是`EnableAutoConfiguration`，`value`是`IdGenerateAutoConfiguration`。

要搞明白这个过程，要从`Application`类的`@SpringBootApplication`注解开始：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

    @AliasFor(annotation = EnableAutoConfiguration.class)
    Class<?>[] exclude() default {};

    @AliasFor(annotation = EnableAutoConfiguration.class)
    String[] excludeName() default {};

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};
}
```
从上面可以看出该注解里面包含了`@EnableAutoConfiguration`注解。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
`@EnableAutoConfiguration`注解会引入`AutoConfigurationImportSelector`类。

该类的`selectImports`方法一个关键方法：
```java
@Override
public class Application {
    // ......
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //配置有没有配置spring.boot.enableautoconfiguration开关，默认为true
        //如果为false，则不执行自动配置的功能，直接返回
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        //找spring-autoconfigure-metadata.properties中的元素
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
                .loadMetadata(this.beanClassLoader);
        //获取EnableAutoConfiguration注解中的属性
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        //获取工程下所有配置key为EnableAutoConfiguration的值，即IdGenerateConfiguration等类。
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        //删除重复的值    
        configurations = removeDuplicates(configurations);
        //获取需要排除的规则列表
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        //检查
        checkExcludedClasses(configurations, exclusions);
        //删除需要排除的值
        configurations.removeAll(exclusions);
        //根据配置文件中配置的开关，过滤一部分不满足条件的值
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return StringUtils.toStringArray(configurations);
    }
    // ......
}
```

这里就是`starter`能够自动配置的**秘密**。

此外，有些朋友看其他人定义的`springboot starter`可能会有疑惑。

先看看`druid-spring-boot-starter`

![img_8.png](/img/spring/spring-boot-starter/img_8.png)

`alibaba`定义的`druid-spring-boot-starter`只有`xxx-spring-boot-starter.jar`文件，而没有`xxx-spring-boot-autoconfigure.jar`文件。

再看看`spring-boot-starter-jdbc`：

![img_9.png](/img/spring/spring-boot-starter/img_9.png)

更神奇的是这个文件中连`pom.xml`都没有，一脸懵逼。。。。。。。

是不是我讲错了？

答：其实没有。

`SpringBoot`的原则是约定优于配置。

从`spring-boot-starter-jdbc`内部空实现来看，它的约定是要把`xxx-spring-boot-starter.jar`和`xxx-spring-boot-autoconfigure.jar`区分开的。个人认为，`alibaba`定义得并不好，没有遵照`springboot`的约定，虽然功能不受影响。(这个地方欢迎一起探讨一下)

而`springboot`自己定义的`spring-boot-starter-jdbc`为什么连`pom.xml`文件也没有呢？

它不需要依赖`xxx-spring-boot-autoconfigure.jar`文件吗？

因为`springboot`把所有的自动配置的类都统一放到`spring-boot-autoconfigure.jar`下面了：

![img_10.png](/img/spring/spring-boot-starter/img_10.png)

`spring.factories`文件内容如下：

![img_11.png](/img/spring/spring-boot-starter/img_11.png)

`SpringBoot`这样集中管理自动配置，而不需要从各个子包中遍历，我个人认为是为了查找效率。

我们最后再看看`spring-cloud-starter-openfegin`

![img_12.png](/img/spring/spring-boot-starter/img_12.png)

明显看到，它是遵循了我们说的原则的。

除此之外，还有一个原则一顺便提一下。

`SpringBoot`和`SpringCloud`系列定义`jar`包的名称是:
 - `spring-boot-starter-xxx.jar`
 - `spring-cloud-starter-xxx.jar`

而我们自己的项目定义的`jar`应该是：
- `xxx-spring-boot-starter.jar`
