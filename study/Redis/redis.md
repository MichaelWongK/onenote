## 1、Redis用过哪些数据结构，以及Redis底层怎么实现

### 数据结构

redis支持丰富的数据结构，**常用**的有string、list、hash、set、sortset

Redis的存储是以`key-value`的形式的。Redis中的key一定是字符串，value可以是string、list、hash、set、sortset这几种常用的。

但要值得注意的是：Redis并**没有直接使用**这些数据结构来实现`key-value`数据库，而是**基于**这些数据结构创建了一个**对象系统**。

- 简单来说：Redis使用对象来表示数据库中的键和值。每次我们在Redis数据库中新创建一个键值对时，**至少会创建出两个对象**。一个是键对象，一个是值对象。
- Redis对`key-value`封装成对象，key是一个对象，value也是一个对象。每个对象都有type(类型)、encoding(编码)、ptr(指向底层数据结构的指针)来表示。

### 底层数据结构实现：

#### SDS简单动态字符串

> 简单动态字符串(Simple dynamic string,SDS)

Redis使用sdshdr结构来表示一个SDS值：

```
struct sdshdr{

    // 字节数组，用于保存字符串
    char buf[];

    // 记录buf数组中已使用的字节数量，也是字符串的长度
    int len;

    // 记录buf数组未使用的字节数量
    int free;
}
```

![img](http://www.micheal.wang:10020/mongo/read/5f40e6ac9c5fd27c4ff5d8b2)

#### Redis链表的特性

- 无环双向链表
- 获取表头指针，表尾指针，链表节点长度的时间复杂度均为O(1)
- 链表使用`void *`指针来保存节点值，可以保存各种不同类型的值

#### 哈希表(字典)

在Redis中，`key-value`的数据结构底层就是哈希表来实现的。对于哈希表来说，我们也并不陌生。在Java中，哈希表实际上就是数组+链表的形式来构建的。

在Redis里边，哈希表使用dictht结构来定义：

```
    typedef struct dictht{

        //哈希表数组
        dictEntry **table;  

        //哈希表大小
        unsigned long size;    

        //哈希表大小掩码，用于计算索引值
        //总是等于size-1
        unsigned long sizemark;     

        //哈希表已有节点数量
        unsigned long used;

    }dictht
```

继续看看哈希表的节点是怎么实现的吧：

```
    typedef struct dictEntry {

        //键
        void *key;

        //值
        union {
            void *value;
            uint64_tu64;
            int64_ts64;
        }v;    

        //指向下个哈希节点，组成链表
        struct dictEntry *next;

    }dictEntry;
```

从结构上看，我们可以发现：Redis实现的哈希表和Java中实现的是**类似**的。只不过Redis多了几个属性来记录常用的值：sizemark(掩码)、used(已有的节点数量)、size(大小)。

#### **Redis中有两个哈希表**：

- ht[0]：用于存放**真实**的`key-vlaue`数据
- ht[1]：用于**扩容(rehash)**

Redis中哈希算法和哈希冲突跟Java实现的差不多，它俩**差异**就是：

- Redis哈希冲突时：是将新节点添加在链表的**表头**。
- JDK1.8后，Java在哈希冲突时：是将新的节点添加到链表的**表尾**。



2、Redis缓存穿透，缓存雪崩

3、如何使用Redis来实现分布式锁

4、Redis的并发竞争问题如何解决

5、Redis持久化的几种方式，优缺点是什么，怎么实现的

6、Redis的缓存失效策略

7、Redis集群，高可用，原理

8、Redis缓存分片

9、Redis的数据淘汰策略

## Redis对象一些细节

- (1：服务器在执行某些命令的时候，会**先检查给定的键的类型**能否执行指定的命令。

- - 比如我们的数据结构是sortset，但你使用了list的命令。这是不对的，服务器会检查一下我们的数据结构是什么才会进一步执行命令

- (2：Redis的对象系统带有**引用计数**实现的**内存回收机制**。

- - 对象不再被使用的时候，对象所占用的内存会释放掉

- (3：Redis会共享值为0到9999的字符串对象

- (4：对象**会记录自己的最后一次被访问时间**，这个时间可以用于计算对象的空转时间。

使用的时候挑选哪些数据结构作为存储，可以简单看看：

- string-->简单的`key-value`
- list-->有序列表(底层是双向链表)-->可做简单队列
- set-->无序列表(去重)-->提供一系列的交集、并集、差集的命令
- hash-->哈希表-->存储结构化数据
- sortset-->有序集合映射(member-score)-->排行榜