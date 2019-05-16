### 简介

Redis是一个开源的使用ANSI [C语言](https://baike.baidu.com/item/C%E8%AF%AD%E8%A8%80)编写、支持网络、可基于内存亦可持久化的日志型、Key-Value[数据库](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93/103728)，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

Redis是当前比较热门的NOSQL系统之一，它是一个key-value存储系统。它支持存储的value类型相对更多，包括string、list、set、zset和hash。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作。在此基础上，Redis支持各种不同方式的排序。

[Redis](http://www.ttlsa.com/redis/)  有很多适合它解决的问题，但是也有很多并不适合它解决的问题。另外，Redis 作为内存数据库，如果用在不适合的场合，对内存的消耗是很可观的，甚至会让系统难以承受。

我们可以对系统存储使用的数据以两种角度分类，一种是按数据的大小划分，分成大数据和小数据，另一种是按数据的冷热程度划分，分成冷数据和热数据，热数据是指读或写比较频繁的数据，反之则是冷数据。

可以举一些具体的例子来说明数据的大小和冷热属性。比如网站总的注册用户数，这明显是一个小而热的数据，小是因为这个数据只有一个值，热是因为注册用户数随时间变化很频繁。再比如，用户最新访问时间数据，这是一个量比较大，冷热不均的数据，大是数据的粒度是用户级别，每一个用户都有数据，如果有一千万用户，就意味着有一千万的数据，冷热不均是因为活跃用户的最新访问时间变化很频繁，但是可能有很大一部非活跃用户访问时间长时间不会发生变化。

大体而言，Redis 最适合处理的是小而热，而且是写频繁，或者读写都比较频繁的热数据。对于大而热的数据，如果其它方式很难解决问题，也可以考虑使用 Redis 解决，但是一定要非常谨慎，防止数据无限膨胀。原因如下：

首先，对于冷数据，无论大小，都不建议放在 Redis 中。Redis 数据要全部放在内存中，资源宝贵，把冷数据放在其中实在是一种浪费，冷数据放在普通的存储比如关系数据库中就好了。

其次，对于热数据，尤其是写频繁的热数据，如果量比较小，是最适合放到 Redis 中的。比如上面提到的网站总的注册用户数，就是典型的 Redis 用做计数器的例子。再比如论坛最新发表列表，最新报名列表，可以控制数量在几百到一千的规模，也是典型的 redis 做最新列表的使用方式。

另外，对于量比较大的热数据（或者冷热不均数据），使用 Redis 时一定要比较谨慎。这种类型数据很容易引起数据膨胀，导致 Redis 消耗内存巨大，让系统难以承受。薄荷的一个惨痛教训是把用户关注（以及被关注）数据放在 Redis 中，这是一种数据量极大，冷热很不均衡的数据，在几百万的用户级别就占用了近 10 GB左右内存，让 Redis 变得难以应付。应对这种类型的数据，可以用普通存储 + 缓存的方式。

如果用对了地方，比如在小而热的数据情形，Redis 表现很棒，如果用错了地方，Redis 也会带来昂贵的代价，所以使用时务必谨慎

### Redis的安装

```
wget http://download.redis.io/releases/redis-5.0.4.tar.gz

tar -zxf redis-5.0.4.tar.gz  //解压文件

cd  redis-5.0.4

make  //执行make 对Redis解压后文件进行编译

cd src 

make install  //编译成功后，进入src文件夹，执行make install进行Redis安装
```

### Redis部署

redis.config 里bind 127.0.0.1 设置成bind 0.0.0.0  （允许所有ip连接 ）

或者注释掉

```
# Redis配置文件样例

# Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
# 启用守护进程后，Redis会把pid写到一个pidfile中，在/var/run/redis.pid
daemonize no

# 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
pidfile /var/run/redis.pid

# 指定Redis监听端口，默认端口为6379
# 如果指定0端口，表示Redis不监听TCP连接
port 6379

# 绑定的主机地址
# 你可以绑定单一接口，如果没有绑定，所有接口都会监听到来的连接
# bind 127.0.0.1

# 可以使用unix socket连接redis，这样就可以减少tcp的连接数，建立TCP需要三次握手才能建立，而断开连接则# 需要四次握手，效率低下，于是果断修改成unix socket连接redis
#
# unixsocket /tmp/redis.sock
# unixsocketperm 755

# 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 0

# 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
# debug (很多信息, 对开发／测试比较有用)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel verbose

# 日志记录方式，默认为标准输出，如果配置为redis为守护进程方式运行，而这里又配置为标准输出，则日志将会发送给/dev/null
logfile stdout

# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility.  Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# 设置数据库的数量，默认数据库为0，可以使用select <dbid>命令在连接上指定数据库id
# dbid是从0到‘databases’-1的数目
databases 16

################################ SNAPSHOTTING（快照）#################################
# Rdb快照 rdb的工作原理:    实际上就是内存快照的形式。
# 每隔N分钟或N次写操作后, 从内存dump数据形成rdb文件,压缩放在备份目录
# 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
# Save the DB on disk:
#
#   save <seconds> <changes>
# 
#
#   满足以下条件将会同步数据:
#   900秒（15分钟）内有1个更改
#   300秒（5分钟）内有10个更改
#   60秒内有10000个更改
#   Note: 可以把所有“save”行注释掉，这样就取消同步操作了
#在2个保存点之间,断电,将会丢失1-N分钟的数据 ,对于商业上面的应用，丢失的数据就是个disaster。     但是
#这个方式下 数据恢复的比较快。   建议使用 rdb 跟 aof 配合使用。
#出于对持久化的更精细要求,redis增添了aof方式 append only file

save 900 1
save 300 10
save 60 10000    

// 后台备份进程出错时,主进程停不停止写入?  主进程不停止 容易造成数据不一致
stop-writes-on-bgsave-error

// 导入rbd恢复时数据时,要不要检验rdb的完整性 验证版本是不是一致
Rdbchecksum yes   

# 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
rdbcompression yes

# 指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb

# 工作目录.
# 指定本地数据库存放目录，文件名由上一个dbfilename配置项指定
# 
# Also the Append Only File will be created inside this directory.
# 
# 注意，这里只能指定一个目录，不能指定文件名
dir ./

################################# REPLICATION（复制，冗余） ############################

# 主从复制。使用slaveof从 Redis服务器复制一个Redis实例。注意，该配置仅限于当前slave有效
# so for example it is possible to configure the slave to save the DB with a
# different interval, or to listen to another port, and so on.
# 设置当本机为slav服务时，设置master服务的ip地址及端口，在Redis启动时，它会自动从master进行数据同步
# slaveof <masterip> <masterport>


# 当master服务设置了密码保护时，slav服务连接master的密码
# 下文的“requirepass”配置项可以指定密码
# masterauth <master-password>

#当slave丢失与master的连接时，或者slave仍然在于master进行数据同步时（还没有与master保持一致），#slave可以有两种方式来响应客户端请求：
#
# 1) 如果 slave-serve-stale-data 设置成 'yes' (the default) slave会仍然响应客户端请求,此时可能会有问题。
#
# 2) 如果 slave-serve-stale data设置成  'no'  slave会返回"SYNC with master in progress"这样的错误信息。 但 INFO 和SLAVEOF命令除外。
#  but to INFO and SLAVEOF.
#
slave-serve-stale-data yes

# Health Check
# 向Master发送ping 时间间隔
# repl-ping-slave-period 10

# 主库传输大量数据或者ping相应的超时时间 默认 60秒
# repl-timeout必须大于repl-ping-slave-period
# repl-timeout 60

################################## SECURITY 安全##############################

#设置客户端连接后进行任何其他指定前需要使用的密码。
# 
#警告：因为redis速度相当快，所以在一台比较好的服务器下，一个外部的用户可以在一秒钟进行150K次的密码尝
#试，这意味着你需要指定非常非常强大的密码来防止暴力破解
#
# 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过auth <password>命令提供密码，默认关闭
# requirepass foobared

# 命令重命名.
#
# 在一个共享环境下可以重命名相对危险的命令。比如把CONFIG重名为一个不容易猜测的字符。
#
#举例:
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
# 
如果想删除一个命令，直接把它重命名为一个空字符""即可，如下：
#
# rename-command CONFIG ""

################################### LIMITS ####################################

# 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，
# 如果设置maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max Number of clients reached错误信息
# maxclients 128

# 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，
# 当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。
# Redis新的vm机制，会把Key存放内存，Value会存放在swap区
# maxmemory <bytes>

#内存清理策略：如果达到了maxmemory，你可以采取如下动作：
# volatile-lru -> 使用LRU算法来删除过期的set
# allkeys-lru -> 删除任何遵循LRU算法的key
# volatile-random ->随机地删除过期set中的key
# allkeys->random -> 随机地删除一个key
# volatile-ttl -> 删除最近即将过期的key（the nearest expire time (minor TTL)）
# noeviction -> 根本不过期，写操作直接报错
# 
# 默认策略:
#
# maxmemory-policy volatile-lru

# 对于处理redis内存来说，LRU和minor TTL算法不是精确的，而是近似的（估计的）算法。所以我们会检查某些样#本来达到内存检查的目的。默认的样本数是3，你可以修改它。 
#Redis默认的灰选择3个样本进行检测，你可以通过maxmemory-samples进行设置
# maxmemory-samples 3

############################## APPEND ONLY MODE（AOF） ###############################


# 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。 
# 开启append only模式之后，redis会把所接收到的每一次写操作请求都追加到appendonly.aof文件中，当#redis重新启动时，会从该文件恢复出之前的状态。
# 但是这样会造成appendonly.aof文件过大，所以redis还支持了BGREWRITEAOF指令，对appendonly.aof 进# # 行重新整理。
# 
# 你可以同时开启asynchronous dumps 和 AOF
appendonly no

# 指定更新日志文件名，默认为appendonly.aof
# appendfilename appendonly.aof

# Redis支持三种同步AOF文件的策略:
#
# no: 不进行同步，系统去操作 . Faster.
# always: 表示每次有写操作都进行同步. Slow, Safest.
# everysec: 表示对写操作进行累积，每秒同步一次. 
appendfsync everysec

# AOF策略设置为always或者everysec时，后台处理进程(后台保存或者AOF日志重写)会执行大量的I/O操作
# 在某些Linux配置中会阻止过长的fsync()请求。注意现在没有任何修复，即使fsync在另外一个线程进行处理
#
# 为了减缓这个问题，可以设置下面这个参数no-appendfsync-on-rewrite
# 如果该参数设置为no，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题
no-appendfsync-on-rewrite no

# AOF 自动重写
# 当AOF文件增长到一定大小的时候Redis能够调用 BGREWRITEAOF 对日志文件进行重写
#
# 它是这样工作的：Redis会记住上次进行些日志后文件的大小(如果从开机以来还没进行过重写，那日子大小在开机##的时候确定)
# 基础大小会同现在的大小进行比较。如果现在的大小比基础大小大制定的百分比，重写功能将启动
# 同时需要指定一个最小大小用于AOF重写，这个用于阻止即使文件很小但是增长幅度很大也去重写AOF文件的情况
# 设置 percentage  指当前aof文件比上次重写的增长比例大小    
auto-aof-rewrite-percentage 100
#aof文件重写最小的文件大小，即最开始aof文件必须要达到这个文件时才触发，后面的每次重写就不会根据这个变量#了(根据上一次重写完成之后的大小).此变量仅初始化启动redis有效.如果是redis恢复时，则lastSize等于初始#aof文件大小.
auto-aof-rewrite-min-size 64mb

################################## SLOW LOG ###################################

# 查询慢于设置的值会记录日志 单位为微秒
slowlog-log-slower-than 10000

# 表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列
slowlog-max-len 1024

################################ VIRTUAL MEMORY ###############################

### WARNING! Virtual Memory is deprecated in Redis 2.4
### The use of Virtual Memory is strongly discouraged.

### WARNING! Virtual Memory is deprecated in Redis 2.4
### The use of Virtual Memory is strongly discouraged.

# Virtual Memory allows Redis to work with datasets bigger than the actual
# amount of RAM needed to hold the whole dataset in memory.
# In order to do so very used keys are taken in memory while the other keys
# are swapped into a swap file, similarly to what operating systems do
# with memory pages.
# 指定是否启用虚拟内存机制，默认值为no，
# VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中
# 把vm-enabled设置为yes，根据需要设置好接下来的三个VM参数，就可以启动VM了
vm-enabled no
# vm-enabled yes

# This is the path of the Redis swap file. As you can guess, swap files
# can't be shared by different Redis instances, so make sure to use a swap
# file for every redis process you are running. Redis will complain if the
# swap file is already in use.
#
# Redis交换文件最好的存储是SSD（固态硬盘）
# 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享 
vm-swap-file /tmp/redis.swap


# 将所有大于vm-max-memory的数据存入虚拟内存，无论vm-max-memory设置多少，所有索引数据都是内存存储的（Redis的索引数据就是keys）
# 也就是说当vm-max-memory设置为0的时候，其实是所有value都存在于磁盘。默认值为0
vm-max-memory 0

# Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的数据大小来设定的。
# 建议如果存储很多小对象，page大小最后设置为32或64bytes；如果存储很大的对象，则可以使用更大的page，如果不确定，就使用默认值
vm-page-size 32

# 设置swap文件中的page数量由于页表（一种表示页面空闲或使用的bitmap）是存放在内存中的，在磁盘上每8个pages将消耗1byte的内存 
vm-pages 134217728

# 设置访问swap文件的I/O线程数，最后不要超过机器的核数，如果设置为0，那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟，默认值为4
vm-max-threads 4

############################### ADVANCED CONFIG ###############################


# 当散列的元素的键和值都小于64字节并且键值对的数量小于512时，使用压缩列表，反之使用hashtable
hash-max-zipmap-entries 512#配置元素个数最多512个
hash-max-zipmap-value 64#配置value最大为64字节

# 当列表的元素长度都小于64字节并且列表元素数量小于512时，使用压缩列表，反之使用linkedlist
list-max-ziplist-entries 512#配置元素个数最多512个
list-max-ziplist-value 64#配置value最大为64字节

# 只要集合存储的整数数量没有超过512，Redis就会使用整数集合表示以减少数据的体积。.
set-max-intset-entries 512

# 　当有序集合的元素都小于64字节并且元素数量小于128个的时候，使用压缩列表，反之使用skiplist
zset-max-ziplist-entries 128
zset-max-ziplist-value 64


# 指定是否激活重置哈希，默认为开启
activerehashing yes

################################## INCLUDES ###################################

# 指定包含其他的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各实例又拥有自己的特定配置文件
# include /path/to/local.conf
# include /path/to/other.conf
```

### redis 设置成服务 并且设置开机启动

1.设置redis.conf中daemonize为yes,确保守护进程开启,也就是在后台可以运行.\(设置为yes后,启动时好像没有redis的启动界面,不知道为什么\)

```
#vi编辑redis安装目录里面的redis.conf文件
[root@localhost /]# vi /usr/redis/redis-3.2.4/redis.conf
```

![](/assets/redis21.png)

2.复制redis配置文件\(启动脚本需要用到配置文件内容,所以要复制\)

```
//在/etc下新建redis文件夹
[root@localhost /]# mkdir /etc/redis
//把安装redis目录里面的redis.conf文件复制到/etc/redis/6379.conf里面,6379.conf是取的文件名称,启动脚本里面的变量会读取这个名称,所以要是redis的端口号改了,这里也要修改
[root@localhost redis]# cp /redis/redis-3.2.4/redis.conf /etc/redis/6379.conf
```

3.复制redis启动脚本

```
#1.redis启动脚本一般在redis根目录的utils,如果不知道路径,可以先查看路径
[root@localhost redis]# find / -name redis_init_script
/usr/redis/redis-3.2.4/utils/redis_init_script
#2.复制启动脚本到/etc/init.d/redis文件中
[root@localhost redis]# cp /usr/redis/redis-3.2.4//utils/redis_init_script /etc/init.d/redis
```

4.修改启动脚本参数

```
[root@localhost redis]# vi /etc/init.d/redis
//在/etc/init.d/redis 文件里的 !/bin/sh的下方添加
#!/bin/sh
# chkconfig: 2345 10 90  
# description: Start and Stop redis 

//继续修改 vi /etc/init.d/redis  
REDISPORT=6379
EXEC=/redis/redis-5.0.4/src/redis-server  //redis安装路径src
CLIEXEC=/redis/redis-5.0.4/src/redis-cli  //redis安装路径src
修改完成后按 Esc ,再按 :wq + Enter(回车) 保存并退出

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/redis/redis-5.0.4/redis.conf"  //修改成redis  redis.conf路径 


启动redis

打开redis命令:service redis start

关闭redis命令:service redis stop

设为开机启动:chkconfig redis on

设为开机关闭:chkconfig redis off
```

