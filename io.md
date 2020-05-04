传统的 IO 大致可以分为4种类型：

* InputStream、OutputStream 基于字节操作的 IO
* Writer、Reader 基于字符操作的 IO
* File 基于磁盘操作的 IO
* Socket 基于网络操作的 IO

java.net下提供的 Scoket 很多时候人们也把它归为 同步阻塞 IO ,因为网络通讯同样是 IO 行为。

java.io下的类和接口很多，但大体都是 InputStream、OutputStream、Writer、Reader 的子集，所有掌握这4个类和File的使用，是用好 IO 的关键。



阻塞IO和非阻塞IO的区别在于第一步，发起IO请求是否会被阻塞，如果阻塞直到完成那么就是传统的阻塞IO，如果不阻塞，那么就是非阻塞IO。

  


同步IO和异步IO的区别就在于第二个步骤是否阻塞，如果实际的IO读写阻塞请求进程，那么就是同步IO。

AIO的做法是，每个水壶上装一个开关，当水开了以后会提醒对应的线程去处理。  
NIO的做法是，一个线程不停的循环观察每一个水壶，根据每个水壶当前的状态去处理。  
BIO的做法是，叫一个线程停留在一个水壶那，直到这个水壶烧开，才去处理下一个水壶。

