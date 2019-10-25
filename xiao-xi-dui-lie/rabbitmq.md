## Rabbitmq



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



