### JMM

JMM（Java Memory Model）即java内存模型

了解jvm内存模型前，了解下cpu和计算机内存的交互情况。【因为Java虚拟机内存模型定义的访问操作与计算机十分相似】

#### CPU和内存的交互

在计算机中，cpu和内存的交互最为频繁，相比内存，磁盘读写太慢，内存相当于高速的缓冲区。

但是随着cpu的发展，内存的读写速度也远远赶不上cpu。因此cpu厂商在每颗cpu上加上高速缓存，用于缓解这种情况。现在cpu和内存的交互大致如下。

![](/assets/jvm1.png)

#### 内存屏障\(Memory Barrier\)

### JVM



