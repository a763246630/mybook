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

