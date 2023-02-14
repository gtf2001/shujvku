# Redis

## 一、NoSQL

NoSQL，泛指非关系型数据库，有时Not only SQL的缩写，是对不同于传统类型的关系型数据库的统称。

- SQL(Structured Query Language)数据数据，指关系型数据库。主要代表：SQL Server、MySQL、Oracle。存储数据时，需要预先定义表。---字段---关系
- NoSQL：泛指非关系型数据库。主要代表：MongoDB、Redis



### SQL和NoSQL对比

- SQL通常以数据库表的形式存储数据。举个栗子，学生借书的数据：

| 学号 | 姓名 | 书名               | 时间                   |
| ---- | ---- | ------------------ | ---------------------- |
| 1    | 熊大 | 《如何种树》       | 2022年11月11日14:37:48 |
| 2    | 熊二 | 《母猪的产后护理》 | 2022年11月11日14:37:56 |
| 3    | 袁菲 | 《如何产后被护理》 | 2022年11月11日14:37:58 |

- 而NoSQL存储方式比较灵活，比如使用类JSON文件存储上表中的数据

~~~json
{
    借阅：[
    {
        学号:1,
        姓名:熊大,
        书名:《如何种树》,
        时间:2022年11月11日14:37:48
    },
    {
        ......
    }
    ]，
    学生信息:[]
}
~~~

### 关系型数据库瓶颈

- 高并发读写需求

  ~~~markdown
  针对网站类用户的并发性访问非常高，而一台数据库的最大连接数有限，且磁盘I/O有限，不能满足多人链接
  ~~~

- 海量数据的高效率读写

  ~~~markdown
  个别网站每天产生的数据量是巨大的，对于关系型数据库来说，在一张包含海量数据的数据库表中进行查询，效率是非常低的
  ~~~

### 数据库的选择

~~~markdown
# 项目开发中，是使用Mysql还是Redis呢？
	一般项目会使用A+B的模式进行开发，数据大多数都存在mysql中，如果对某些数据由比较频繁的读写时，可以将这部分的数据存储在Redis中。
~~~

## 二、 Redis简介

### 1、 Redis是什么

~~~markdown
Redis是一个基于内存运行的、速度非常快的单线程NoSQL数据库，基于Key-Value的格式进行数据存储，同时支持数据持久化到硬盘。
~~~

### 2、 Redis的特点

~~~markdown
1. 高性能：读写速度很快
2. 数据类型丰富：key-value的格式存储数据，key一般是字符串，value可以有五种类型
3. 基于内存存储，支持持久化（将内存中的数据保存到硬盘中）/持久化机制（AOF/RDB）
~~~

## 三、安装Redis

### 1、 下载并且解压

~~~markdown
1. wget https://download.redis.io/releases/redis-3.0.7.tar.gz
2. 解压：[root@10 app]# tar -zxvf redis-3.0.7.tar.gz -C /usr/local/
~~~

### 2、 make命令

~~~markdown
cd指令，进入到解压目录中，然后执行指令：make
~~~

- 执行make的时候，如果出现异常

  - make[2]：cc：Command not found

    ~~~markdown
    没有安装gcc
    解决方案：[root@10 redis-3.0.7]# yum install gcc
    ~~~

  - zmalloc.h:51:31: error: jemalloc/jemalloc.h: No such file or directory

    ~~~markdown
    异常原因：一些编译依赖或者原来编译遗留出现的问题
    解决方案：make disclean 清理一下，然后再make
    作业：研究clean和disclean的区别
    ~~~

### 3、 make test

在make成功之后，执行指令make test

- make test的时候出现异常

  - 异常一：

    ~~~markdown
    需要安装tcl
    yum install -y tcl
    ~~~

  - 如果yum不能用

    ~~~markdown
    需要修改以下文件的第一行：
    vi /usr/bin/yum
    vi /usr/libexec/urlgrabber-ext-down 
    
    在这两个文件夹中第一行，改成：
    #! /usr/bin/python2.7.5
    ~~~

### 4.  make install 

~~~markdown
安装，执行指令：make install 
~~~

### 5、 启动服务

- 第一种：前台模式

~~~markdown
直接执行指令：redis-server
~~~

