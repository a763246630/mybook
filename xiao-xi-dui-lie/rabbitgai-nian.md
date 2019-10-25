RabbitMQ是流行的开源消息队列系统，是**AMQP（Advanced Message Queuing Protocol高级消息队列协议）**的标准实现，用erlang语言开发。RabbitMQ据说具有良好的**性能和时效性**，同时还能够非常好的支持**集群和负载部署**，非常适合在较大规模的分布式系统中使用。

为什么选择rabbitmq？

1)**开源，性能好，稳定性保证**, 

2)**提供了消息的可靠性投递（**confirm**），返回模式** 

3)**与**sping amqp **整合和完美，提供丰富的**api 

4)**集群模式十分丰富**(HA**模式 镜像队列模型**) 

5)保证数据不丢失的情况下，保证很好的性能 

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

  * fanout：不需要RouteKey，需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

    适用场景：

    第一：大型玩家在玩在线游戏的时候，可以用它来广播重大消息。这让我想到电影微微一笑很倾城中，有款游戏需要在世界上公布玩家重大消息，也许这个就是用的MQ实现的。这让我不禁佩服肖奈，人家在大学的时候就知道RabbitMQ的这种特性了。

    第二：体育新闻实时更新到手机客户端。

    第三：群聊功能，广播消息给当前群聊中的所有人。

  * direct：该类型不需要Exchange进行绑定，**消息发送时需要RouteKey，Exchange收到消息后会转发RouteKey对应的Queue中**,如果vhost中不存在RouteKey中指定的队列名，则该消息会被抛弃。

    适用场景：

    这种类型的Exchange，通常是将同一个message以一种循环的方式分发到不同的Queue，即不同的消费者手中，使用这种方式，值得注意的是message在消费者之间做了一个均衡，而不是说message在Queues之间做了均衡。

  * topic：与direct类型相似，只是规则没有那么严格，可以模糊匹配和多条件匹配

    使用场景：

    新闻的分类更新

    同意任务多个工作者协调完成

    同一问题需要特定人员知晓

    Topic Exchange的使用场景很多，我们公司就在使用这种模式，将足球事件信息发布，需要使用这些事件消息的人只需要绑定对应的Exchange就可以获取最新消息。

  * headers：该类型不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配

性能排序：fanout &gt; direct &gt;&gt; topic。比例大约为11：10：6

##### 死信队列介绍

* 死信队列：DLX，`dead-letter-exchange`

* 利用DLX，当消息在一个队列中变成死信`(dead message)`之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX

##### 消息变成死信有以下几种情况

* 消息被拒绝\(basic.reject / basic.nack\)，并且requeue = false

* 消息TTL过期

* 队列达到最大长度

**过期消息：**

在 rabbitmq 中存在2种方可设置消息的过期时间，第一种通过对队列进行设置，这种设置后，该队列中所有的消息都存在相同的过期时间，第二种通过对消息本身进行设置，那么每条消息的过期时间都不一样。如果同时使用这2种方法，那么以过期时间小的那个数值为准。当消息达到过期时间还没有被消费，那么那个消息就成为了一个 死信 消息。

**队列设置：**在队列申明的时候使用 x-message-ttl 参数，单位为 毫秒

**单个消息设置：**是设置消息属性的 expiration 参数的值，单位为 毫秒

**延时队列：**在rabbitmq中不存在延时队列，但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

| arguments键 | 值 | 意义 |
| :--- | :--- | :--- |
| x-message-ttl | 数字类型，标志时间，以豪秒为单位 | 标志队列中的消息存活时间，也就是说队列中的消息超过了制定时间会被删除 |
| x-expires | 数字类型，标志时间，以豪秒为单位 | 队列自身的空闲存活时间，当前的queue在指定的时间内，没有consumer、basic.get也就是未被访问，就会被删除。 |
| x-max-length和x-max-length-bytes | 数字 | 最大长度和最大占用空间，设置了最大长度的队列，在超过了最大长度后进行插入会删除之前插入的消息为本次的留出空间,相应的最大占用大小也是这个道理，当超过了这个大小的时候，会删除之前插入的消息为本次的留出空间。 |
| x-dead-letter-exchange和x-dead-letter-routing-key | 字符串 | 消息因为超时或超过限制在队列里消失，这样我们就丢失了一些消息，也许里面就有一些是我们做需要获知的。而rabbitmq的死信功能则为我们带来了解决方案。设置了dead letter exchange与dead letter routingkey（要么都设定，要么都不设定）那些因为超时或超出限制而被删除的消息会被推动到我们设置的exchange中，再根据routingkey推到queue中 |
| x-max-priority | 数字 | 队列所支持的优先级别，列如设置为5，表示队列支持0到5六个优先级别，5最高，0最低，当然这需要生产者在发送消息时指定消息的优先级别，消息按照优先级别从高到低的顺序分发给消费者 |
| alternate-exchange |  | 下面简称AE，当一个消息不能被route的时候，如果exchange设定了AE，则消息会被投递到AE。如果存在AE链，则会按此继续投递，直到消息被route或AE链结束或遇到已经尝试route过消息的AE。 |

**延时队列：**在rabbitmq中不存在延时队列，但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

##### 死信处理过程

* DLX也是一个正常的Exchange，和一般的Exchange没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性。

* 当这个队列中有死信时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列。

* 可以监听这个队列中的消息做相应的处理。

##### 死信队列设置

1. 首先需要设置死信队列的exchange和queue，然后进行绑定：

```
Exchange: dlx.exchange


Queue: dlx.queue


RoutingKey: #


#表示只要有消息到达了Exchange，那么都会路由到这个queue上
```

1. 然后需要有一个监听，去监听这个队列进行处理

2. 然后我们进行正常声明交换机、队列、绑定，只不过我们需要在队列加上一个参数即可：`arguments.put(" x-dead-letter-exchange"，"dlx.exchange");`，这样消息在过期、requeue、 队列在达到最大长度时，消息就可以直接路由到死信队列！

此时关闭消费端，然后启动生产端，查看管控台队列的消息情况，`test_dlx_queue`的值为1，而`dlx_queue`的值为0。10s后的队列结果如图，由于生产端发送消息时指定了消息的过期时间为10s，而此时没有消费端进行消费，消息便被路由到死信队列中。

生产者 --&gt; 消息 --&gt; 交换机 --&gt; 队列 --&gt; 变成死信 --&gt; DLX交换机 --&gt;队列 --&gt; 消费者

* 声明了一个direct模式的exchange。

* 声明了一个死信队列deadLetterQueue，该队列配置了一些属性`x-dead-letter-exchange`表明死信交换机，`x-dead-letter-routing-key`表明死信路由键，因为是direct模式，所以需要设置这个路由键。

* 声明了一个替补队列redirectQueue，变成死信的消息最终就是存放在这个队列的。

* 声明绑定关系，分别是死信队列以及替补队列和交换机的绑定。



