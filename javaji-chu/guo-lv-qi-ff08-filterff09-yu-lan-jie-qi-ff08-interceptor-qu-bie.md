## 过滤器（Filter）与拦截器（Interceptor \)区别

#### 过滤器

它依赖于servlet容器。在实现上，基于函数回调，它可以对几乎所有请求进行过滤，。使用过滤器的目的，是用来做一些过滤操作，获取我们想要获取的数据，比如：在Javaweb中，对传入的request、response提前过滤掉一些信息，或者提前设置一些参数，然后再传入servlet或者Controller进行业务逻辑操作。通常用的场景是：在过滤器中修改字符编码（CharacterEncodingFilter）、在过滤器中修改HttpServletRequest的一些参数（XSSFilter(自定义过滤器)），如：过滤低俗文字、危险字符等。
 



**拦截器是基于java的反射机制的，**

它依赖于web框架，在SpringMVC中就是依赖于SpringMVC框架。在实现上,基于Java的反射机制，属于面向切面编程（AOP）的一种运用，就是在service或者一个方法前，调用一个方法，或者在方法后，调用一个方法，比如动态代理就是拦截器的简单实现，在调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在调用方法后打印出字符串，甚至在抛出异常的时候做业务逻辑的操作。由于拦截器是基于web框架的调用，因此可以使用Spring的依赖注入（DI）进行一些业务操作，同时一个拦截器实例在一个controller生命周期之内可以多次调用。 



调用顺序  Tomcat->filter—>Servlet->Inteceptor->controller

两者的本质区别：拦截器（Interceptor）是基于Java的反射机制，而过滤器（Filter）是基于函数回调。从灵活性上说拦截器功能更强大些，Filter能做的事情，都能做，而且可以在请求前和请求后都执行，比较灵活。Filter主要是针对URL地址做一个编码的事情、过滤掉没用的参数、安全校验（比较泛的，比如登录不登录之类），太细的话，还是建议用interceptor。不过还是根据不同情况选择合适的。
 

SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理。

1、日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。
2、权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；
3、性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；
4、通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。
5、OpenSessionInView：如hibernate，在进入处理器打开Session，在完成后关闭Session。

## 补充说明：

Spring的拦截器与Servlet的Filter有相似之处，比如二者都是AOP编程思想的体现，都能实现权限检查、日志记录等。不同的是：

1. 使用范围不同：Filter是Servlet规范规定的，只能用于Web程序中。而拦截器既可以用于Web程序，也可以用于Application、Swing程序中。
2. 规范不同：Filter是在Servlet规范中定义的，是Servlet容器支持的。而拦截器是在Spring容器内的，是Spring框架支持的。
3. 使用的资源不同：同其他的代码块一样，拦截器也是一个Spring的组件，归Spring管理，配置在Spring文件中，因此能使用Spring里的任何资源、对象，例如Service对象、数据源、事务管理等，通过IOC注入到拦截器即可；而Filter则不能。
4. 深度不同：Filter在只在Servlet前后起作用。而拦截器能够深入到方法前后、异常抛出前后等，因此拦截器的使用具有更大的弹性。所以在Spring构架的程序中，要优先使用拦截器。 
   　　Filter是一个可以复用的代码片段，可以用来转换Http请求、响应和头信息。Filter不像Servlet，它不能产生一个请求或者响应，它只是修改对某一资源的请求，或者修改从某一资源的响应。
   　　JSR中说明的是，按照多个匹配的Filter，是按照其在web.xml中配置的顺序来执行的。所以这也就是，把自己的Filter或者其他的Filter（比如UrlRewrite的Filter）放在Struts2的 DispatcherFilter的前面的原因。因为它们需要在请求被Struts2框架处理之前，做一些前置的工作。
   当Filter被调用，并且进入了Struts2的DispatcherFilter中后，Struts2会按照在Action中配置的Interceptor Stack中的Interceptor的顺序，来调用Interceptor。

 