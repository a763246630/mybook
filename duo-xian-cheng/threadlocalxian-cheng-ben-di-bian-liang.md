## AQS 抽象队列同步器

AbstractQueueSyncronizer 由两种 等待队列 和条件队列组成

Sync同步器

state 记录上锁次数

exclusiveOwnerThread 独占锁的线程





锁 特性管理

阻塞等待队列 

共享/独占

公平/非公平

##### 可重入

多次获取锁，递归里面调用一个带锁的方法（不可重入会造成死锁）

允许中断

