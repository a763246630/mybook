### Zookeeper常用命令

zk服务命令

```
启动ZK服务: sh bin/zkServer.sh start

查看ZK服务状态:sh bin/zkServer.sh status

停止ZK服务:sh bin/zkServer.sh stop

重启ZK服务:sh bin/zkServer.sh restart 

连接服务器:sh zkCli.sh -server 127.0.0.1:2181
```

命令行工具的一些常用操作命令如下：

```
查看根目录节点： ls /

查看根节点状态：stat /

查看根节点数据和详情：get /

显示客户所支持的所有命令 help

连接zk服务端，与close命令配合使用可以连接或者断开zk服务端 connect 127.0.0.1:2181

获取路径下的节点信息  ls /test

设置节点的数据  set /test "test1"

删除节点命令，此命令与delete命令不同的是delete不可删除有子节点的节点，但是rmr命令可以删除，注意路径为
绝对路径  rmr /test/test1

删除配额，-n为子节点个数，-b为节点数据长度  delquota –n 2

退出 quit

设置和显示监视状态，on或者off。 printwatches on

创建节点，其中-s为顺序充点，-e临时节点 create /zookeeper/node1"test_create"

断开客户端与服务端的连接 close

ls2为ls命令的扩展，比ls命令多输出本节点信息  ls2 /test

列出最近的历史命令 history

显示配额 如listquota /zookeeper /zookeeper节点个数限额为2，长度无限额。

setAcl 设置节点Acl,此处重点说一下acl，acl由大部分组成：1为scheme，2为user，3为permission，一般情况下表示为scheme:id:permissions  

scheme和id
world: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的

auth: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)

digest: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication

ip: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段

super: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

permissions
CREATE(c): 创建权限，可以在在当前node下创建child node

DELETE(d): 删除权限，可以删除当前的node

READ(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes

WRITE(w): 写权限，可以向当前node写数据

ADMIN(a): 管理权限，可以设置当前node的permission

综上，一个简单使用setAcl命令，则可以为：

setAcl /zookeeper/node1 world:anyone:cdrw

 getAcl命令
获取节点Acl。

如getAcl /zookeeper/node1

'world,'anyone

: cdrwa

注：可参见setAcl命令。

sync命令
强制同步。

如sync /zookeeper

由于请求在半数以上的zk server上生效就表示此请求生效，那么就会有一些zk server上的数据是旧的。sync命令就是强制同步所有的更新操作。

redo命令
再次执行某命令。

如redo 10

其中10为命令ID，需与history配合使用。

addauth命令
节点认证。

如addauth digest username:password，可参见setAcl命令digest处。

使用方法：

一、通过setAcl设置用户名和密码

setAcl pathdigest:username:base64(sha1(password)):crwda

二、认证

addauth digest username:password

delete命令
删除节点。

如delete /zknode1

setquota命令
设置子节点个数和数据长度配额。

如setquota –n 4 /zookeeper/node 设置/zookeeper/node子节点个数最大为4

setquota –b 100 /zookeeper/node 设置/zookeeper/node节点长度最大为100



1.ls -- 查看某个目录包含的所有文件，例如：
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /

2.ls2 -- 查看某个目录包含的所有文件，与ls不同的是它查看到time、version等信息，例如：
[zk: 127.0.0.1:2181(CONNECTED) 1] ls2 /

3.create -- 创建znode，并设置初始内容，例如：
[zk: 127.0.0.1:2181(CONNECTED) 1] create /test "test" 
Created /test

创建一个新的 znode节点“ test ”以及与它关联的字符串

4.get -- 获取znode的数据，如下：
[zk: 127.0.0.1:2181(CONNECTED) 1] get /test

5.set -- 修改znode内容，例如：
[zk: 127.0.0.1:2181(CONNECTED) 1] set /test "ricky"

6.delete -- 删除znode，例如：
[zk: 127.0.0.1:2181(CONNECTED) 1] delete /test

7.quit -- 退出客户端

8.help -- 帮助命令
```

