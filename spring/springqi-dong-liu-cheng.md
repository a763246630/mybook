## Spring Boot启动流程

#### 1.执行启动类SpringApplication构造

SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources)

1.1 ResourceLoader接口 

接口有一个特别重要的方法：Resource getResource(String location)，返回Resource实例

内部构造：

```java
public interface ResourceLoader {

    // 从 classpath 加载资源时的前缀
    String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

    // 关键-> 取得 Resource 对象，即获取资源
    Resource getResource(String location);

    ClassLoader getClassLoader();
}
```

DefaultResourceLoader ： 作为 ResourceLoader 接口的直接实现类，该类实现了基本的资源加载功能，可以实现对单个资源的加载。

ResourcePatternResolver ：该接口继承了 ResourceLoader，定义了加载多个资源的方法， 可以实现对多个资源的加载。

1.2 Set<Class<?>>   primarySources存储启动类 

1.3 this.webApplicationType = deduceWebApplicationType();   应用的类型 ，创建的是一个 SERVLET （WEB）应用还是 REACTIVE应用或者是 NONE

```
ClassUtils.isPresent 传入类className 返回类是否存在 true or false
#server config
#是否设定web应用，none-非web，servlet-web应用
spring.main.web-application-type=servlet
#加载springboot banner的方式：off-关闭，console-控制台，log-日志
spring.main.banner-mode=off
```

1.4 setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class))；

 获取或创建 ApplicationContextInitializer 实例放到 List<ApplicationContextInitializer<?>>  initializers 里