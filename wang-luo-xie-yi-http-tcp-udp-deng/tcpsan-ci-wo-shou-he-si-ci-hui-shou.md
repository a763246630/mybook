### TCP三次握手（建立连接）

为什么是三次握手,而不是两次或多次?



![](/assets/tcp1.png)

* **序号**：seq序号，占32位，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记。

* **确认序号**: Ack序号，占32位，只有ACK标志位为1时，确认序号才有效,Ack=seq+1

* **标志位**：共6个，即URG、ACK、PSH、RST、SYN、FIN等，具体含义如下：

> （A）URG：紧急指针（urgent pointer）有效。
>
> （B）ACK：确认字符,确认序号有效。
>
> （C）PSH：接收方应该尽快将这个报文交给应用层。
>
> （D）RST：重置连接。
>
> （E）SYN：同步序列编号,发起一个新连接。
>
> （F）FIN：释放一个连接。
>
> **需要注意的是：**
>
> （A）不要将确认序号Ack与标志位中的ACK搞混了。
>
> （B）确认方Ack=发起方Req+1，两端配对。
>
> **CLOSED**：初始状态，表示TCP连接是“关闭着的”或“未打开的”。
>
> **LISTENING** （服务端状态）
>
> 表示服务器端的某个SOCKET处于监听状态，当提供的服务没有被连接时，处于LISTENING状态，端口是开放的，等待被连接。
>
> **SYN_SENT** 
>
> 客户端调用connect，发送一个SYN请求建立一个连接，在发送连接请求后等待匹配的连接请求，此时状态为SYN_SENT.
>
> **SYN_RCVD**
>
> 服务端被动打开后,接收到了客户端的SYN并且发送了ACK时的状态。再进一步接收到客户端的ACK就进入ESTABLISHED状态。
>
> **ESTABLISHED**
>
> 建立连接

所谓三次握手（Three-Way Handshake）即建立TCP连接，就是指建立一个TCP连接时，需要客户端和服务端总共发送3个包以确认连接的建立。在socket编程中，这一过程由客户端执行connect来触发，整个流程如下图所示：

![](/assets/tcp2.png)

* **第一次握手**

客户端client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN\_SENT状态，等待Server确认

* **第二次握手**

Server收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给Client以确认连接请求，Server进入SYN\_RCVD状态。

* **第三次握手**

Client收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给Server，Server检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了。

**SYN攻击：**

在三次握手过程中，Server发送SYN-ACK之后，收到Client的ACK之前的TCP连接称为半连接（half-open connect），此时Server处于SYN\_RCVD状态，当收到ACK后，Server转入ESTABLISHED状态。SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server回复确认包，并等待Client的确认，由于源地址是不存在的，因此，Server需要不断重发直至超时，这些伪造的SYN包将产时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络堵塞甚至系统瘫痪。SYN攻击时一种典型的DDOS攻击，检测SYN攻击的方式非常简单，即当Server上有大量半连接状态且源IP地址是随机的，则可以断定遭到SYN攻击了，使用如下命令可以让之现行：

```
#netstat -nap | grep SYN_RECV
```

### TCP四次挥手（终止TCP连接）

三次握手耳熟能详，四次挥手估计就少有人知道了。所谓四次挥手（Four-Way Wavehand）即终止TCP连接，就是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。在socket编程中，这一过程由客户端或服务端任一方执行close来触发，整个流程如下图所示：

![](/assets/tcp4.png)

由于TCP连接时全双工的，因此，每个方向都必须要单独进行关闭，这一原则是当一方完成数据发送任务后，发送一个FIN来终止这一方向的连接，收到一个FIN只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个TCP连接上仍然能够发送数据，直到这一方向也发送了FIN。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭，上图描述的即是如此。

**第一次挥手：**

Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN\_WAIT\_1状态。

**第二次挥手：**

Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE\_WAIT状态。

**第三次挥手：**

Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST\_ACK状态。

**第四次挥手：**

Client收到FIN后，Client进入TIME\_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

上面是一方主动关闭，另一方被动关闭的情况，实际中还会出现同时发起主动关闭的情况，具体流程如下图：

![](/assets/tcp5.png)

流程和状态在上图中已经很明了了，在此不再赘述，可以参考前面的四次挥手解析步骤。

# 结语

**关于三次握手与四次挥手通常都会有典型的面试题，在此提出供有需求的XDJM们参考：**

> \(1\) 三次握手是什么或者流程？四次握手呢？答案前面分析就是。
>
> \(2\) 为什么建立连接是三次握手，而关闭连接却是四次挥手呢？

这是因为服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