- 第二种：守护进程模式，指定配置文件启动

  - 在redis的解压目录中，找到配置文件模板(redis.conf)，复制到如下位置

  ~~~markdown
  cp redis.conf /usr/local/redis-3.0.7/redis.conf
  ~~~

  - 通过vi命令进行修改

  ~~~markdown
  daemonize yes # 守护进程模式启动--后台
  
  # 以下四个重要配置，不改也行
  port 6379 # 端口
  logfile ./redis.log # 日志文件的存储位置
  pidfile ./redis.pid # 进程id的存储位置
  dir /usr/local/ridis_conf/ # 工作目录 rdb、aof文件的存储位置

- 然后执行redis-server redis.conf

### 6、 链接redis

执行指令：

- redis-cli：链接的是端口为6379，Host为127.0.0.1（localhost）的redis服务器
- redis-cli -p 6379 -h 192.168.0.20 连接端口为6379 Host为192.168.0.20的redis服务器

- 关闭redis服务
  - 127.0.0.1:6379> shutdown
  - 此时给redis服务器发送了一个shutdown命令，会关闭Redis的服务
- 关闭服务
  - redis-cli -p 6379 shutdown
- 关闭服务   -kill 进程号
  - [root@10 redis-3.0.7]# ps -ef | grep redis 查看进程ID
  - kill -9 进程号

### 7、 关闭服务

redis-cli连接了redis服务器后，可以通过shutdown指令关闭连接并且关闭服务

或者redis-cli -p 6379 shutdown

如果你只想关闭连接(客户端)，在redis命令中，crtl+c即可，此时服务不会关闭。

### 8、 redis的使用介绍

~~~markdown
1. redis默认有16个数据库，分别是0-15，默认最开始使用的是0号库
2. redis中可以切换数据库
	select 1
	切换到redis的1号库

# redis中操作库的命令
	- 清空当前的库：FLUSHDB
	- 清空所有的库：FLUSHALL
~~~



## 四、 Redis的数据类型（逢面试必考）

Redis支持五种数据类型：String（字符串）、list（列表）、set（集合）及zset（有序集合）、hash（哈希）

### 1、 String字符串

存储单值

| 命令        | 说明                                                 | 示例                       |
| ----------- | ---------------------------------------------------- | -------------------------- |
| set         | 设置一个key/value                                    | set name xx                |
| get         | 根据key获得对应的value                               | get name                   |
| mset        | 一次设置多个key value                                | mset age 18 salary 300000  |
| mget        | 一次获得多个key value                                | mget name age salary       |
| getset      | 获得原始的key的值，同时设置新值                      | getset age 20              |
| del         | 删除key-value                                        | del name                   |
| strlen      | 获得对应的key存储的value的长度                       | strlen name                |
| append      | 为对应的key的value拼接内容                           | append name hahaha         |
| getrange    | 截取value的内容，对原始的值没有影响                  | getrange name 0 2          |
| setex       | 设置一个key-value的存活的有效期（秒 ）               | setex name 10 tom          |
| psetex      | 设置一个key-value的存活的有效期（毫秒）              | psetex course 10000 redis  |
| setnx       | 只有当这个key不存在的时候，等效于set操作             | setnx birth 2023           |
| msetnx      | 可以同时设置多个key，在多个key都不存在的情况下生效。 | msetnx course mysql123 p 5 |
| decr        | 进行数值类型的-1操作                                 | decr age                   |
| incr        | 进行数值类型的+1操作                                 | incr age                   |
| decrby      | 根据提供的数据进行减法运算                           | decrby age 10              |
| incrby      | 根据提供的数据进行加法运算                           | incrby age 10              |
| incrbyfloat | 根据提供的数据加入浮点数                             | incrbyfloat age 1.5        |

### 2、 List列表

存储多值

