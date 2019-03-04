## HTTP time out

[TOC]

#### connect  timed out  

 是`client`发出`sync`包，server端在指定的时间内没有回复`ack`导致的.没有回复`ack`的原因可能是[网络丢包](https://www.baidu.com/s?wd=%E7%BD%91%E7%BB%9C%E4%B8%A2%E5%8C%85&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)、防火墙阻止服务端返回syn的ack包等 

#### read timed out 

表示已经连接成功(即三次握手已经完成)，但是服务器没有及时返回数据(没有在设定的时间内返回数据)，导致读超时

## 1. Socket timeout

Java socket有如下两种timeout：

1. 建立连接timeout，暂时就叫 connect timeout；
2. 读取数据timeout，暂时就叫so timeout。

### 1.1 建立连接connect timeout

当不设置该参数时，指客户端请求和服务端建立tcp连接时，会一直阻塞直到连接建立成功，或抛异常。当设置了connectTimeout， 客户端请求和服务端建立连接时，阻塞时间超过connectTimeout时，就会抛出异常java.net.ConnectException: Connection timed out: connect。

我们看如下精简后的代码，首先是服务端：

```
`serverSocket = ``new` `ServerSocket(``8080``);``Socket socket = serverSocket.accept();`
```

服务端开启ServerSocket监听8080端口，再看客户端：

```
`socket = ``new` `Socket();``socket.connect(``new` `InetSocketAddress(``"localhost"``, ``8080``));``System.out.println(``"Connected."``);`
```

打印“Connected.”，修改客户端代码中的主机名为一个不存在的主机：

```
`socket = ``new` `Socket();``long` `t1 = ``0``;``try` `{``    ``t1 = System.currentTimeMillis();``    ``socket.connect(``new` `InetSocketAddress(``"www.ss.ssss"``, ``8080``));``} ``catch` `(IOException e) {``    ``long` `t2 = System.currentTimeMillis();``    ``e.printStackTrace();``    ``System.out.println(``"Connect failed, take time -> "` `+ (t2 - t1) + ``"ms."``);``}`
```

抛出异常：java.net.ConnectException: Connection timed out: connect，并打印：Connect failed, take time -> 18532ms. 也就是当未设置connect timeout时，connect方法会阻塞直到底层异常抛出。经过测试socket有个默认的超时时间，大概在20秒左右（测试的值，不一定准确，待研究JVM源码）。下面我们来设置connect timeout，再看看效果：

```
`socket = ``new` `Socket();``long` `t1 = ``0``;``try` `{``    ``t1 = System.currentTimeMillis();``    ``// 设置connect timeout 为2000毫秒``    ``socket.connect(``new` `InetSocketAddress(``"www.ss.ssss"``, ``8080``), ``2000``);``} ``catch` `(IOException e) {``    ``long` `t2 = System.currentTimeMillis();``    ``e.printStackTrace();``    ``System.out.println(``"Connect failed, take time -> "` `+ (t2 - t1) + ``"ms."``);``}`
```

抛出异常：java.net.SocketTimeoutException: connect timed out，并打印：Connect failed, take time -> 2014ms. 这里就是connect timeout发挥作用了。

### 1.2 读取数据so timeout

先看下jdk源码注释：

> Enable/disable SO_TIMEOUT with the specified timeout, in milliseconds. With this option set to a non-zero timeout, a read() call on the InputStream associated with this Socket will block for only this amount of time. If the timeout expires, a java.net.SocketTimeoutException is raised, though the Socket is still valid. The option must be enabled prior to entering the blocking operation to have effect. The timeout must be > 0. A timeout of zero is interpreted as an infinite timeout.

这个参数通过socket.setSoTimeout(int timeout)方法设置，可以看出它的意思是，socket关联的InputStream的read()方法会阻塞，直到超过设置的so timeout，就会抛出SocketTimeoutException。当不设置这个参数时，默认值为无穷大，即InputStream的read方法会一直阻塞下去，除非连接断开。

下面通过代码来看下效果：

服务端代码：

```
`serverSocket = ``new` `ServerSocket(``8080``);``Socket socket = serverSocket.accept();`
```

服务端只接受socket但不发送任何数据给客户端。客户端代码：

```
`socket = ``new` `Socket();``socket.connect(``new` `InetSocketAddress(``"localhost"``, ``8080``));``System.out.println(``"Connected."``);``in = socket.getInputStream();``System.out.println(``"reading..."``);``in.read();``System.out.println(``"read end"``);`
```

客户端建立连接就开始读取InputStream。打印：

```
`Connected.``reading...`
```

并且一直阻塞在in.read(); 上。接下来我设置so timeout，代码如下：

```
`long` `t1 = ``0``;``try` `{``    ``socket = ``new` `Socket();``    ``socket.connect(``new` `InetSocketAddress(``"localhost"``, ``8080``));``    ``// 设置so timeout 为2000毫秒``    ``socket.setSoTimeout(``2000``);``    ``System.out.println(``"Connected."``);``    ``in = socket.getInputStream();``    ``System.out.println(``"reading..."``);``    ``t1 = System.currentTimeMillis();``    ``in.read();``} ``catch` `(IOException e) {``    ``long` `t2 = System.currentTimeMillis();``    ``System.out.println(``"read end, take -> "` `+ (t2 - t1) + ``"ms"``);``    ``e.printStackTrace();``} ``finally` `{``    ``if` `(``this``.reader != ``null``) {``        ``try` `{``            ``this``.reader.close();``        ``} ``catch` `(IOException e) {``        ``}``    ``}``}`
```

抛出异常：java.net.SocketTimeoutException: Read timed out， 打印：read end, take -> 2000ms ， 说明so timeout起作用了。

### 1.3 小结

我们可以通过设置connect timeout来控制连接建立的超时时间（不是绝对的，当设置的主机名不合法，比如我设置主机名为abc，会抛异常java.net.UnknownHostException: abc，但是此时connect timeout设置是不起作用的，测试得出的结论，仅供参考）。

通过设置so timeout可以控制流读取数据的超时时间。

## 2. 使用案例

### 2.1 MySQL jdbc timeout

查阅MySQL Connector/J 5.1 Developer Guide 中的jdbc配置参数，有

```
`connectTimeout``Timeout ``for` `socket connect (in milliseconds), with ``0` `being no timeout. Only works on JDK-``1.4` `or newer. Defaults to ``'0'``.``Default: ``0``Since version: ``3.0``.``1``socketTimeout``Timeout on network socket operations (``0``, the ``default` `means no timeout).``Default: ``0``Since version: ``3.0``.``1`
```

这两个参数分别就是对应上面我们分析的connect timeout和so timeout。

参数的设置方法有两种，一种是通过url设置，

```
`jdbc:mysql:``//[host1][:port1][,[host2][:port2]]...[/[database]] [?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]`
```

即在url后面通过?加参数，比如jdbc:mysql://192.168.1.1:3306/test?connectTimeout=2000&socketTime=2000

还有一种方式是：

```
`Properties info = ``new` `Properties();``info.put(``"user"``, ``this``.username);``info.put(``"password"``, ``this``.password);``info.put(``"connectTimeout"``, ``"2000"``);``info.put(``"socketTime"``, ``"2000"``);``return` `DriverManager.getConnection(``this``.url, info);`
```

### 2.2 Jedis timeout

Jedis是最流行的redis java客户端工具，redis.clients.jedis.Jedis对象的构造器中就有参数设置，

```
`public` `Jedis(``final` `String host, ``final` `int` `port, ``final` `int` `connectionTimeout, ``final` `int` `soTimeout) {``super``(host, port, connectionTimeout, soTimeout);``}`
```

Jedis中so timeout个人觉得是有比较重要意义的，首先jedis so timeout默认值为2000毫秒，jedis的操作流程是客户端发送命令给客户端执行，然后客户端就开始执行InputStream.read()读取响应，当某个命令比较耗时（比如数据非常多的情况下执行“keys *”），而导致客户端迟迟没有收到响应，就可能导致java.net.SocketTimeoutException: Read timed out异常抛出。一般是不建议客户端执行非常耗时的命令，但是也不排除有这种特殊逻辑，那这时候就有可能需要修改Jeids中这个so timeout的值。

## 3. 总结

了解了这两个timeout之后，可以更好的处理一些网络服务的客户端和服务端，同时对排查一些问题也很有帮助。一般的成熟的网络服务和客户端都应该有这两个参数的配置方法，当使用遇到类似问题可以从这个方向去考虑下。