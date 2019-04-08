```

wget http://erlang.org/download/otp_src_19.3.tar.gz

开始安装
tar -zxvf otp_src_19.3.tar.gz
cd otp_src_19.3
./configure --prefix=/usr/local/erlang --enable-hipe --enable-threads --enable-smp-support --enable-kernel-poll --without-javac
这步可能会出现提示提示缺少的组件，详情见常见问题
make && make install (ps:超慢)
ln -s /usr/local/erlang/bin/erl /usr/local/bin/
如果上步都已经完成 则可以使用了
```

### liunx下安装mysql，make时 *** No targets specified and no makefile found. stop.

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