| 命令    | 说明                                                         | 示例                               |
| ------- | ------------------------------------------------------------ | ---------------------------------- |
| lpush   | 将某个值加入到一个key的列表的头部(left push)                 | lpush user fzy                     |
| lpushx  | 将某个值加入到一个key的列表的头部(left push) 必须要保证这个key存在 | lpushx users yf                    |
| rpush   | 将某个值加入到一个key的列表的尾部(right push)                | rpush user fzy                     |
| rpushx  | 将某个值加入到一个key的列表的尾部(right push) 必须要保证这个key存在 | rpushx users yf                    |
| lpop    | 返回并且删除列表的第一个元素                                 | lpop user                          |
| rpop    | 返回并且删除列表的最后一个元素                               | rpop user                          |
| lrange  | 获取某一个下标区间内的元素                                   | lrange user 0 10/lrange user -5 -1 |
| llen    | 获取列表的长度                                               | llen user                          |
| lset    | 设置某一个元素的值                                           | lset user 2 Tonny                  |
| lindex  | 获取某一个位置的元素                                         | lindex user 3                      |
| lrem    | 从列表头开始，删除对应个数的指定元素                         | lrem user 10 yf                    |
| ltrim   | 保留列表中的特定区间内的元素，将其他元素都删除               | ltrim user 1 2                     |
| linsert | 在某一个元素之前、之后插入新元素                             | linsert user after CGK LWT         |

### 3、 set集合

set集合是一个无序集合，并且不允许有相同元素





## 五、 Redis持久化【重点】

### 1. 什么是持久化？

持久化的含义就是指把内存中的数据保存到可永久存储的设备中（磁盘），以便数据可以重用

- 存：读内存中Redis数据--->通过某种持久化的方式--->存储到磁盘中
- 取：读磁盘中的数据--->读取在内存中

redis是内存数据库，速度快的同时也对数据的安全性产生了问题，redis进程爆炸，redis数据库里的数据会丢失。

为了解决这个问题，redis提供了持久化功能：

- RDB持久化：快照，将Redis在内存中的数据库，定时dump到磁盘上

- AOF持久化：append-only file-redis的操作日志，以追加的形式写入文件

将内存中的数据写入硬盘中，当redis重启后，可以从硬盘中恢复数据。

### 2. RDB持久化

~~~markdown
在某一时刻，将redis服务器中内存中的所有数据写入到磁盘中，默认已经开启
~~~

#### 2.1 RDB开发步骤

- 客户端触发

~~~markdown
BGSAVE和SAVA指令可以主动触发快照（RDB)持久化。
主动结束redis服务时，也会触发RDB持久化。
~~~

- 自动触发

编辑redis.conf文件

~~~markdown
save 900 1		# 900秒内超过1个key被修改，会触发快照保存
save 300 10		# 300秒内超过10个key被修改，会触发快照保存
save 60 10000	# 60s内超过10000个key被修改，会触发快照保存

dbfilename dump.rdb		# 快照的文件名
stop-writes-on-bgsave-error yes	# 快照失败后是否继续写操作
rdbcompression yes		# 是否压缩快照文件
~~~

#### 2.2 RDB的运行原理

- 在某些时刻（满足rdb持久化的时刻），Redis会通过fork产生一个子进程，一个父进程的内存空间的快照（副本），其中有和父进程当前时刻相同的数据
- 父进程继续处理client的请求，子进程负责将快照写入一个临时文件（dump.rdb）
- 子进程写完以后，用临时文件继续替换原来的快照文件，然后子进程退出。

#### 2.3 RDB触发方式

- 根据配置save 900 1等，在满足条件的时候自动触发

- 手动执行bgsave指令触发，在后台进行保存

- 手动执行save指令触发，但是会造成持久化过程中主进程阻塞。在主进程阻塞的期间，服务器不能处理任何客户端的请求。（不常用）

  如果数据量特别大的时候可以考虑，抢资源

- 当shutdown关闭redis的时候，会自动触发

#### 2.4 RDB的注意事项

- 如果发生了系统崩溃，则会丢失最近一次rdb持久化之后的数据，如果项目不能接受这样的数据损失，不建议使用RDB
- 如果数据量巨大，则创建子进程的时间较长、资源较多，导致redis卡顿，要谨慎设置save参数时间间隔；或者如果条件允许，可以每天在闲的时候手动同步。
- 将生成的快照文件，留在原地，每次重启redis的时候，恢复数据状态
- 将生成的快照文件，复制到其他的redis服务器中，相当于数据移植。

### 3. AOF持久化

#### 3.1 AOF配置

编辑redis.conf文件

~~~markdown
appendonly  yes			# 启动AOF机制


# appendfsync alway		# 每次只要收到命令就立马强制写入磁盘，保证完全持久化，但是会产生极大地IO开销

