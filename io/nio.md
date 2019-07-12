## NIO （同步非阻塞I/O）

**JDK 1.4**中的java.nio._包中引入新的Java I/O库，其目的是提高速度。实际上，“旧”的I/O包已经使用NIO重新实现过，即使我们\*不显式_的使用NIO编程，也能从中受益。速度的提高在文件I/O和网络I/O中都可能会发生，但本文只讨论后者。

缓冲区buffer

​    buffer是一个对象，包含了读取和写入的数据，在nio中，所有的数据都是通过缓冲区来处理的。在写入数据时，也是写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

​    **缓冲区实际是一个数组结构**，并提供了对数据结构化访问以及维护读写位置等信息。

​    8种基本类型都有相应的缓冲区：ByteBuffe、CharBuffer、 ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。他们实现了相同的接口：Buffer。

### Channel

一个Channel（通道）代表和某一实体的连接，这个实体可以是文件、网络套接字等。也就是说，通道是Java NIO提供的一座桥梁，用于我们的程序和操作系统底层I/O服务进行交互。

通道是一种很基本很抽象的描述，和不同的I/O服务交互，执行不同的I/O操作，实现不一样，因此具体的有FileChannel、SocketChannel等。

通道使用起来跟Stream比较像，可以读取数据到Buffer中，也可以把Buffer中的数据写入通道。

* 一个通道，既可以读又可以写，而一个Stream是单向的（所以分 InputStream 和 OutputStream）
* 通道有非阻塞I/O模式

![](/assets/noi1.png)

#### 实现

Java NIO中最常用的通道实现是如下几个，可以看出跟传统的 I/O 操作类是一一对应的。

* FileChannel：读写文件
* DatagramChannel: UDP协议网络通信
* SocketChannel：TCP协议网络通信
* ServerSocketChannel：监听TCP连接

### Buffer

NIO中所使用的缓冲区不是一个简单的byte数组，而是封装过的Buffer类，通过它提供的API，我们可以灵活的操纵数据，下面细细道来。

与Java基本类型相对应，NIO提供了多种 Buffer 类型，如ByteBuffer、CharBuffer、IntBuffer等，区别就是读写缓冲区时的单位长度不一样（以对应类型的变量为单位进行读写）。

Buffer中有3个很重要的变量，它们是理解Buffer工作机制的关键，分别是

- capacity （总容量）
- position （指针当前位置）
- limit （读/写边界位置）

Buffer的工作方式跟C语言里的字符数组非常的像，类比一下，capacity就是数组的总长度，position就是我们读/写字符的下标变量，limit就是结束符的位置。

在对Buffer进行读/写的过程中，position会往后移动，而 limit 就是 position 移动的边界。由此不难想象，在对Buffer进行写入操作时，limit应当设置为capacity的大小，而对Buffer进行读取操作时，limit应当设置为数据的实际结束位置。（注意：将Buffer数据 写入 通道是Buffer 读取 操作，从通道 读取 数据到Buffer是Buffer 写入 操作）

在对Buffer进行读/写操作前，我们可以调用Buffer类提供的一些辅助方法来正确设置 position 和 limit 的值，主要有如下几个

- flip(): 设置 limit 为 position 的值，然后 position 置为0。对Buffer进行读取操作前调用。
- rewind(): 仅仅将 position
  置0。一般是在重新读取Buffer数据前调用，比如要读取同一个Buffer的数据写入多个通道时会用到。
- clear(): 回到初始状态，即 limit 等于 capacity，position 置0。重新对Buffer进行写入操作前调用。
- compact(): 将未读取完的数据（position 与 limit 之间的数据）移动到缓冲区开头，并将 position
  设置为这段数据末尾的下一个位置。其实就等价于重新向缓冲区中写入了这么一段数据。

然后，看一个实例，使用 FileChannel 读写文本文件，通过这个例子验证通道可读可写的特性以及Buffer的基本用法（注意 FileChannel 不能设置为非阻塞模式）。

### Selector

Selector（选择器）是一个特殊的组件，用于采集各个通道的状态（或者说事件）。我们先将通道注册到选择器，并设置好关心的事件，然后就可以通过调用select()方法，静静地等待事件发生。

通道有如下4个事件可供我们监听：

- Accept：有可以接受的连接
- Connect：连接成功
- Read：有数据可读
- Write：可以写入数据了

#### 为什么要用Selector

非阻塞模式下，通过Selector，我们的线程只为已就绪的通道工作，不用盲目的重试了。比如，当所有通道都没有数据到达时，也就没有Read事件发生，我们的线程会在select()方法处被挂起，从而让出了CPU资源。