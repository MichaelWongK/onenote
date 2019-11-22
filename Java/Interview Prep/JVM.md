# 								JVM 面试题

##### 1、你知道哪些或者你们线上使⽤什么GC策略？它有什么优势，适⽤于什么场景？**

参考 触发JVM进行Full GC的情况及应对策略。

> https://blog.csdn.net/chenleixing/article/details/46706039/

##### **2、Java类加载器包括⼏种？它们之间的⽗⼦关系是怎么样的？双亲委派机制是什么意思？有什么好处？**

启动Bootstrap类加载、扩展Extension类加载、系统System类加载。

父子关系如下：

- 启动类加载器 ，由C++ 实现，没有父类；
- 扩展类加载器，由Java语言实现，父类加载器为null；
- 系统类加载器，由Java语言实现，父类加载器为扩展类加载器；
- 自定义类加载器，父类加载器肯定为AppClassLoader。

双亲委派机制：类加载器收到类加载请求，自己不加载，向上委托给父类加载，父类加载不了，再自己加载。

优势避免Java核心API篡改。详细查看：深入理解Java类加载器(ClassLoader)

> https://blog.csdn.net/javazejian/article/details/73413292/

##### **3、如何⾃定义⼀个类加载器？你使⽤过哪些或者你在什么场景下需要⼀个⾃定义的类加载器吗？**

自定义类加载的意义：

1. 加载特定路径的class文件
2. 加载一个加密的网络class文件
3. 热部署加载class文件

##### **4、堆内存设置的参数是什么？**

-Xmx 设置堆的最大空间大小

-Xms 设置堆的最小空间大小

##### **5、Perm Space中保存什么数据？会引起OutOfMemory吗？**

加载class文件。

会引起，出现异常可以设置 -XX:PermSize 的大小。JDK 1.8后，字符串常量不存放在永久带，而是在堆内存中，JDK8以后没有永久代概念，而是用元空间替代，元空间不存在虚拟机中，二是使用本地内存。

详细查看Java8内存模型—永久代(PermGen)和元空间(Metaspace)

> https://www.cnblogs.com/paddix/p/5309550.html/

##### **6、做GC时，⼀个对象在内存各个Space中被移动的顺序是什么？**

标记清除法，复制算法，标记整理、分代算法。

新生代一般采用复制算法 GC，老年代使用标记整理算法。

垃圾收集器：串行新生代收集器、串行老生代收集器、并行新生代收集器、并行老年代收集器。

CMS（Current Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，它是一种并发收集器，采用的是Mark-Sweep算法。

详见 Java GC机制。

> http://www.cnblogs.com/dolphin0520/p/3783345.htmll/

##### **7、你有没有遇到过OutOfMemory问题？你是怎么来处理这个问题的？处理 过程中有哪些收获？**

permgen space、heap space 错误。

**常见的原因**

- 内存加载的数据量太大：一次性从数据库取太多数据；
- 集合类中有对对象的引用，使用后未清空，GC不能进行回收；
- 代码中存在循环产生过多的重复对象；
- 启动参数堆内存值小。

详见 Java 内存溢出（java.lang.OutOfMemoryError）的常见情况和处理方式总结。

> http://outofmemory.cn/c/java-outOfMemoryError/

###### **8、JDK 1.8之后Perm Space有哪些变动? MetaSpace⼤⼩默认是⽆限的么? 还是你们会通过什么⽅式来指定⼤⼩?**

JDK 1.8后用元空间替代了 Perm Space；字符串常量存放到堆内存中。

MetaSpace大小默认没有限制，一般根据系统内存的大小。JVM会动态改变此值。

-XX:MetaspaceSize：分配给类元数据空间（以字节计）的初始大小（Oracle逻辑存储上的初始高水位，the initial high-water-mark）。此值为估计值，MetaspaceSize的值设置的过大会延长垃圾回收时间。垃圾回收过后，引起下一次垃圾回收的类元数据空间的大小可能会变大。

-XX:MaxMetaspaceSize：分配给类元数据空间的最大值，超过此值就会触发Full GC，此值默认没有限制，但应取决于系统内存的大小。JVM会动态地改变此值。

##### **9、jstack 是⼲什么的? jstat 呢？如果线上程序周期性地出现卡顿，你怀疑可 能是 GC 导致的，你会怎么来排查这个问题？线程⽇志⼀般你会看其中的什么 部分？**

jstack 用来查询 Java 进程的堆栈信息。

jvisualvm 监控内存泄露，跟踪垃圾回收、执行时内存、cpu分析、线程分析。

详见Java jvisualvm简要说明，可参考 线上FullGC频繁的排查。

**Java jvisualvm简要说明**

> https://blog.csdn.net/a19881029/article/details/8432368/

**线上FullGC频繁的排查**

> https://blog.csdn.net/wilsonpeng3/article/details/70064336/

##### **10、StackOverflow异常有没有遇到过？⼀般你猜测会在什么情况下被触发？如何指定⼀个线程的堆栈⼤⼩？⼀般你们写多少？**

栈内存溢出，一般由栈内存的局部变量过爆了，导致内存溢出。出现在递归方法，参数个数过多，递归过深，递归没有出口。