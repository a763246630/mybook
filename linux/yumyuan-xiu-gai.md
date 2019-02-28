**linux ip addr 不显示ip问题**

```
1.vi /etc/systemconfig/network-scripts/ifcfg-ens33

2.ONBOOT=no 改成yes

3.systemctl restart network
```

**yum源修改**

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

