## Spring Boot启动流程


DefaultResourceLoader ： 作为 ResourceLoader 接口的直接实现类，该类实现了基本的资源加载功能，可以实现对单个资源的加载。

ResourcePatternResolver ：该接口继承了 ResourceLoader，定义了加载多个资源的方法， 可以实现对多个资源的加载。

1.2 Set<Class<?>>   primarySources存储启动类 

1.3 this.webApplicationType = deduceWebApplicationType();   应用的类型 ，创建的是一个 SERVLET （WEB）应用还是 REACTIVE应用或者是 NONE
ClassUtils.isPresent 传入类className 返回类是否存在 true or false
#server config
#是否设定web应用，none-非web，servlet-web应用
spring.main.web-application-type=servlet
#加载springboot banner的方式：off-关闭，console-控制台，log-日志
spring.main.banner-mode=off
```

1.4 setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class))；

面的 SpringFactoriesLoader.loadFactoryNames() ，是从 META-INF/spring.factories 的资源文件中，读取 key 为org.springframework.context.ApplicationContextInitializer 的 value。

org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer

org.springframework.boot.context.ContextIdApplicationContextInitializer

org.springframework.boot.context.config.DelegatingApplicationContextInitializer

org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer

org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

createSpringFactoriesInstances 用上面获取的名字反射创建实例,

 获取或创建所有 类型的ApplicationContextInitializer 实例放到 List<ApplicationContextInitializer<?>>  initializers 里

1.5 setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class)); 

面的 SpringFactoriesLoader.loadFactoryNames() ，是从 META-INF/spring.factories 的资源文件中，读取 key 为org.springframework.context.ApplicationContextInitializer 的 value。

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

createSpringFactoriesInstances 用上面获取的名字反射创建实例,

 获取或创建所有 类型的ApplicationListener 实例放到 List<ApplicationContextInitializer<?>>  initializers 里