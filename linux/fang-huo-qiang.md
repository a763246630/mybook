iptables 和 firewalls 两种安全策略

防火墙会从上至下的顺序来读取配置的策略规则，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。一般而言，防火墙策略规则的设置有两种：一种是“通”（即放行），一种是“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

iptables服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

> 在进行路由选择前处理数据包（PREROUTING）；
>
> 处理流入的数据包（INPUT）；
>
> 处理流出的数据包（OUTPUT）；
>
> 处理转发的数据包（FORWARD）；
>
> 在进行路由选择后处理数据包（POSTROUTING）
>
> 一般来说，从内网向外网发送的流量一般都是可控且良性的，因此我们使用最多的就是INPUT规则链，该规则链可以增大黑客人员从外网入侵内网的难度。

1）Netfilter：数据包过滤机制 
2）TCP Wrappers：程序管理机制

iptables 通过ip和端口限制访问

查看端口是否可访问：telnet ip 端口号 （如本机的35465：telnet localhost 35465）

开放的端口位于/etc/sysconfig/iptables中

查看时通过 more /etc/sysconfig/iptables 命令查看

查看端口是否可访问：telnet ip 端口号 （如本机的35465：telnet localhost 35465）

开放的端口位于/etc/sysconfig/iptables中

查看时通过 more /etc/sysconfig/iptables 命令查看

![](/assets/fhq1.png)

如果想开放端口（如：8889）

（1）通过vi /etc/sysconfig/iptables 进入编辑增添一条-A INPUT -p tcp -m tcp --dport 8889 -j ACCEPT 即可

（2）执行 /etc/init.d/iptables restart 命令将iptables服务重启

\#（3）保存 /etc/rc.d/init.d/iptables save

![](/assets/fhq2.png)

注：如若不想修改iptables表，可以直接输入下面命令：

\#**iptables -I INPUT -p tcp --dport 8889 -j ACCEPT**

若/etc/sysconfig/iptables不存在，

原因：在新安装的[linux](http://lib.csdn.net/base/linux)系统中，防火墙默认是被禁掉的，一般也没有配置过任何防火墙的策略，所有不存在/etc/sysconfig/iptables文件。

解决：

1. 在控制台使用iptables命令随便写一条防火墙规则，如：iptables -P OUTPUT ACCEPT
2. 使用service iptables save进行保存，默认就保存到了/etc/sysconfig目录下的iptables文件中 

1、iptables命令格式

```
 iptables \[-t 表\] －命令 匹配 操作 （大小写敏感）
```

动作选项

```
 ACCEPT          接收数据包




 DROP             丢弃数据包




 REDIRECT      将数据包重新转向到本机或另一台主机的某一个端口，通常功能实现透明代理或对外开放内网的某些服务




 SNAT             源地址转换




 DNAT             目的地址转换




 MASQUERADE       IP伪装




 LOG               日志功能
```

2、定义规则

①先拒绝所有的数据包，然后再允许需要的数据包

```
  iptalbes -P INPUT DROP




  iptables -P FORWARD DROP




  iptables -P OUTPUT ACCEPT
```

②查看nat表所有链的规则列表

```
  iptables -t nat -L
```

③增加，插入，删除和替换规则

```
 iptables \[-t 表名\]
```

&lt;

-A\|I\|D\|R

&gt;

链名 \[规则编号\] \[-i\|o 网卡名称\] \[-p 协议类型\] \[-s 源ip\|源子网\] \[--sport 源端口号\] \[-d 目的IP\|目标子网\] \[--dport 目标端口号\] \[-j 动作\]

```
参数：-A 增加




           -I 插入




           -D 删除




           -R 替换
```

三、例子

①iptables -t filter -A INPUT -s 192.168.1.5 -i eth0 -j DROP

禁止IP为192.168.1.5的主机从eth0访问本机②iptables -t filter -I INPUT 2 -s 192.168.5.0/24 -p tcp --dport 80 -j DROP

禁止子网192.168.5.0访问web服务③iptables -t filter -I INPUT 2 -s 192.168.7.9 -p tcp --dport ftp -j DROP

禁止IP为192.168.7.9访问FTP服务

④iptables -t filter -L INPUT

查看filter表中INPUT链的规则

⑤iptables -t nat -F

删除nat表中的所有规则

⑥iptables -I FORWARD -d wwww.playboy.com -j DROP

禁止访问

[www.playboy.com](http://www.playboy.com/)

网站

⑦iptables -I FORWARD -s 192.168.5.23 -j DROP

禁止192.168.5.23上网

防火墙是整个数据包进入主机前的第一道关卡。防火墙主要通过Netfilter与TCPwrappers两个机制来管理的。  
1）Netfilter：数据包过滤机制  
2）TCP Wrappers：程序管理机制  
关于数据包过滤机制有两个软件：firewalld与iptables

![](/assets/fhq3.png)

2 iptables通过控制端口来控制服务，而firewalld则是通过控制协议来控制端口  
我们这里先对firewalld做实验。Iptables和firewalld只能开一个。  
在学习之前先对iptables firewalld 内核之间的关系有一个了解。

![](/assets/fhq4.png)

（一） 
（1）firewalld

安装：
[root@route ~]# yum install firewalld
[root@route ~]# systemctl start firewalld
[root@route ~]# systemctl status firewalld
firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled)
   Active: active (running) since 日 2017-12-03 21:12:02 EST; 49s ago
 Main PID: 2362 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─2362 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

12月 03 21:12:02 route systemd[1]: Started firewalld - dynamic firewall daemon.
12月 03 21:12:45 route systemd[1]: Started firewalld - dynamic firewall daemon.
[root@route ~]# systemctl enable firewalld
[root@route ~]# firewall-cmd --state 

关闭firewalld  使用iptables

 systemctl stop firewalld

 systemctl mask firewalld

 systemctl enable iptables

service iptables save

