### JMM

JMM（Java Memory Model）即java内存模型

了解jvm内存模型前，了解下cpu和计算机内存的交互情况。【因为Java虚拟机内存模型定义的访问操作与计算机十分相似】

#### CPU和内存的交互

在计算机中，cpu和内存的交互最为频繁，相比内存，磁盘读写太慢，内存相当于高速的缓冲区。

但是随着cpu的发展，内存的读写速度也远远赶不上cpu。因此cpu厂商在每颗cpu上加上高速缓存，用于缓解这种情况。现在cpu和内存的交互大致如下。

![](/assets/jvm1.png)

cpu上加入了高速缓存这样做解决了处理器和内存的矛盾\(一快一慢\)，但是引来的新的问题 - **缓存一致性**

在多核cpu中，每个处理器都有各自的高速缓存，而主内存确只有一个 。

```
CPU要读取一个数据时，首先从一级缓存中查找，如果没有找到再从二级缓存中查找，如果还是没有就从三级缓存或内存中查找，每个cpu有且只有一套自己的缓存。
```

如何保证多个处理器运算涉及到同一个内存区域时，多线程场景下会存在缓存一致性问题，那么运行时保证数据一致性？

为了解决这个问题，各个处理器需遵循一些协议保证一致性。【如MSI，MESI啥啥的协议。。】

![](/assets/jvm2.png)

#### 内存屏障\(Memory Barrier\)

### JVM


