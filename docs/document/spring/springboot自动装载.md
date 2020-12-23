# springboot自动装载

## 从注解开始分析

![img](https:////upload-images.jianshu.io/upload_images/7220517-e39e0b05b79de03f.png?imageMogr2/auto-orient/strip|imageView2/2/w/834/format/webp)

image.png



SpringBootApplication.java:



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
//重点看这里
@SpringBootConfiguration
//重点看这里
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM,
                classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication { …… }  
```

分别查看SpringBootApplication上的注解，发现EnableAutoConfiguration.java有比较重要的信息:



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
//重点看这里
@AutoConfigurationPackage
//重点看这里
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    /**
     * Exclude specific auto-configuration classes such that they will never be applied.
     * @return the classes to exclude
     */
    Class<?>[] exclude() default {};

    /**
     * Exclude specific auto-configuration class names such that they will never be
     * applied.
     * @return the class names to exclude
     * @since 1.3.0
     */
    String[] excludeName() default {};

}
```

> @AutoConfigurationPackage：表示将该类所对应的包加入到自动配置。即将MainApplication.java对应的包名加入到自动装配。@Import(AutoConfigurationImportSelector.class) ：这里在程序初始化的时候，会执行AutoConfigurationImportSelector.AutoConfigurationGroup.process，自动装载bean，详情逻辑请继续看下面。

AutoConfigurationImportSelector.java:



```java
public class AutoConfigurationImportSelector
        implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
        BeanFactoryAware, EnvironmentAware, Ordered {

}
```

AutoConfigurationImportSelector的类图：



![img](https:////upload-images.jianshu.io/upload_images/7220517-48bb03138bdc2306.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

> 可见其实现了DeferredImportSelector类，而DeferredImportSelector又继承ImportSelector类。该实现关系请记住，将有利于下面的代码分析。

在这里，先跟大家说一下，程序初始化的时候会执行AutoConfigurationImportSelector.AutoConfigurationGroup.process（为什么会执行该方法？详情查看2）
 那么该方法实现了什么逻辑呢？



```java
@Override
        public void process(AnnotationMetadata annotationMetadata,
                DeferredImportSelector deferredImportSelector) {
            Assert.state(
                    deferredImportSelector instanceof AutoConfigurationImportSelector,
                    () -> String.format("Only %s implementations are supported, got %s",
                            AutoConfigurationImportSelector.class.getSimpleName(),
                            deferredImportSelector.getClass().getName()));
            //重点看这里：获取自动装配实体
            AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
                    .getAutoConfigurationEntry(getAutoConfigurationMetadata(),
                            annotationMetadata);
            this.autoConfigurationEntries.add(autoConfigurationEntry);
            for (String importClassName : autoConfigurationEntry.getConfigurations()) {
                this.entries.putIfAbsent(importClassName, annotationMetadata);
            }
        }
```

其中再看一下getAutoConfigurationEntry方法：



```php
/**
     * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
     * of the importing {@link Configuration @Configuration} class.
     * @param autoConfigurationMetadata the auto-configuration metadata
     * @param annotationMetadata the annotation metadata of the configuration class
     * @return the auto-configurations that should be imported
     */
    protected AutoConfigurationEntry getAutoConfigurationEntry(
            AutoConfigurationMetadata autoConfigurationMetadata,
            AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
       //猜测：获取配置类，从哪里获取？再细看这里的代码
        List<String> configurations = getCandidateConfigurations(annotationMetadata,
                attributes);
      //删除重复名称的bean
        configurations = removeDuplicates(configurations);
      //获取元数据的不应包括的类名（excludeName、exclude）
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
      //删除不应包括的类名
        configurations.removeAll(exclusions);
      //过滤需要过滤的配置类
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationEntry(configurations, exclusions);
    }
```

下面来重点看看该方法的实现：



```dart
       //猜测：获取配置类，从哪里获取？再细看这里的代码
        List<String> configurations = getCandidateConfigurations(annotationMetadata,
                attributes);
```

org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getCandidateConfigurations：



```java
/**
     * Return the auto-configuration class names that should be considered. By default
     * this method will load candidates using {@link SpringFactoriesLoader} with
     * {@link #getSpringFactoriesLoaderFactoryClass()}.
     * @param metadata the source metadata
     * @param attributes the {@link #getAttributes(AnnotationMetadata) annotation
     * attributes}
     * @return a list of candidate configurations
     */
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
            AnnotationAttributes attributes) {
       //实质调用了SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, this.beanClassLoader);
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
                getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                "No auto configuration classes found in META-INF/spring.factories. If you "
                        + "are using a custom packaging, make sure that file is correct.");
        return configurations;
    }

  /**
     * Return the class used by {@link SpringFactoriesLoader} to load configuration
     * candidates.
     * @return the factory class
     */
    protected Class<?> getSpringFactoriesLoaderFactoryClass() {
        return EnableAutoConfiguration.class;
    }
```

从上面的代码注解中,继续深入loadFactoryNames方法，即获取EnableAutoConfiguration为key的所有类名
 org.springframework.core.io.support.SpringFactoriesLoader#loadFactoryNames：



```dart
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();
        return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    }

    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = cache.get(classLoader);
        if (result != null) {
            return result;
        }

        try {
            Enumeration<URL> urls = (classLoader != null ?
        //注解1：FACTORIES_RESOURCE_LOCATION=META-INF/spring.factories,即从该路径下获取类   
                classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                    ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
            result = new LinkedMultiValueMap<>();
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                for (Map.Entry<?, ?> entry : properties.entrySet()) {
                    String factoryClassName = ((String) entry.getKey()).trim();
                    for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                        result.add(factoryClassName, factoryName.trim());
                    }
                }
            }
            cache.put(classLoader, result);
            return result;
        }
        catch (IOException ex) {
            throw new IllegalArgumentException("Unable to load factories from location [" +
                    FACTORIES_RESOURCE_LOCATION + "]", ex);
        }
    }
```

> 注解1：就是加载所有包的spring.factories文件，然后逐个解释
>  其中：
>  Properties properties = PropertiesLoaderUtils.loadProperties(resource);
>  解释spring.factories文件
>  如下图所示，某个包下的spring.factories文件：

![img](https:////upload-images.jianshu.io/upload_images/7220517-82552b86ec288d6b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

> 转化为
>  org.springframework.boot.autoconfigure.EnableAutoConfiguration->
>  org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration

相关流程图如下：



![img](https:////upload-images.jianshu.io/upload_images/7220517-f8cef421ad9a3847.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

# 2. 为什么会执行AutoConfigurationImportSelector.AutoConfigurationGroup.process？

从springApplication.run开始：



![img](https:////upload-images.jianshu.io/upload_images/7220517-19d2410f439b386f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

> 代码太多，挑重点的说：
>  org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass
>  会处理如下图所示的注解，在分析@Import注解时，会查看该类是不是实现DeferredImportSelector类，如果是，则会调用this.deferredImportSelectorHandler.process()。很明显，在前面提到，AutoConfigurationImportSelector实现了DeferredImportSelector，所以在后面的逻辑中，会调用AutoConfigurationImportSelector.AutoConfigurationGroup.process方法

![img](https:////upload-images.jianshu.io/upload_images/7220517-8292263536edee02.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

> 总结：
>
> 1. springboot会自动扫描mainApplication类所在包的所有bean
> 2. springboot会自动扫描jar包下面的/META-INF/spring.factories文件，将EnableAutoConfiguration为key的所有类名加载出来，并进行去重、过滤

