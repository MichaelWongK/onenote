1.Java Thread.start() 如何调用 run() 方法的

![](C:\Users\Administrator\Desktop\面试\Java多线程\threadstart.jpg)



Java程序的调度都是依赖于JVM中的栈, 而栈的执行则是由线程进行的, 而这个线程其实就是操作系统的线程. 

在Java中我们启动一个线程的话, 其实就是启动了一个操作系统的线程, 例如使用pthread开启一个线程. Java中或者说JVM并没有线程, 只有持有线程的线程对象.

这么说可能还是不太清晰, 从源码看一下.

当Java中的Thread类被加载进JVM之后, 会调用一个registerNatives()的一个本地方法.![img](https://pic4.zhimg.com/80/v2-6c78cb554b6861a529ddc36fec3c81b8_720w.jpg)

这个方法是在jvm中实现的, 在Thread.c中实现了该方法![img](https://pic3.zhimg.com/80/v2-94d7073c033a2e27165849a0c7424eac_720w.jpg)

在methods这个数组中做了一个映射, 将java中线程的native方法与C的函数指针进行了关联.

调用Java中Thread的start()方法时, 其实就是调用的JVM_StartThread这个函数指针![img](https://pic4.zhimg.com/80/v2-2a137d28c9eb6dc55eec5fc8b4e8f7e3_720w.jpg)

看到没, 在jvm中,是直接创建了一个c++的JavaThread类型的native_thread对象, 紧接着就开始调用jvm中的Thread的start()函数. 

在创建JavaThread的时候, 其内部又创建了一个OSThrea![img](https://pic1.zhimg.com/80/v2-a079bd7b138f7653b3179de9dda034cb_720w.jpg)![img](https://pic1.zhimg.com/80/v2-ddb866bd314c9dc5ede350505368a762_720w.jpg)

在OSThread中就利用pthread_create()创建出了一个线程. 在调用pthread_create()函数的时候, 下面俩个参数需要看一下

- java_start, 线程启动后的回调函数
- thread, 回调函数的参数



 在java_start()函数先进行一些状态检查设置, 最后调用thread(JavaThread类型, pthread_create最后一个参数)的run()方法, 进行一些与Java线程相关的状态设置, 例如方法栈之类的(此时的处理就从系统线程那转移到了Java线程).

![img](https://pic2.zhimg.com/50/v2-b4228ae1506de4ed07d9758ae95ca1d3_hd.jpg)![(https://

在thread_main_inner()方法中就开始调用刚开始传递的entry_point()函数了![img](https://pic2.zhimg.com/80/v2-a275a112791c95a2492dd5be5d8b4a28_720w.jpg)![img](https://pic2.zhimg.com/80/v2-6cdf1949edd75531426f433e43419076_720w.jpg)

此时就开始调用Java中的Thread的run()方法了.

------

回过头来刚开始 在JVM_StartThread()中最后一行调用了

![img](https://pic4.zhimg.com/50/v2-fd5bd59168fc293a16c79f78a242e834_hd.jpg)![img](https://pic4.zhimg.com/80/v2-fd5bd59168fc293a16c79f78a242e834_720w.jpg)

 看一下这个方法的实现![img](https://pic4.zhimg.com/80/v2-95e6d5e50547c7fbeffe9e6de35c0950_720w.jpg)

![img](https://pic4.zhimg.com/50/v2-4c64512d749d333aae90a95f940cc17c_hd.jpg)![(https://![img](https://pic4.zhimg.com/80/v2-bc3ce8125148f0b29961c6a2997535b3_720w.jpg)

这里主要是进行一些线程状态的设置.