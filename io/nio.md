NIO （同步非阻塞I/O）

**JDK 1.4**中的java.nio.*包中引入新的Java I/O库，其目的是提高速度。实际上，“旧”的I/O包已经使用NIO重新实现过，即使我们**不显式**的使用NIO编程，也能从中受益。速度的提高在文件I/O和网络I/O中都可能会发生，但本文只讨论后者。

缓冲区buffer

​    buffer是一个对象，包含了读取和写入的数据，在nio中，所有的数据都是通过缓冲区来处理的。在写入数据时，也是写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

​    **缓冲区实际是一个数组结构**，并提供了对数据结构化访问以及维护读写位置等信息。

​    8种基本类型都有相应的缓冲区：ByteBuffe、CharBuffer、 ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。他们实现了相同的接口：Buffer。

### Channel

一个Channel（通道）代表和某一实体的连接，这个实体可以是文件、网络套接字等。也就是说，通道是Java NIO提供的一座桥梁，用于我们的程序和操作系统底层I/O服务进行交互。

通道是一种很基本很抽象的描述，和不同的I/O服务交互，执行不同的I/O操作，实现不一样，因此具体的有FileChannel、SocketChannel等。

通道使用起来跟Stream比较像，可以读取数据到Buffer中，也可以把Buffer中的数据写入通道。

- 一个通道，既可以读又可以写，而一个Stream是单向的（所以分 InputStream 和 OutputStream）
- 通道有非阻塞I/O模式

#### 实现

Java NIO中最常用的通道实现是如下几个，可以看出跟传统的 I/O 操作类是一一对应的。

- FileChannel：读写文件
- DatagramChannel: UDP协议网络通信
- SocketChannel：TCP协议网络通信
- ServerSocketChannel：监听TCP连接