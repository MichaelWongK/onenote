Redis 即 REmote Dictionary Server (远程字典服务)；

而Redis的协议规范是 Redis Serialization Protocol (Redis序列化协议)

该协议是用于与Redis服务器通信的，用的较多的是Redis-cli通过pipe与Redis服务器联系；

  协议如下：

​     客户端以规定格式的形式发送命令给服务器；

​     服务器在执行最后一条命令后，返回结果。

## 客户端发送命令的格式(类型)：5种类型

  间隔符号，在Linux下是\r\n，在Windows下是\n

### 1. 简单字符串 Simple Strings, 以 "+"加号 开头

   格式：+ 字符串 \r\n

​        字符串不能包含 CR或者 LF(不允许换行)

   eg: "+OK\r\n"

   注意：为了发送二进制安全的字符串，一般推荐使用后面的 Bulk Strings类型

### 2. 错误 Errors, 以"-"减号 开头

　　格式：- 错误前缀 错误信息 \r\n

​        错误信息不能包含 CR或者 LF(不允许换行)，Errors与Simple Strings很相似，不同的是Erros会被当作异常来看待

   eg: "-Error unknow command 'foobar'\r\n"

### 3. 整数型 Integer， 以 ":" 冒号开头

　　格式：: 数字 \r\n

   eg: ":1000\r\n"

### 4. 大字符串类型 Bulk Strings, 以 "$"美元符号开头，长度限制512M

　　格式：$ 字符串的长度 \r\n 字符串 \r\n

​        字符串不能包含 CR或者 LF(不允许换行);

   eg: "$**6**\r\n**foobar**\r\n"   其中字符串为 foobar，而6就是foobar的字符长度

​      "$0\r\n\r\n"    空字符串

​      "$-1\r\n"      null

### 5. 数组类型 Arrays，以 "*"星号开头

　　格式：* 数组元素个数 \r\n 其他所有类型 (结尾不需要\r\n)

​       注意：只有元素个数后面的\r\n是属于该数组的，结尾的\r\n一般是元素的

   eg: "*0\r\n"    空数组

​      "*2\r\n$2\r\nfoo\r\n$3\r\nbar\r\n"    数组包含2个元素，分别是字符串foo和bar

　　　　"*3\r\n:1\r\n:2\r\n:3\r\n"    数组包含3个整数：1、2、3

​      "*5\r\n:1\r\n:2\r\n:3\r\n:4\r\n$6\r\nfoobar\r\n"  包含混合类型的数组

​      "*-1\r\n"     Null数组

​      "*2\r\n*3\r\n:1\r\n:2\r\n:3\r\n*2\r\n+Foo\r\n-Bar\r\n"  数组嵌套，外层数组包含2个数组，整理后如下：

​         "*2\r\n

　　　　　　*3\r\n:1\r\n:2\r\n:3\r\n

　　　　　　*2\r\n+Foo\r\n-Bar\r\n"