\[TOC\]

### Spring中Bean生命周期过程

1.Spring对Bean进行实例化（相当于程序中的new Xx\(\)）

2.Spring将值和Bean的引用注入进Bean对应的属性中

3.如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName\(\)方法**（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的**）

4.如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanDactory\(BeanFactory bf\)方法并把BeanFactory容器实例作为参数传入。**（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）**

5.如果Bean实现了ApplicationContextAware接口，Spring容器将调用setApplicationContext\(ApplicationContext ctx\)方法，把y应用上下文作为参数传入.**\(作用与BeanFactory类似都是为了获取ApplicationContext spring 上下文\)**

6.如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法**（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）**

7.如果Bean实现了**InitializingBean**接口，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。

8.如果Bean实现了**BeanPostProcess**接口，Spring将调用它们的postProcessAfterInitialization（后初始化）方法**（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同\)**

注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。

9、当Bean不再需要时，会经过清理阶段，如果Bean实现了**DisposableBean**这个接口，会调用那个其实现的destroy\(\)方法；

10、最后，如果这个Bean的配置了destroy-method属性，会自动调用其配置的销毁方法。

### Spring事务

什么是事务:事务逻辑上的一组操作,组成这组操作的各个逻辑单元,要么一起成功,要么一起失败，不存在中间状态.

#### 事务的特性

##### 原子性 （atomicity）

关注事务状态,要么一起成功,要么一起失败，不存在中间状态.

##### 一致性 （consistency）

关注事务可见性，中间状态的数据对外不可见，只有最终和最初状态可见.

##### 隔离性 （isolation）

一个事务执行的过程中,不应该受到其他事务的干扰

##### 持久性（durability）

事务提交后,数据持久化到数据库

#### 各种隔离级别可能出现的问题

##### 脏读:一个事务读到了另一个事务未提交的数据

##### 不可重复读:一个事务多次读取相同的数据得到的值不一样。

##### 幻读:一个事务修改了表中所有数据，另一个事务新增了一条数据，只是第一个事务发现还有没有修改的数据像出现了幻觉一样。

#### 事务隔离级别

##### Mysql INODB默认:可重复读

Oracle 默认:读已提交

##### 未提交读（READ\_UNCOMMITTED）

隔离级别低，并发性能高，会出现脏读，不可重复读，幻读。

##### 已提交读（READ\_COMMITTED）

锁定修改行，会出现不可重复度，幻读。

##### 可重复读（REPEATABLE\_READ）

锁定所有读取行，会出现幻读

##### 串行读（SERIALIZABLE）

锁表，每次只能由一个线程处理表数据。

事务的7种传播级别（Propagation）

1.REQUIRED（依赖，必须）

如果有事务支持当前事务，没有则自己创建一个事务。

2.SUPPORTS（支持）

如果有事务支持当前事务，没有则以无事务的方式执行。

3.NOT\_SUPPORTED（不支持）

以非事务的方式执行操作，如果有事务存在，就把当前事务挂起。

4.REQUIRED\_NEW（必须新的）

不管有没有事务，都创建新事务，原来的事务挂起，新的执行完毕，继续执行老的。

5.MANDATORY\(强制\)

必须在一个事务中执行，否则抛出异常。

6.NEVER（从不）

必须在一个非事务方式中执行，否则抛出异常。

7.NESTED（嵌套）

如果存在事务，则创建事务以当前事务嵌套事务运行，如果没有，则新建一个事务执行，（等价于REQUIRED）。

当执行到内部事务B，外部事务会挂起，新建一个子事务B并设置savepoint等B事务执行完，外部事务才执行。

外部事务失败，内部外部事务都会回滚。内部事务B失败回滚抛出异常被外部事务捕获未抛出则外部事务A仍可以

提交，相反未捕获则回滚。

### Spring IOC

控制反转，将对象管理权（创建，销毁）全交给spring容器去管理。

### Spring AOP

面向切面编程，选择一个切入点，将原来的方法进行包装后执行。

cglib静态代理：代码编译期，利用ASM修改类字节码生产代理类.

jdk动态代理：运行时,通过反射获取到类信息，进行包装后加装载到jvm,要求类必须要实现接口。

Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。

Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。

Joint point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。

Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和

around 来区别是在每个 joint point 之前、之后还是代替执行的代码。

Target（目标对象）：织入 Advice 的目标对象.。

Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程

### Springboot 如何区分是浏览器请求还是ajax

通过 request head 来识别

accept:text/html 是浏览器

accept:_/_ 是请求接口