appendfsync everysec	# 每秒钟强制写入磁盘一次，在性能和持久化方面做了一定的平衡（推荐使用）

# appendfsync no		# 完全依赖OS，虽然不对redis的性能产生影响，但是一旦操作系统缓存区满了，会阻塞redis（不推荐使用）

appendfilename "appendonly.aof"  # 设置aof文件名
~~~

#### 3.2 AOF持久化

AppendOnly File：保存的是对redis服务器的写命令来记录数据库状态，保存操作记录

#### 3. 3 AOF运行机制

~~~markdown
redis将每一个写操作（执行成功），写入aof文件，记录的是所有数据的改动行为，redis服务器重启的时候只需要从头到尾执行一次aof文件，即可恢复数据，也可以将aof文件复制到别的服务器，做数据移植。
~~~

#### 3.4 AOF细节

~~~markdown
- AOF文件会不断增长，可能会比RDB大若干倍，在极端情况下，可能会对磁盘空间产生压力
- Redis重启的时候，需要重新执行一个非常大的AOF文件，时间很长
- AOF同步时间间隔小，数据更安全，理论上会最多丢失一秒的数据/但是开销很大
~~~

### 4. RDB和AOF的对比

~~~markdown
- RDB体量小，AOF文件体量更大
- RDB同步时间间隔较大，AOF同步时间间隔小，所以AOF从安全性上来说，更安全
- RDB有更快的恢复速度，可以用来做版本控制。RDB每次进行快照都会重新记录整个数据的所有信息。RDB在恢复数据时更快，可以最大化redis的性能。
- 通过使用RDB和AOF，用户可以再重启或者系统崩溃的时候保留数据，但是随着负载量变大和数据安全的重要性，可以使用redis的复制特性做更好的数据安全保障。
# 作业：了解redis的复制特性
~~~

### 5、 AOF重写

#### 5.1 重写机制

~~~markdown
AOF采用了文件追加的方式持久化数据，所以文件会越来越大
~~~

- 为了减少aof文件的体量，可以手动发送bgrewriteaof命令，则会创建一个子进程，通过移除aof文件中的冗余命令来重写aof文件，生成体量相对较小的aof，然后替换掉之前的、大体量的aof文件
- 也可以设置

~~~markdown
# 比上次重写的时候的体量变大了100%的时候自动触发重写
auto-aof-rewrite-percentage 100
# 在体量超过64mb的时候自动触发重写
auto-aof-rewrite-min-size 64mb
# 在体量超过64mb并且比上次重写的时候的体量变大了100%的时候自动触发重写
~~~

#### 5.2 重写原理

Redis将AOF重写程序放到子进程（后台）执行

- 子进程进行AOF重写，主进程继续处理命令请求
- 子进程带有主进程的数据副本，使用进程而不是线程，是为了保证数据的安全性



子进程进行AOF重写的问题

- 子进程在进行AOF重写的期间，服务器的主进程还要继续处理命令请求，而新的命令可能会对现有的数据进行修改，这会让数据库的数据和记录的AOF数据不一致。

解决方案：

- 为了解决这种数据不一致的问题，Redis增加了一个AOF重写缓存，这个缓存在启动子进程的时候开始启用，Redis服务器主进程在执行完写命令之后，会同时将这个写命令追加到AOF文件和AOF重写缓冲区。

子进程进行AOF重写的时候，主进程需要执行以下操作

- 执行client发来的命令请求
- 将写命令追加到现有的AOF文件中
- 将写命令追加到AOF重写缓存中

### 主从

~~~markdown
开启两个服务器，一个Master、一个Slaver。两个服务器配置没有任何改变。
在从服务器中的配置里面加上一句：
replicaof host port
replicaof 10.31.1.101 6379
~~~

# Python-Redis

~~~markdown
1. pip install redis # 允许使用python去操作redis

2. pip install redis-py-cluster # 允许python操作redis集群

3. pip install django-redis==4.8.0 # 在django中使用redis
~~~



## 一、 连接redis

### 1. python-redis

~~~python
from redis import Redis

# 声明了一个redis的对象
red = Redis(host='10.13.1.101',port=6379)

results = red.lrange('user',0,10)
for result in results:
    print(result.decode('utf-8'))
~~~











































