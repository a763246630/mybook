# 安装并配置git

## linux

- yum安装git

```
yum install -y git
```

-  yum卸载git

```
yum remove git
```

- 安装包安装git

```
$ wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.4.tar.gz
```

```
$ tar -zxf git-1.7.2.2.tar.gz
```

```
$ cd git-1.7.2.2
```

```
$ make prefix=/usr/local all
```

```
$ make prefix=/usr/local install
```

- 配置环境变量 

```
vim /etc/profile 最后加上
```

## windows

- git 目录下的 bin（如 C:\Program Files (x86)\Git\bin ）添加到 PATH 环境变量。

```
whereis git //查询git 目录
```

- git  配置sshkey


```
cd /root/.ssh
```

```
ssh-keygen -t rsa -b 4096 -C “your_email@example.com” 然后回车三连击…
```

```
ssh-keygen -t rsa -b 4096 -C “763246630@qq.com“
```

- 可以看到当前目录下多出两个文件 id_rsa.pub 和 id_rsa 带后缀是公钥，不带是私钥


- vim id_rsa.pub 打开公钥，将全部内容复制(私钥别动)