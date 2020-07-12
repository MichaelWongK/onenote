## 一、什么是Redis

​	"Redis is an open source (BSD licensed),**in-memory data structure store**, used as a database,**cache** and message broker."

Redis是一个开源的，**基于内存的数据结构存储**，可用作于数据库、**缓存**、消息中间件。

- 官方的解释上，我们可以知道：Redis是基于内存，支持多种数据结构。
- 从经验的角度上，我们可以知道：Redis常用作于缓存。

## 1.1为什么要用Redis？

​	Redis是**基于内存**，常用作于**缓存**的一种技术，并且Redis存储的方式是以`key-value`的形式。

类似Java的Map容器特性？为什么还要用Redis?

- Java实现的Map是**本地缓存**，如果有多台实例(机器)的话，每个实例都需要**各自**保存一份缓存，缓存**不具有一致性**
- Redis实现的是**分布式缓存**，如果有多台实例(机器)的话，每个实例都**共享**一份缓存，缓存**具有一致性**。
- Java实现的Map**不是专业**做缓存的，JVM内存太大容易挂掉的。一般用做于容器来存储临时数据，缓存的数据随着JVM销毁而结束。Map所存储的数据结构，缓存过期机制等等是需要程序员自己手写的。
- Redis是**专业**做缓存的，可以用几十个G内存来做缓存。Redis一般用作于缓存，可以将缓存数据保存在硬盘中，Redis重启了后可以将其恢复。原生提供丰富的数据结构、缓存过期机制等等简单好用的功能。

> 为什么要用redis而不用map做缓存?https://segmentfault.com/q/1010000009106416
>
> \

## 1.2为什么要用缓存？

​	如果我们的网站出现了性能问题(访问时间慢)，按经验来说，一般是由于**数据库撑不住了**。因为一般数据库的读写都是要经过**磁盘**的，而磁盘的速度可以说是相当慢的(相对内存来说)

> 科普文：让 CPU 告诉你硬盘和网络到底有多慢https://zhuanlan.zhihu.com/p/24726196

![preview](D:\workspace\git\onenote\study\Redis\file\01.jpg)

![img](D:\workspace\git\onenote\study\Redis\file\02.jpg)

​		Mybatis、Hibernate有一级缓存、二级缓存（本地缓存）目的是为了**不用每次读取的时候，都要查一次数据库**

即——> 提升查询性能

有了缓存之后的访问：

![img](D:\workspace\git\onenote\study\Redis\file\03.jpg)



## 二、Redis的数据结构	

> - Redis 命令参考：http://doc.redisfans.com/
> - try Redis(不用安装Redis即可体验Redis命令)：http://try.redis.io/

​	Redis支持丰富的数据结构，**常用**的有string、list、hash、set、sortset



​	"Redis is written in ANSI C"-->Redis由C语言编写

​	Redis的存储是以`key-value`的形式的。Redis中的key一定是字符串，value可以是string、list、hash、set、sortset这几种常用的。

![img](D:\workspace\git\onenote\study\Redis\file\04.jpg)

注意：Redis并**没有直接使用**这些数据结构来实现`key-value`数据库，而是**基于**这些数据结构创建了一个**对象系统**。

- 简单来说：Redis使用对象来表示数据库中的键和值。每次我们在Redis数据库中新创建一个键值对时，**至少会创建出两个对象**。一个是键对象，一个是值对象。

Redis中的每个对象都由一个redisObject结构来表示：

```
    typedef struct redisObject{

      // 对象的类型
      unsigned type 4:;

      // 对象的编码格式
      unsigned encoding:4;

      // 指向底层实现数据结构的指针
      void * ptr;

      //.....

    }robj;
```

![img](D:\workspace\git\onenote\study\Redis\file\05.jpg)

​																				数据结构对应的类型与编码





































