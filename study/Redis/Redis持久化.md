# Redis持久化

Redis是基于内存的，如果不想办法将数据保存在硬盘上，一旦Redis重启(退出/故障)，内存的数据将会全部丢失。

- 我们肯定不想Redis里头的数据由于某些故障全部丢失(导致所有请求都走MySQL)，即便发生了故障也希望可以将Redis原有的数据恢复过来，这就是持久化的作用。

Redis提供了两种不同的持久化方法来讲数据存储到硬盘里边：

- RDB(基于快照)，将某一时刻的所有数据保存到一个RDB文件中。
- AOF(append-only-file)，当Redis服务器执行**写命令**的时候，将执行的**写命令**保存到AOF文件中。

## RDB(快照持久化)

RDB持久化可以**手动**执行，也可以根据服务器配置**定期**执行。RDB持久化所生成的RDB文件是一个经过**压缩**的二进制文件，Redis可以通过这个文件**还原**数据库的数据。

![RDB文件还原数据](http://www.micheal.wang:10020/mongo/read/5f437cfc9c5fd27c4ff5d8ba)RDB文件还原数据

有两个命令可以生成RDB文件：

- `SAVE`会**阻塞**Redis服务器进程，服务器不能接收任何请求，直到RDB文件创建完毕为止。
- `BGSAVE`创建出一个**子进程**，由子进程来负责创建RDB文件，服务器进程可以继续接收请求。

Redis服务器在启动的时候，如果发现有RDB文件，就会**自动**载入RDB文件(不需要人工干预)

- 服务器在载入RDB文件期间，会处于阻塞状态，直到载入工作完成。

除了手动调用`SAVE`或者`BGSAVE`命令生成RDB文件之外，我们可以使用配置的方式来**定期**执行：

在默认的配置下，如果以下的条件被触发，就会执行`BGSAVE`命令

```
    save 900 1              #在900秒(15分钟)之后，至少有1个key发生变化，
    save 300 10            #在300秒(5分钟)之后，至少有10个key发生变化
    save 60 10000        #在60秒(1分钟)之后，至少有10000个key发生变化
```

原理大概就是这样子的(结合上面的配置来看)：

```
struct redisServer{
    // 修改计数器
    long long dirty;

    // 上一次执行保存的时间
    time_t lastsave;

    // 参数的配置
    struct saveparam *saveparams;
};
```

遍历参数数组，判断修改次数和时间是否符合，如果符合则调用`besave()`来生成RDB文件

![Redis服务器的状态](http://www.micheal.wang:10020/mongo/read/5f437d179c5fd27c4ff5d8bc)Redis服务器的状态

总结：通过手动调用`SAVE`或者`BGSAVE`命令或者配置条件触发，将数据库**某一时刻**的数据快照，生成RDB文件实现持久化。



## AOF(文件追加)

上面已经介绍了RDB持久化是通过将某一时刻数据库的数据“快照”来实现的，下面我们来看看AOF是怎么实现的。

- AOF是通过保存Redis服务器所执行的**写命令**来记录数据库的数据的。

![AOF原理图](http://www.micheal.wang:10020/mongo/read/5f4382ad9c5fd27c4ff5d8be)AOF原理图

比如说我们对空白的数据库执行以下写命令：

```
redis> SET meg "hello"
OK

redis> SADD fruits "apple" "banana" "cherry"
(integer) 3

redis> RPUSH numbers 128 256 512
(integer) 3 
```

Redis会产生以下内容的AOF文件：

![AOF文件](http://www.micheal.wang:10020/mongo/read/5f4382d89c5fd27c4ff5d8c0)AOF文件

这些都是以Redis的命令**请求协议格式**保存的。Redis协议规范(RESP)参考资料：

- https://www.cnblogs.com/tommy-huang/p/6051577.html

AOF持久化功能的实现可以分为3个步骤：

- 命令追加：命令写入aof_buf缓冲区
- 文件写入：调用flushAppendOnlyFile函数，考虑是否要将aof_buf缓冲区写入AOF文件中
- 文件同步：考虑是否将内存缓冲区的数据真正写入到硬盘

![AOF持久化步骤](http://www.micheal.wang:10020/mongo/read/5f4383009c5fd27c4ff5d8c2)AOF持久化步骤

flushAppendOnlyFile函数的行为由服务器配置的**appendfsyn选项**来决定的：

```
    appendfsync always     # 每次有数据修改发生时都会写入AOF文件。
    appendfsync everysec   # 每秒钟同步一次，该策略为AOF的默认策略。
    appendfsync no         # 从不同步。高效但是数据不会被持久化。
```

从字面上应该就更好理解了，这里我就不细说了…

下面来看一下AOF是如何载入与数据还原的：

- 创建一个**伪客户端**(本地)来执行AOF的命令，直到AOF命令被全部执行完毕。

![redis伪客户端载入AOF文件](http://www.micheal.wang:10020/mongo/read/5f43831e9c5fd27c4ff5d8c4)

### AOF重写

从前面的示例看出，我们写了三条命令，AOF文件就保存了三条命令。如果我们的命令是这样子的：

```
redis > RPUSH list "Java" "3y"
(integer)2

redis > RPUSH list "Java3y"
integer(3)

redis > RPUSH list "yyy"
integer(4)
```

同样地，AOF也会保存3条命令。我们会发现一个问题：上面的命令是可以**合并**起来成为1条命令的，并不需要3条。这样就可以**让AOF文件的体积变得更小**。

AOF重写由Redis自行触发(参数配置)，也可以用`BGREWRITEAOF`命令**手动触发**重写操作。

- 要值得说明的是：**AOF重写不需要对现有的AOF文件进行任何的读取、分析。AOF重写是通过读取服务器当前数据库的数据来实现的**！

比如说现在有一个Redis数据库的数据如下：

![Redis数据库的数据](http://www.micheal.wang:10020/mongo/read/5f43854b9c5fd27c4ff5d8c6)Redis数据库的数据

新的AOF文件的命令如下，**没有一条是多余的**！

![AOF重写后的命令](http://www.micheal.wang:10020/mongo/read/5f4385749c5fd27c4ff5d8c8)AOF重写后的命令

### AOF后台重写

Redis将AOF重写程序放到**子进程**里执行(`BGREWRITEAOF`命令)，像`BGSAVE`命令一样fork出一个子进程来完成重写AOF的操作，从而不会影响到主进程。

AOF后台重写是不会阻塞主进程接收请求的，新的写命令请求可能会导致**当前数据库和重写后的AOF文件的数据不一致**！

为了解决数据不一致的问题，Redis服务器设置了一个**AOF重写缓冲区**，这个缓存区会在服务器**创建出子进程之后使用**。

![AOF后台重写过程](http://www.micheal.wang:10020/mongo/read/5f4385a79c5fd27c4ff5d8ca)															AOF后台重写过程

## RDB和AOF对过期键的策略

RDB持久化对过期键的策略：

- 执行`SAVE`或者`BGSAVE`命令创建出的RDB文件，程序会对数据库中的过期键检查，**已过期的键不会保存在RDB文件中**。
- 载入RDB文件时，程序同样会对RDB文件中的键进行检查，**过期的键会被忽略**。

RDB持久化对过期键的策略：

- 如果数据库的键已过期，但还没被惰性/定期删除，AOF文件不会因为这个过期键产生任何影响(也就说会保留)，当过期的键被删除了以后，会追加一条DEL命令来显示记录该键被删除了
- 重写AOF文件时，程序会对RDB文件中的键进行检查，**过期的键会被忽略**。

复制模式：

- **主服务器来控制**从服务器统一删除过期键(保证主从服务器数据的一致性)

## RDB和AOF用哪个？

RDB和AOF并不互斥，它俩可以**同时使用**。

- RDB的优点：载入时**恢复数据快**、文件体积小。
- RDB的缺点：会一定程度上**丢失数据**(因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。)
- AOF的优点：丢失数据少(默认配置只丢失一秒的数据)。
- AOF的缺点：恢复数据相对较慢，文件体积大

如果Redis服务器**同时开启**了RDB和AOF持久化，服务器会**优先使用AOF文件**来还原数据(因为AOF更新频率比RDB更新频率要高，还原的数据更完善)

可能涉及到RDB和AOF的配置：

```
redis持久化，两种方式
1、rdb快照方式
2、aof日志方式

----------rdb快照------------
save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /var/rdb/

-----------Aof的配置-----------
appendonly no # 是否打开 aof日志功能

appendfsync always #每一个命令都立即同步到aof，安全速度慢
appendfsync everysec
appendfsync no 写入工作交给操作系统，由操作系统判断缓冲区大小，统一写入到aof  同步频率低，速度快


no-appendfsync-on-rewrite yes 正在导出rdb快照的时候不要写aof
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb 


./bin/redis-benchmark -n 20000
```

官网文档：

- https://redis.io/topics/persistence#rdb-advantages