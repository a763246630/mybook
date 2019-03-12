### zookeeper安装配置

下载链接：http://archive.apache.org/dist/zookeeper/

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

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
tickTime=2000
dataDir=/usr/myapp/zookeeper-3.4.5/data
dataLogDir=/usr/myapp/zookeeper-3.4.5/logs
clientPort=2181
initLimit=5
syncLimit=2
server.1=192.168.220.128:2888:3888
server.2=192.168.220.129:2888:3888
server.3=192.168.220.130:2888:3888
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

对于129和130，由于安装目录都是zookeeper-3.4.5所以dataDir和dataLogDir不需要改变，又由于在不同机器上所以clientPort也不需要改变

所以此时129和130的conf/zoo.cfg的内容与128一样即可。

 

#### 3.2 data/myid文件修改

128 data/myid修改如下：

```
echo '1' > data/myid
```

129 data/myid修改如下：

```
echo '2' > data/myid
```

130 data/myid修改如下：

```
echo '3' > data/myid
```

最后使用1.4的命令把三个zookeeper都启动即可，启动顺序随意没要求。