## 服务管理

* 查询服务端口占用

```
lsof -i:端口号 或 netstat -tunlp|grep 端口号
安装lsof  
yum install lsof   
安装 netstat 
yum install net-tools
```

* 杀进程

```
kill -9 [pid]
```

* 查看内存占用

```
free -h
```

![](/node_modules/img/free -h.png)



**CentOS7中执行**
**service iptables start/stop**
**会报错Failed to start iptables.service: Unit iptables.service failed to load: No such file or directory.**

```
在CentOS 7防火墙由firewalld来管理，

如果要添加范围例外端口 如 1000-2000
语法命令如下：启用区域端口和协议组合
firewall-cmd [--zone=<zone>] --add-port=<port>[-<port>]/<protocol> [--timeout=<seconds>]
此举将启用端口和协议的组合。端口可以是一个单独的端口 <port> 或者是一个端口范围 <port>-<port> 。协议可以是 tcp 或 udp。
实际命令如下：

添加
firewall-cmd --zone=public --add-port=80/tcp --permanent （--permanent永久生效，没有此参数重启后失效）

firewall-cmd --zone=public --add-port=1000-2000/tcp --permanent 

重新载入
firewall-cmd --reload
查看
firewall-cmd --zone=public --query-port=80/tcp
删除
firewall-cmd --zone=public --remove-port=80/tcp --permanent

当然你可以还原传统的管理方式。

执行一下命令：

systemctl stop firewalld
systemctl mask firewalld

并且安装iptables-services：
yum install iptables-services

设置开机启动：
systemctl enable iptables

systemctl stop iptables
systemctl start iptables
systemctl restart iptables
systemctl reload iptables

保存设置：
service iptables save

OK，再试一下应该就好使了



开放某个端口 在/etc/sysconfig/iptables里添加

-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```

```
nohup命令及其输出文件  

nohup ./start.sh &    默认输出到nohup.out文件

nohup ./start.sh >output 2>&1 &  指定输出到output文件

nohup /dev/null 2>&1
 1. 带&的命令行，即使terminal（终端）关闭，或者电脑死机程序依然运行（前提是你把程序递交到服务器上)； 
 2. 2>&1的意思 
　　这个意思是把标准错误（2）重定向到标准输出中（1），而标准输出又导入文件output里面，所以结果是标准错误和标准输出都导入文件output里面了。 至于为什么需要将标准错误重定向到标准输出的原因，那就归结为标准错误没有缓冲区，而stdout有。这就会导致 >output 2>output 文件output被两次打开，而stdout和stderr将会竞争覆盖，这肯定不是我门想要的. 
　　这就是为什么有人会写成： nohup ./command.sh >output 2>output出错的原因了  
最后谈一下/dev/null文件的作用，这是一个无底洞，任何东西都可以定向到这里，但是却无法打开。 所以一般很大的stdou和stderr当你不关心的时候可以利用stdout和stderr定向到这里>./command.sh >/dev/null 2>&1 

```

### 服务开机启动

#### 一.、在/etc/rc.local文件中添加自启动命令

```
执行命令： 编辑"/etc/rc.local"，添加你想开机运行的命令

运行程序脚本：然后在文件最后一行添加要执行程序的全路径。

例如，每次开机时要执行一个hello.sh，这个脚本放在/usr下面，那就可以在"/etc/rc.local"中加一行"/usr/./hello.sh"，或者" cd /opt && ./hello.sh "

注意，你的命令应该添加在：exit 0 之前
```

#### 二、在/etc/init.d目录下添加自启动脚本

linux在“/etc/rc.d/init.d”下有很多的文件，每个文件都是可以看到内容的，其实都是一些shell脚本或者可执行二进制文件
Linux开机的时候，会加载运行/etc/init.d目录下的程序，因此我们可以把想要自动运行的脚本放到这个目录下即可。系统服务的启动就是通过这种方式实现的。

#### 三、把脚本注册为服务

注册zookeeper 为服务，进入cd /etc/rc.d/init.d/，新建zkService内容如下：

```
#!/bin/sh
//是指此脚本使用/bin/sh来解释执行
#chkconfig: 2345 20 80
//2345表示系统运行级别是2，3，4或者5时都启动此服务，20，是启动的优先级，80是关闭的优先级，如果启动优先级配置的数太小时如0时，则有可能启动不成功，因为此时可能其依赖的网络服务还没有启动，从而导致自启动失败。
#description:zookeeper
#processname:zookeeper
export JAVA_HOME=/java/jdk1.8/
case $1 in
        start) su root /zookeeper/zookeeper-3.4.9/bin/zkServer.sh start;;
        stop) su root /zookeeper/zookeeper-3.4.9/bin/zkServer.sh stop;;
        status) su root /zookeeper/zookeeper-3.4.9/bin/zkServer.sh status;;
        restart) su root /zookeeper/zookeeper-3.4.9/bin/zkServer.sh restart;;
        *) echo "require start|stop|status|restart";;
esac
```

写了脚本文件之后事情还没有完，继续完成以下几个步骤：

```
chmod +x zkService　　　　　　　　 #增加执行权限 
chkconfig --add zkService 　　　 #把服务添加到系统服务列表
chkconfig zkService on 　　　　　 #设定服务的开关（on/off）
chkconfig --list myStart.s   #就可以看到已经注册了startTest的服务删除服务
chkconfig  --del myStarts.sh #删除服务
chkconfig --list                #列出所有的系统服务 
chkconfig --add httpd           #增加httpd服务 
chkconfig --del httpd           #删除httpd服务 
chkconfig --level httpd 2345 on #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list mysqld         #列出mysqld服务设置情况 
chkconfig --level 35 mysqld on  #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭 
chkconfig mysqld on             #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级 
```

