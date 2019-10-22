## 线程池ThreadPool

\[TOC\]

**更快读写的存储介质+减少IO+减少CPU计算=性能优化。**

线程池（thread pool）：是一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 例如，线程数一般取cpu数量+2比较合适，线程数过多会导致额外的线程切换开销。

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

  ```
    ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
    ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
    ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
    ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
  ```

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

* unit   keepAliveTime存活时间 的单位

* workQueue 队列  
  ThreadPoolExecutors线程添加策略  
  1、线程数量未到corePoolSize\(核心线程数\)，则新建一个线程\(核心线程\)执行任务  
  2、线程数量达到了corePoolSize，则将任务移入队列（workqueue）等待核心线程执行完后执行  
  3、队列已满，新建线程\(非核心线程\)执行任务  
  4、队列已满，线程数又达到了maximumPoolSize（最大线程数），执行拒绝策略

  在使用ThreadPoolExecutor线程池的时候，需要指定一个实现了BlockingQueue接口的任务等待队列。在ThreadPoolExecutor线程池的API文档中，一共推荐了三种等待队列，它们是：SynchronousQueue、LinkedBlockingQueue和ArrayBlockingQueue；

  1. **直接提交**。工作队列的默认选项是 `SynchronousQueue`，它将任务直接提交给线程而不保存它们。如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
  2. **无界队列**。使用无界队列（例如，不具有预定义容量的 `LinkedBlockingQueue`）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
  3. **有界队列**。当使用有限的 maximumPoolSizes 时，有界队列（如 `ArrayBlockingQueue`）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。

  注意事项

  1. ThreadPoolExecutor的使用还是很有技巧的。
  2. 使用无界queue可能会耗尽系统资源。
  3. 使用有界queue可能不能很好的满足性能，需要调节线程数和queue大小
  4. 线程数自然也有开销，所以需要根据不同应用进行调节。

  通常来说对于静态任务可以归为：  
  1. 数量大，但是执行时间很短  
  2. 数量小，但是执行时间较长  
  3. 数量又大执行时间又长  
  4. 除了以上特点外，任务间还有些内在关系  
  5. CPU密集或者IO密集型任务

* threadFactory 实现工厂自定义线程创建方法

```
//线程的创建工厂
ThreadFactory threadFactory = new ThreadFactory() {
    private final AtomicInteger mCount = new AtomicInteger(1);

    public Thread newThread(Runnable r) {
        return new Thread(r, "AdvacnedAsyncTask #" + mCount.getAndIncrement());
    }
};
```

#### Executors提供的线程池配置方案

构造一个固定线程数目的线程池，配置的corePoolSize与maximumPoolSize大小相同，同时使用了一个无界LinkedBlockingQueue存放阻塞任务，因此多余的任务将存在再阻塞队列，不会由RejectedExecutionHandler处理

Executors提供的几种定制线程

**ExecutorService newFixedThreadPool\(int nThreads\):固定大小线程池。**

该线程池keepalive为0 线程执行完直接释放 ，核心线程和最大线程数相等,    workqueue选择了LinkedBlockingQueue,队列是无界的.

**ExecutorService newSingleThreadExecutor\(\)：单线程。**

和 newFixedThreadPool 类似 ，唯一区别线程数固定为1。

**ExecutorService newCachedThreadPool\(\)：无界线程池，可以进行自动线程回收。**

该线程池使用synchronousQueue ,keepalive为60秒,核心线程数为0 最大线程池数为Integer.MAX\_VALUE

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
  * threadcount = tasks/\(1/taskcost\) =tasks\_taskcout =  \(500~1000\)\_0.1 = 50~100 个线程。corePoolSize设置应该大于50
  * 根据8020原则，如果80%的每秒任务数小于800，那么corePoolSize设置为80即可
  * queueCapacity = \(coreSizePool/taskcost\)\*responsetime
  * 计算可得 queueCapacity = 80/0.1\*1 = 80。意思是队列里的线程可以等待1s，超过了的需要新开线程来执行
  * 切记不能设置为Integer.MAX\_VALUE，这样队列会很大，线程数只会保持在corePoolSize大小，当任务陡增时，不能新开线程来执行，响应时间会随之陡增。
  * maxPoolSize = \(max\(tasks\)- queueCapacity\)/\(1/taskcost\)
  * 计算可得 maxPoolSize = \(1000-80\)/10 = 92
  * （最大任务数-队列容量）/每个线程每秒处理能力 = 最大线程数
  * rejectedExecutionHandler：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理                  rejected = new ThreadPoolExecutor.AbortPolicy\(\);//默认，队列满了丢任务抛出异常  
    rejected = new ThreadPoolExecutor.DiscardPolicy\(\);//队列满了丢任务不异常  
    rejected = new ThreadPoolExecutor.DiscardOldestPolicy\(\);//将最早进入队列的任务删，之后再尝试加入队列  
    rejected = new ThreadPoolExecutor.CallerRunsPolicy\(\);//如果添加到线程池失败，那么主线程会自己去执行该任务

  * keepAliveTime和allowCoreThreadTimeout采用默认通常能满足

