## Spring 启动流程

### SpringBoot 启动类SpringApplication构造方法







##### getRunListeners

我们先看看SpringApplicationRunListeners和SpringApplicationRunListener。

SpringApplicationRunListeners的类注释很简单：

一个存SpringApplicationRunListener的集合，里面有些方法，后续都会讲到；

SpringApplicationRunListener的接口注释也简单：

监听SpringApplication的run方法。通过SpringFactoriesLoader加载SpringApplicationRunListener（一个或多个），SpringApplicationRunListener的实现类必须声明一个接收SpringApplication实例和String[]数组的公有构造方法。

　　接下来我们看看getRunListeners方法，源代码如下

```
private SpringApplicationRunListeners getRunListeners(String[] args) {
        Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
        return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
                SpringApplicationRunListener.class, types, this, args));
    }
```

返回一个新的SpringApplicationRunListeners实例对象；



getApplicationListener　

getApplicationListeners方法过滤出的监听器都会被调用，过滤出来的监听器包括LoggingApplicationListener、BackgroundPreinitializer、DelegatingApplicationListener、LiquibaseServiceLocatorApplicationListener、EnableEncryptablePropertiesBeanFactoryPostProcessor五种类型的对象。这五个对象的onApplicationEvent都会被调用。

 那么这五个监听器的onApplicationEvent都做了些什么了，我这里大概说下，细节的话大家自行去跟源码

LoggingApplicationListener：检测正在使用的日志系统，默认是logback，支持3种，优先级从高到低：logback > log4j > javalog。此时日志系统还没有初始化

BackgroundPreinitializer：另起一个线程实例化Initializer并调用其run方法，包括验证器、消息转换器等等

DelegatingApplicationListener：此时什么也没做

LiquibaseServiceLocatorApplicationListener：此时什么也没做

EnableEncryptablePropertiesBeanFactoryPostProcessor：此时仅仅打印了一句日志，其他什么也没做