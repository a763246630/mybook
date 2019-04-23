[TOC]

### **linux ip addr 不显示ip问题**

```
1.vi /etc/systemconfig/network-scripts/ifcfg-ens33

2.ONBOOT=no 改成yes

3.systemctl restart network
```

### **yum源修改**

```
方法一：使用wget 修改成阿里云的yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

方法二：
vim /etc/yum.repos.d/CentOS-Base.repo
修改 baseurl 和gpg 
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/ 
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

linux 开启自启动脚本

```
一、在/etc/rc.local中添加
如果不想将脚本粘来粘去，或创建链接什么的，
则:
step1. 先修改好脚本，使其所有模块都能在任意目录启动时正常执行;
step2. 再在/etc/rc.local的末尾添加一行以绝对路径启动脚本的行;
如:
$ vim /etc/rc.local
#!/bin/sh
#
touch /var/lock/subsys/local
. /etc/rc.d/rc.tune
/opt/pjt_test/test.pl

保存并退出;
再重启动测试下，则在其它的程序都启动完成后，将启动脚本;

二、1.init.d目录下都为可执行程序，他们其实是服务脚本，按照一定格式编写，Linux 在启动时会自动执行，类似Windows下的服务

2.用root帐号登录，vi /etc/rc.d/init.d/mystart，追加如下内容：
#!/bin/bash
#chkconfig:2345 80 05 --指定在哪几个级别执行，0一般指关机，
6指的是重启，其他为正常启动。80为启动的优先级，05为关闭的优先机
#description:mystart service
RETVAL=0
start(){ --启动服务的入口函数
echo -n "mystart serive ..."
cd /home/test1
su test1 -c "python /home/test1/test.py"

}

stop(){ --关闭服务的入口函数
echo "mystart service is stoped..."
}

case $1 in --使用case，可以进行交互式操作
start)
start
;;
stop)
stop
;;
esac
exit $RETVAL

3、运行chmod +r /etc/rc.d/init.d/mystart,使之可直接执行
4、运行chkconfig --add mystart,把该服务添加到配置当中
5、运行chkconfig --list mystart,可以查看该服务进程的状态
```

### shutdown 命令

```
shutdown 参数说明:

　　[-t] 在改变到其它runlevel之前﹐告诉init多久以后关机。
　
    shutdown -r now 

　　[-r] 重启计算器。

　　[-k] 并不真正关机﹐只是送警告信号给

　　每位登录者〔login〕。

　　[-h] 关机后关闭电源〔halt〕。

　　[-n] 不用init﹐而是自己来关机。不鼓励使用这个选项﹐而且该选项所产生的后果往往不总是你所预期得到的。

　　[-c] cancel current process取消目前正在执行的关机程序。所以这个选项当然没有时间参数﹐但是可以输入一个用来解释的讯息﹐而这信息将会送到每位使用者。

　　[-f] 在重启计算器〔reboot〕时忽略fsck。

　　[-F] 在重启计算器〔reboot〕时强迫fsck。

　　[-time] 设定关机〔shutdown〕前的时间。

　　**2.halt----最简单的关机命令**

　　其实halt就是调用shutdown -h。halt执行时﹐杀死应用进程﹐执行sync系统调用﹐文件系统写操作完成后就会停止内核。

　　参数说明:

　　[-n] 防止sync系统调用﹐它用在用fsck修补根分区之后﹐以阻止内核用老版本的超级块〔superblock〕覆盖修补过的超级块。

　　[-w] 并不是真正的重启或关机﹐只是写

　　wtmp〔/var/log/wtmp〕纪录。

　　[-d] 不写wtmp纪录〔已包含在选项[-n]中〕。

　　[-f] 没有调用shutdown而强制关机或重启。

　　[-i] 关机〔或重启〕前﹐关掉所有的网络接口。

　　[-p] 该选项为缺省选项。就是关机时调用poweroff。
　　
 halt   立刻关机
 poweroff 立刻关机
 shutdown -h now 立刻关机(root用户使用)
 shutdown -h 10 10分钟后自动关机
```



