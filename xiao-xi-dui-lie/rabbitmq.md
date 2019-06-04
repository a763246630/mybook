## Rabbitmq

RabbitMQ是流行的开源消息队列系统，是AMQP（Advanced Message Queuing Protocol高级消息队列协议）的标准实现，用erlang语言开发。RabbitMQ据说具有良好的性能和时效性，同时还能够非常好的支持集群和负载部署，非常适合在较大规模的分布式系统中使用。

\[TOC\]

#### 安装和启动

```
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-3.7.15-1.el7.noarch.rpm

设置环境变量
# vim /etc/profile
在末尾加入以下内容：
#set RabbitMQ environment
export PAHT=$PATH:/usr/local/RabbitMQ/sbin
使环境变量生效
# source /etc/profile
启用WEB管理插件
# cd /usr/local/RabbitMQ/sbin
查看插件列表
# ./rabbitmq-plugins list
# ./rabbitmq-plugins enable rabbitmq_management
后台运行
# ./rabbitmq-server -detached

rabbitmqctl stop ：停止rabbitmq 
rabbitmq-server restart : 重启rabbitmq
好了，到这里rabbitmq已经配置好了，可以启动了：
1 我们再来查看看一下rabbitmq的默认监听端口5672
2 #netstat -tnlp|grep 5672
3 最好我们就可以在浏览器上输入http://ip:15672/登录管理界面了
4 使用登录的用户名和密码默认都是guest
添加用户和虚拟机
复制代码
添加用户：
# ./rabbitmqctl add_user username password
如：./rabbitmqctl add_user admin 123456
授权用户管理员： # ./rabbitmqctl set_user_tags admin administrator
如：./rabbitmqctl set_user_tags admin administrator
添加虚拟机： # ./rabbitmqctl add_vhost vhostname
如：./rabbitmqctl add_vhost admin_vhost
授权用户到虚拟机： # ./rabbitmqctl set_permissions -p vhostname username ".*" ".*" ".*"
如：./rabbitmqctl set_permissions -p admin_vhost admin ".*" ".*" ".*"

常见问题
erlang 和 rabbitmq 版本不匹配 decription  nopro
```

RabbitMQ是用Erlang语言编写的，服务器必须先安装erlang

```
直接使用对应的rpm 
rpm -i erlang-22.0.1-1.el7.x86_64.rpm
```

#### VirtualHost

```
RabbitMq的VirtualHost（虚拟消息服务器），每个VirtualHost相当于一个相对独立的RabbitMQ服务器；每个VirtualHost之间是相互隔离的，exchange、queue、message不能互通。 

拿数据库（用MySQL）来类比：RabbitMq相当于MySQL，RabbitMq中的VirtualHost就相当于MySQL中的一个库。

创建VirtualHost
一、命令行
rabbitmqctl add_vhost 虚拟服务器名称
例如：rabbitmqctl add_vhost my_test
```

#### 概念

* Broker：消息队列服务器实体。
* Exchange：消息交换机，指定消息按什么规则，路由到哪个队列。
* Queue：消息队列，每个消息都会被投入到一个或者多个队列里。
* Binding：绑定，它的作用是把exchange和queue按照路由规则binding起来。
* Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
* Vhost：虚拟主机，一个broker里可以开设多个vhost，用作不用用户的权限分离。
* Producer：消息生产者，就是投递消息的程序。
* Consumer：消息消费者，就是接受消息的程序。
* Channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

![](/assets/rabbitmq1.png)

* 消息队列的使用过程大概如下：
  * 消息接收
    * 客户端连接到消息队列服务器，打开一个channel。
    * 客户端声明一个exchange，并设置相关属性。
    * 客户端声明一个queue，并设置相关属性。
    * 客户端使用routing key，在exchange和queue之间建立好绑定关系。
  * 消息发布
    * 客户端投递消息到exchange。
    * exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。
* AMQP 里主要要说两个组件：
  * Exchange 和 Queue
  * 绿色的 X 就是 Exchange ，红色的是 Queue ，这两者都在 Server 端，又称作 Broker
  * 这部分是 RabbitMQ 实现的，而蓝色的则是客户端，通常有 Producer 和 Consumer 两种类型。
* Exchange通常分为四种：

  * fanout：该类型路由规则非常简单，会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，相当于广播功能，无须对消息的routingkey进行匹配操作。

  ![](/assets/fanout.png)

* direct：该类型路由规则会将消息路由到binding key与routing key完全匹配的Queue中

       ![](/assets/direct.png)

* topic：与direct类型相似，只是规则没有那么严格，可以模糊匹配和多条件匹配

* headers：该类型不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配



