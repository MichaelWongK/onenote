## 按功能分类

标记了**※**的表示常用参数

------

### 内存参数

### -Xmx[value] ※

设置堆内存最大值

> -Xmx1g
> 或者
> -Xmx1024m

#### -Xms[value] ※

设置堆内存最小值。一般与`-Xmx`设置一样大

> -Xmx1g

#### -Xmn[value] ※

设置新生代大小

> -Xmn256m

#### -Xss[value] ※

设置栈空间大小

> -Xss128k

#### -XX:SurvivorRatio=[value] ※

新生代Eden和Survivor划分比例

> -XX:SurvivorRatio=8

#### -XX:PermSize=[value] ※

设置永久代初始大小。JDK8中已移除

> -XX:PermSize=128m

#### -XX:MaxPermSize=[value] ※

设置永久代最大值。JDK8中已移除

> -XX:MaxPermSize=128m

#### -XX:MetaspaceSize=[value] ※

设置meta区大小。JDK8增加

> -XX:MetaspaceSize=128m

#### -XX:MaxMetaspaceSize=[value] ※

设置永久代最大值。JDK8中已移除

> -XX:MaxMetaspaceSize=128m

#### -XX:ReservedCodeCacheSize

用于设置Code Cache大小，JIT编译的代码都放在Code Cache中，若Code Cache空间不足则JIT无法继续编译，并且会去优化，比如编译执行改为解释执行，由此，性能会降低

> -XX:ReservedCodeCacheSize=128m

------

### 行为相关参数

Behavioral Options，即影响VM基本行为的参数

#### -XX:-DisableExplicitGC ※

------

### GC参数

#### -XX:-UseSerialGC

使用串行垃圾回收器回收新生代

#### -XX:+UseParNewGC ※

使用并行垃圾回收器回收新生代

#### -XX:ParallerGCThreads

当使用`-XX:+UseParNewGC`时，该参数设定GC的线程数，默认与CPU核数相同

#### -XX:-UseParallelGC

使用Parallel Scavenge垃圾回收器

> **-XX:UseParallelGC = “Parallel Scavenge” + “Serial Old”**

#### -XX:-UseParallelOldGC

用并行垃圾回收器进行full gc

> **-XX:UseParallelOldGC = “Parallel Scavenge” + “Parallel Old”**

#### -XX:-UseConcMarkSweepGC ※

使用CMS做为垃圾回收器

注：当前常见的垃圾回收器组合是下面这种：

> **-XX:+UseConcMarkSweepGC -XX:+UseParNewGC**

#### -Xloggc:[path] ※

设置gc日志位置

> -Xloggc:/opt/logs/mobile/admin.gc.log

#### -XX:+PrintGC ※

打印GC详情

输出形式:

> [GC 118250K->113543K(130112K), 0.0094143 secs]
> [Full GC 121376K->10414K(130112K), 0.0650971 secs]

#### -XX:+PrintGCDetails ※

打印GC更详细的信息

输出形式:

> [GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]
> [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]