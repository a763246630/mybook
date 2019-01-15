## HTTP协议

基于TCP的文本传输协议





#### 特点

**简单快速**：客户像服务器请求服务时，只需传送请求方法和路径。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快

**灵活**：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记

**无状态**：HTTP协议是**无状态协议**。**无状态是指协议对于事务处理没有记忆能力**。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。