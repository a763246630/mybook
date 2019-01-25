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



