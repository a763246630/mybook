## 线程池ThreadPool

**线程池**（thread pool）：是一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 例如，线程数一般取cpu数量+2比较合适，线程数过多会导致额外的线程切换开销。

线程池的[伸缩性](https://baike.baidu.com/item/伸缩性)对性能有较大的影响。

* 创建太多线程，将会浪费一定的资源，有些线程未被充分使用。
* 销毁太多线程，将导致之后浪费时间再次创建它们。
* 创建线程太慢，将会导致长时间的等待，性能变差。
* 销毁线程太慢，导致其它线程[资源](https://baike.baidu.com/item/资源)饥饿。 \[1\] 

#### JAVA线程池ThreadPoolExecutor参数

* corePoolSize：核心线程数

  * 核心线程会一直存活，即使没有任务要执行

  * 当线程数小于核心线程数时，即使有线程空闲，线程池也会优先创建新线程处理

  * 设置allowCoreThreadTimeout=true（默认false）时，核心线程会超时关闭

* queueCapacity：任务队列容量（阻塞队列）

  * 当核心线程数达到最大时，新任务会放在队列中排队等待执行

* maxPoolSize：最大线程数

  * 当线程数&gt;=corePoolSize，且任务队列已满时。线程池会创建新线程来处理任务
  * 当线程数=maxPoolSize，且任务队列已满时，线程池会拒绝处理任务而抛出异常

* keepAliveTime：线程空闲时间

  * 当线程空闲时间达到keepAliveTime时，线程会释放，直到空闲线程数量等于corePoolSize不再释放
  * 如果allowCoreThreadTimeout=true，则会直到线程可以全部释放

* allowCoreThreadTimeout：允许核心线程超时

* rejectedExecutionHandler：任务拒绝处理器

* 两种情况会拒绝处理任务：

  * 当线程数已经达到maxPoolSize，且队列已满，会拒绝新任务
  * 当线程池被调用shutdown\(\)后，会等待线程池里的任务执行完毕，再shutdown。如果在调用shutdown\(\)和线程池真正shutdown之间提交任务，会拒绝新任务
  * 线程池会调用rejectedExecutionHandler来处理这个任务。如果没有设置默认是AbortPolicy，会抛出异常
  * ThreadPoolExecutor类有几个内部实现类来处理这类情况：
  * AbortPolicy 丢弃任务，抛运行时异常
  * CallerRunsPolicy 执行任务
  * DiscardPolicy 忽视，什么都不会发生
  * DiscardOldestPolicy 从队列中踢出最先进入队列（最后一个执行）的任务
  * 实现RejectedExecutionHandler接口，可自定义处理器

- unit   keepAliveTime存活时间 的单位
- workQueue







**ThreadPoolExecutor执行顺序：**

​     线程池按以下行为执行任务

1. 当线程数小于核心线程数时，创建线程。
2. 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。
3. 当线程数大于等于核心线程数，且任务队列已满
   1. 若线程数小于最大线程数，创建线程
   2. 若线程数等于最大线程数，抛出异常，拒绝任务

**如何设置参数**

* 默认值
  * corePoolSize=1
  * queueCapacity=Integer.MAX\_VALUE
  * maxPoolSize=Integer.MAX\_VALUE
  * keepAliveTime=60s
  * allowCoreThreadTimeout=false
  * rejectedExecutionHandler=AbortPolicy\(\)
* 如何来设置
  * 需要根据几个值来决定
  * tasks ：每秒的任务数，假设为500~1000
  * taskcost：每个任务花费时间，假设为0.1s
  * responsetime：系统允许容忍的最大响应时间，假设为1s
  * corePoolSize = 每秒需要多少个线程处理？ 
  * threadcount = tasks/\(1/taskcost\) =tasks_taskcout =  \(500~1000\)_0.1 = 50~100 个线程。corePoolSize设置应该大于50
  * 根据8020原则，如果80%的每秒任务数小于800，那么corePoolSize设置为80即可
  * queueCapacity = \(coreSizePool/taskcost\)\*responsetime
  * 计算可得 queueCapacity = 80/0.1\*1 = 80。意思是队列里的线程可以等待1s，超过了的需要新开线程来执行
  * 切记不能设置为Integer.MAX\_VALUE，这样队列会很大，线程数只会保持在corePoolSize大小，当任务陡增时，不能新开线程来执行，响应时间会随之陡增。
  * maxPoolSize = \(max\(tasks\)- queueCapacity\)/\(1/taskcost\)
  * 计算可得 maxPoolSize = \(1000-80\)/10 = 92
  * （最大任务数-队列容量）/每个线程每秒处理能力 = 最大线程数
  * rejectedExecutionHandler：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理
  * keepAliveTime和allowCoreThreadTimeout采用默认通常能满足
* 以上都是理想值，实际情况下要根据机器性能来决定。如果在未达到最大线程数的情况机器cpu load已经满了，则需要通过升级硬件（呵呵）和优化代码，降低taskcost来处理。

### 合理的设置线程池参数

**如何合理地估算线程池大小？**

这个问题虽然看起来很小，却并不那么容易回答。大家如果有更好的方法欢迎赐教，先来一个天真的估算方法：假设要求一个系统的TPS（Transaction Per Second或者Task Per Second）至少为20，然后假设每个Transaction由一个线程完成，继续假设平均每个线程处理一个Transaction的时间为4s。那么问题转化为：



**如何设计线程池大小，使得可以在1s内处理完20个Transaction？**

计算过程很简单，每个线程的处理能力为0.25TPS，那么要达到20TPS，显然需要20/0.25=80个线程。

很显然这个估算方法很天真，因为它没有考虑到CPU数目。一般服务器的CPU核数为16或者32，如果有80个线程，那么肯定会带来太多不必要的线程上下文切换开销。 

再来第二种简单的但不知是否可行的方法（N为CPU总核数）：

- 如果是CPU密集型应用，则线程池大小设置为N+1

  尽量使用较小的线程池，一般为CPU核心数+1。 
  因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

- 如果是IO密集型应用，则线程池大小设置为2N+1

  可以使用稍大的线程池，一般为2*CPU核心数。 
  IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。

- 混合型任务

  可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 
  只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。 
  因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。 

  ```
  对于计算密集型的任务，在拥有N个处理器的系统上，当线程池的大小为N+1时，通常能实现最优的效率。(即使当计算密集型的线程偶尔由于缺失故障或者其他原因而暂停时，这个额外的线程也能确保CPU的时钟周期不会被浪费。)
  ```

如果一台服务器上只部署这一个应用并且只有这一个线程池，那么这种估算或许合理，具体还需自行测试验证。



**线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。**

- 如果是CPU密集型应用，则线程池大小设置为N+1

  因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

- 如果是IO密集型应用，则线程池大小设置为2N+1

  可以使用稍大的线程池，一般为2*CPU核心数。 
  IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。

  混合型任务 
  可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 
  只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。 
  因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。 