* 以上都是理想值，实际情况下要根据机器性能来决定。如果在未达到最大线程数的情况机器cpu load已经满了，则需要通过升级硬件（呵呵）和优化代码，降低taskcost来处理。

### 合理的设置线程池参数

**如何合理地估算线程池大小？**

这个问题虽然看起来很小，却并不那么容易回答。大家如果有更好的方法欢迎赐教，先来一个天真的估算方法：假设要求一个系统的TPS（Transaction Per Second或者Task Per Second）至少为20，然后假设每个Transaction由一个线程完成，继续假设平均每个线程处理一个Transaction的时间为4s。那么问题转化为：

**如何设计线程池大小，使得可以在1s内处理完20个Transaction？**

计算过程很简单，每个线程的处理能力为0.25TPS，那么要达到20TPS，显然需要20/0.25=80个线程。

很显然这个估算方法很天真，因为它没有考虑到CPU数目。一般服务器的CPU核数为16或者32，如果有80个线程，那么肯定会带来太多不必要的线程上下文切换开销。

再来第二种简单的但不知是否可行的方法（N为CPU总核数）：

* 如果是CPU密集型应用，则线程池大小设置为N+1

  尽量使用较小的线程池，一般为CPU核心数+1。  
  因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

* 如果是IO密集型应用，则线程池大小设置为2N+1

  可以使用稍大的线程池，一般为2\*CPU核心数。  
  IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。

* 混合型任务

  可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。  
  只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。  
  因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。

  ```
  对于计算密集型的任务，在拥有N个处理器的系统上，当线程池的大小为N+1时，通常能实现最优的效率。(即使当计算密集型的线程偶尔由于缺失故障或者其他原因而暂停时，这个额外的线程也能确保CPU的时钟周期不会被浪费。)
  ```

如果一台服务器上只部署这一个应用并且只有这一个线程池，那么这种估算或许合理，具体还需自行测试验证。

* 如果是CPU密集型应用，则线程池大小设置为N+1

  因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

* 如果是IO密集型应用，则线程池大小设置为2N+1

  可以使用稍大的线程池，一般为2\*CPU核心数。  
  IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。

  混合型任务  
  可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。  
  只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。  
  因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。

如果一台服务器上只部署这一个应用并且只有这一个线程池，那么这种估算或许合理，具体还需自行测试验证。

接下来在这个文档：服务器性能IO优化 中发现一个估算公式：

最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）\* CPU数目

比如平均每个线程CPU运行时间为0.5s，而线程等待时间（非CPU运行时间，比如IO）为1.5s，CPU核心数为8，那么根据上面这个公式估算得到：\(\(0.5+1.5\)/0.5\)\*8=32。这个公式进一步转化为：

最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）\* CPU数目

可以得出一个结论：

**线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。**

上一种估算方法也和这个结论相合。

一个系统最快的部分是CPU，所以决定一个系统吞吐量上限的是CPU。增强CPU处理能力，可以提高系统吞吐量上限。但根据短板效应，真实的系统吞吐量并不能单纯根据CPU来计算。那要提高系统吞吐量，就需要从“系统短板”（比如网络延迟、IO）着手：

* 尽量提高短板操作的并行化比率，比如多线程下载技术
* 增强短板能力，比如用NIO替代IO

第一条可以联系到Amdahl定律，这条定律定义了串行系统并行化后的加速比计算公式：

加速比=优化前系统耗时 / 优化后系统耗时

加速比越大，表明系统并行化的优化效果越好。Addahl定律还给出了系统并行度、CPU数目和加速比的关系，加速比为Speedup，系统串行化比率（指串行执行代码所占比率）为F，CPU数目为N：

Speedup &lt;= 1/ \(F + \(1-F\)/N\)

当N足够大时，串行化比率F越小，加速比Speedup越大。

##### 

# 不同的业务放在不同的线程/线程池里面

开发的时候我们基本上不会考虑到这种问题，整个服务就共用一个线程池，甚至有些系统是单线程的。

一旦出现问题整个服务就一起挂掉了

这个肯定是我们不想看到的。

解决这个问题方法就是把不同模块放在不同的线程里面，如果之前使用的是线程池那么 不同业务也要用不同的线程池分开。因为如果这个业务有问题，这个业务所在的线程池也会很快的阻塞掉。

如果不同的业务分开到不同的线程池里面去，至少不会因为这个业务导致其他业务不可用。

