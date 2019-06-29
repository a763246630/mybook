## Spring Boot启动流程

#### 1.执行启动类SpringApplication构造

SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources)

ResourceLoader接口 

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
