## 服务管理

* 查询服务端口占用

```
lsof -i:端口号 或 netstat -tunlp|grep 端口号
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



CentOS7中执行
service iptables start/stop
会报错Failed to start iptables.service: Unit iptables.service failed to load: No such file or directory.

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
--------------------- 