## Rabbitmq

RabbitMQ是流行的开源消息队列系统，是AMQP（Advanced Message Queuing Protocol高级消息队列协议）的标准实现，用erlang语言开发。RabbitMQ据说具有良好的性能和时效性，同时还能够非常好的支持集群和负载部署，非常适合在较大规模的分布式系统中使用。

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

```



RabbitMQ是用Erlang语言编写的，服务器必须先安装erlang

```
wget http://erlang.org/download/otp_src_22.0.tar.gz

开始安装
tar -zxvf otp_src_20.0.tar.gz
cd otp_src_20.0
./configure --prefix=/usr/local/erlang --with-ssl --enable-threads --enable-smp-support --enable-kernel-poll --enable-hipe --without-javac
这步可能会出现提示提示缺少的组件，详情见常见问题
make && make install (ps:超慢)
ln -s /usr/local/erlang/bin/erl /usr/local/bin/
如果上步都已经完成 则可以使用了

```







####  

```
ssl : No usable OpenSSL found

wget http://www.openssl.org/source/openssl-1.0.1i.tar.gz
tar -zxf openssl-1.0.1i.tar.gz 
cd openssl-1.0.1i
./config --prefix=/home/ssl  
sed -i "s|CFLAG= |CFLAG= -fPIC |" Makefile
make && make install 


编译erlang的时候要做改动：
./configure --with-ssl=/home/ssl/ --prefix=/home/erl
```

### linux下安装mysql，make时 *** No targets specified and no makefile found. stop.

```
yum list|grep ncurses


yum -y install ncurses-devel


yum install ncurses-devel  

安装完重新执行以下命令
./configure --prefix=/usr/local/erlang --enable-hipe --enable-threads --enable-smp-support --enable-kernel-poll --without-javac
这步可能会出现提示提示缺少的组件，详情见常见问题
make && make install 
ln -s /usr/local/erlang/bin/erl /usr/local/bin/
如果上步都已经完成 则可以使用了
```


