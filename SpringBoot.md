## SpringBoot

### 一、Spring与SpringBoot

SpringBoot是整合Spring技术栈的一站式框架

SpringBoot是简化Spring技术栈的快速开发脚手架

#### 1.1 时代背景

**微服务：**

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务允许在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

**分布式的困难：**

- 远程调用
- 服务发现
- 负载均衡
- 服务容错
- 配置管理
- 服务监控
- 日志管理
- 任务调度

**分布式的解决：**

SprngBoot + SpringCloud

```java
package cn.imut;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

import java.util.Arrays;

/**
 * 主程序类
 */
@SpringBootApplication
public class SpringbootLearnApplication {

    public static void main(String[] args) {
        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(SpringbootLearnApplication.class, args);
        //查看容器里的组件
        String[] beanDefinitionNames = run.getBeanDefinitionNames();
        for (String s : beanDefinitionNames) {
            System.out.println(s);
        }
    }

}
```



### 二、底层注解

#### 2.1 @Configuration

```java
@Configuration(proxyBeanMethods = true)			//告诉SpringBoot这是一个配置类 == 配置文件
```

- 配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单例的
- 配置类本身也是组件
- **Full模式与Lite模式**
  - Lite(false)，可以加速容器启动过程，减少判断
  - Full模式，方法会被调用得到之间单例组件

#### 2.2 @Import

```java
@Import(DBHelper.class)		//给容器中自动创建出这两个类型的组件
```

#### 2.3 @Conditional

条件装配、

```java
@ConditionalOnBean(name = "tom")
```

#### 2.4 @ImportResource

导入资源、

```java
@ImportResource("classpath:beans.xml")
```

#### 2.5 @ConfigurationProperties

配置绑定、

### 三、自动配置

```java
package cn.imut;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

import java.util.Arrays;

/**
 * 主程序类
 */
@SpringBootApplication
public class SpringbootLearnApplication {

    public static void main(String[] args) {
        //返回IOC容器
        SpringApplication.run(SpringbootLearnApplication.class, args);
    }
}
```

@SpringBootApplication：该注解是启动类核心注解

点进去之后：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

1、@SpringBootConfiguration，点进去

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
```

@Configuration、代表当前是一个配置类

2、@ComponentScan，指定扫描哪些Spring注解

3、@EnableAutoConfiguration，点进去

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

3.1、@AutoConfigurationPackage、点进去

自动配置包

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)		//给容器中导入一个组件
public @interface AutoConfigurationPackage {
```

3.1.1、@Import里的 Registrar，点进去

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImports(metadata));
    }

}
```

批量注册一系列组件！

将指定的一个包下的所有组件导入进来（Main程序所在的包下）

3.2、@Import(AutoConfigurationImportSelector.class)，点进去

第一个方法：

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

重点是：getAutoConfigurationEntry方法、均是调用此方法进行的获取

点进去、

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

​    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);

configurations里，就是我们需要导入的所有组件！

点进入 getCandidateConfigurations、

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
getBeanClassLoader());
Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
+ "are using a custom packaging, make sure that file is correct.");
return configurations;
}
```

点进去 loadFactoryNames、这个工厂

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    ClassLoader classLoaderToUse = classLoader;
    if (classLoaderToUse == null) {
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
    }
    String factoryTypeName = factoryType.getName();
    return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

点进去 loadSpringFactories、

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    Map<String, List<String>> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }
    ......
```

通过DeBug、知道其 通过 META-INF/Spring.factories位置来加载一个文件！

默认扫描我们当前系统里面所有META-INF/Spring.factorie位置的文件

spring-boot-autoconfigure-2.4.1.jar包里的 META-INF/Spring.factorie

文件里写死了！！！SpringBoot一启动就要给容器中加载的所有配置类

