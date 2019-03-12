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