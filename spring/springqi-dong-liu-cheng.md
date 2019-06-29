## Spring Boot启动流程

#### 1.执行启动类SpringApplication构造

 SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources)

ResourceLoader接口 

接口有一个特别重要的方法：Resource getResource(String location)，返回Resource实例

内部构造：



- DefaultResourceLoader ： 作为 ResourceLoader 接口的直接实现类，该类实现了基本的资源加载功能，可以实现对单个资源的加载。
- ResourcePatternResolver ：该接口继承了 ResourceLoader，定义了加载多个资源的方法， 可以实现对多个资源的加载。

Set<Class<?>>   primarySources存储启动类 

