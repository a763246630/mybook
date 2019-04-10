### zookeeper安装配置

下载链接：[http://archive.apache.org/dist/zookeeper/](http://archive.apache.org/dist/zookeeper/)

```
tar -zxf zookeeper-3.4.5.tar.gz
```

#### 配置

在主目录下创建data和logs两个目录用于存储数据和日志：

```
cd /zookeeper/zookeeper-3.4.5
mkdir data
mkdir logs
```

在conf目录下新建zoo.cfg文件，写入以下内容保存：

```
tickTime=2000
dataDir=/zookeeper/zookeeper-3.4.5/data
dataLogDir=/zookeeper/zookeeper-3.4.5/logs
clientPort=2181
```

#### 启动和停止

进入bin目录，启动、停止、重启分和查看当前节点状态（包括集群中是何角色）别执行：

```
./zkServer.sh start
./zkServer.sh stop
./zkServer.sh restart
./zkServer.sh status
```

### 集群模式

集群模式就是在不同主机上安装zookeeper然后组成集群的模式；下边以在192.168.220.128/129/130三台主机为例。

将第1.1到1.3步中安装好的zookeeper打包复制到129和130上，并都解压到同样的目录下。

#### 

#### 3.1 conf/zoo.cfg文件修改

三个zookeeper的conf/zoo.cfg修改如下：

```
tickTime=2000
dataDir=/usr/myapp/zookeeper-3.4.5/data
dataLogDir=/usr/myapp/zookeeper-3.4.5/logs
clientPort=2181
initLimit=5
syncLimit=2
server.1=192.168.175.131:2888:3888
server.2=192.168.175.132:2888:3888
server.3=192.168.175.133:2888:3888
表单server.X的条目列出构成ZooKeeper服务的服务器。当服务器启动时，它通过查找数据目录中的文件myid来知道它是哪个服务器 。该文件包含服务器编号，以ASCII格式显示。

最后，请注意每个服务器名称后面的两个端口号：“2888”和“3888”。对等体使用前端口连接到其他对等体。这样的连接是必要的，使得对等体可以进行通信，例如，以商定更新的顺序。更具体地说，一个ZooKeeper服务器使用这个端口来连接追随者到领导者。当新的领导者出现时，追随者使用此端口打开与领导者的TCP连接。因为默认领导选举也使用TCP，所以我们目前需要另外一个端口进行领导选举。这是服务器条目中的第二个端口。
```

对于131和132，由于安装目录都是zookeeper-3.4.5所以dataDir和dataLogDir不需要改变，又由于在不同机器上所以clientPort也不需要改变

所以此时131和132的conf/zoo.cfg的内容与133一样即可。

防火墙开启端口

```
firewall-cmd --zone=public --add-port=2181/tcp --permanent
[root@localhost zookeeper-3.4.11]# firewall-cmd --zone=public --add-port=2888/tcp --permanent
success
[root@localhost zookeeper-3.4.11]# firewall-cmd --zone=public --add-port=3888/tcp --permanent
success
[root@localhost zookeeper-3.4.11]# systemctl restart firewalld
```

#### 3.2 data/myid文件修改

131 data/myid修改如下：

```
 执行  echo '1' > data/myid
```

132 data/myid修改如下：

```
 执行  echo '2' > data/myid
```

133 data/myid修改如下：

```
  执行  echo '3' > data/myid
```

最后使用1.4的命令把三个zookeeper都启动即可，启动顺序随意没要求。

#### 4.查看是否成功

131

```
[root@localhost zookeeper-3.4.11]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.11/bin/../conf/zoo.cfg
Mode: follower
```

132

```
[root@localhost zookeeper-3.4.11]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.11/bin/../conf/zoo.cfg
Mode: leader
```

133

```
[root@localhost zookeeper-3.4.11]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.11/bin/../conf/zoo.cfg
Mode: follower
```

其实也可以查看启动过程

```
zkServer.sh start-foreground
```



