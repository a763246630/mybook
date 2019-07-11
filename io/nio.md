NIO （同步非阻塞I/O）

**JDK 1.4**中的java.nio.*包中引入新的Java I/O库，其目的是提高速度。实际上，“旧”的I/O包已经使用NIO重新实现过，即使我们**不显式**的使用NIO编程，也能从中受益。速度的提高在文件I/O和网络I/O中都可能会发生，但本文只讨论后者。

缓冲区buffer

​    buffer是一个对象，包含了读取和写入的数据，在nio中，所有的数据都是通过缓冲区来处理的。在写入数据时，也是写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

​    **缓冲区实际是一个数组结构**，并提供了对数据结构化访问以及维护读写位置等信息。

​    8种基本类型都有相应的缓冲区：ByteBuffe、CharBuffer、 ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。他们实现了相同的接口：Buffer。

### Channel

