## Spring Boot启动流程

#### 1.执行启动类SpringApplication构造

SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources)

1.1 ResourceLoader接口 

接口有一个特别重要的方法：Resource getResource(String location)，返回Resource实例

内部构造：

```
public interface ResourceLoader {

    // 从 classpath 加载资源时的前缀
    String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

    // 关键-> 取得 Resource 对象，即获取资源
    Resource getResource(String location);

    ClassLoader getClassLoader();
}
```



面的 SpringFactoriesLoader.loadFactoryNames() ，是从 META-INF/spring.factories 的资源文件中，读取 key 为org.springframework.context.ApplicationContextInitializer 的 value。

```
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer
org.springframework.boot.context.ContextIdApplicationContextInitializer
org.springframework.boot.context.config.DelegatingApplicationContextInitializer
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

createSpringFactoriesInstances 用上面获取的名字反射创建实例,

 获取或创建所有 类型的ApplicationContextInitializer 实例放到 List<ApplicationContextInitializer<?>>  initializers 里

1.5 setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class)); 

面的 SpringFactoriesLoader.loadFactoryNames() ，是从 META-INF/spring.factories 的资源文件中，读取 key 为org.springframework.context.ApplicationContextInitializer 的 value。

```
org.springframework.boot.ClearCachesApplicationListener
org.springframework.boot.builder.ParentContextCloserApplicationListener
org.springframework.boot.context.FileEncodingApplicationListener
org.springframework.boot.context.config.AnsiOutputApplicationListener
org.springframework.boot.context.config.ConfigFileApplicationListener
org.springframework.boot.context.config.DelegatingApplicationListener
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener
org.springframework.boot.context.logging.LoggingApplicationListener
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
org.springframework.boot.autoconfigure.BackgroundPreinitializer
org.springframework.cloud.bootstrap.BootstrapApplicationListener
org.springframework.cloud.bootstrap.LoggingSystemShutdownListener
org.springframework.cloud.context.restart.RestartListener
```

createSpringFactoriesInstances 用上面获取的名字反射创建实例,

 获取或创建所有 类型的ApplicationListener 实例放到 List<ApplicationContextInitializer<?>>  initializers 里