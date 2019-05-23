## 线程池ThreadPool

**线程池**（thread pool）：是一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 例如，线程数一般取cpu数量+2比较合适，线程数过多会导致额外的线程切换开销。

线程池的[伸缩性](https://baike.baidu.com/item/%E4%BC%B8%E7%BC%A9%E6%80%A7)对性能有较大的影响。

- 创建太多线程，将会浪费一定的资源，有些线程未被充分使用。
- 销毁太多线程，将导致之后浪费时间再次创建它们。
- 创建线程太慢，将会导致长时间的等待，性能变差。
- 销毁线程太慢，导致其它线程[资源](https://baike.baidu.com/item/%E8%B5%84%E6%BA%90)饥饿。 [1] 

#### JAVA线程池ThreadPoolExecutor参数

- corePoolSize：核心线程数

  - 核心线程会一直存活，即使没有任务要执行

  - 当线程数小于核心线程数时，即使有线程空闲，线程池也会优先创建新线程处理

  - 设置allowCoreThreadTimeout=true（默认false）时，核心线程会超时关闭

- queueCapacity：任务队列容量（阻塞队列）

  - 当核心线程数达到最大时，新任务会放在队列中排队等待执行

- maxPoolSize：最大线程数
  - 当线程数>=corePoolSize，且任务队列已满时。线程池会创建新线程来处理任务
  - 当线程数=maxPoolSize，且任务队列已满时，线程池会拒绝处理任务而抛出异常

- keepAliveTime：线程空闲时间

  - 当线程空闲时间达到keepAliveTime时，线程会释放，直到空闲线程数量等于corePoolSize不再释放
  - 如果allowCoreThreadTimeout=true，则会直到线程可以全部释放

- allowCoreThreadTimeout：允许核心线程超时

- rejectedExecutionHandler：任务拒绝处理器

- 两种情况会拒绝处理任务：

  - 当线程数已经达到maxPoolSize，且队列已满，会拒绝新任务

  - 当线程池被调用shutdown()后，会等待线程池里的任务执行完毕，再shutdown。如果在调用shutdown()和线程池真正shutdown之间提交任务，会拒绝新任务

  - 线程池会调用rejectedExecutionHandler来处理这个任务。如果没有设置默认是AbortPolicy，会抛出异常

  - ThreadPoolExecutor类有几个内部实现类来处理这类情况：

  - AbortPolicy 丢弃任务，抛运行时异常

  - CallerRunsPolicy 执行任务

  - DiscardPolicy 忽视，什么都不会发生

  - DiscardOldestPolicy 从队列中踢出最先进入队列（最后一个执行）的任务

  - 实现RejectedExecutionHandler接口，可自定义处理器

 



**ThreadPoolExecutor执行顺序：**

​     线程池按以下行为执行任务

1. 当线程数小于核心线程数时，创建线程。
2. 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。
3. 当线程数大于等于核心线程数，且任务队列已满
   1. 若线程数小于最大线程数，创建线程
   2. 若线程数等于最大线程数，抛出异常，拒绝任务



### 合理的设置线程池参数

