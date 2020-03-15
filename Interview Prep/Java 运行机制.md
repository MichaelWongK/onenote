## 										Java 运行机制

在本篇文章中，将重点研究java源代码的执行原理，即从程序员编写JAVA源代码，到最终形成产品，在整个过程中，都经历了什么？每一步又是怎么执行的？执行原理又是什么？

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpWWvhoePcouuFX8LeKyjpImHG9lQSOVlLTBcfjpdaURqu4Rwia2sibAzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**一、编写java源程序**

java源文件:指存储java源码的文件。

先来看看如下代码：

```
//MyTest被public修饰，故存储该java源码的文件名为MyTestpublic class MyTest {  public static void main(String[] args){System.out.println("Test Java execute process.");}}//由于MyTest被public修饰了，故Class A不能用public修饰class A{}//由于MyTest被public修饰了，故Class B不能用public修饰class B{}
```

1、java源文件名就是该源文件中public类的名称

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpD1xqJ7iaBn1iajIm9ib9os4W1AP4GuDCqVKRic8K946mgHw3RXB286MpQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2、一个java源文件可以包含多个类，但只允许一个类为public

**二、编译java源代码**

当java源程序编码结束后，就需要编译器编译。

安装好jdk后，我们打开jdk目录，有两个.exe文件，即javac.exe(编译源代码，xxx.java文件) 和 java.exe(执行字节码，xxx.class文件).

如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpamriaYwNSzA2qt7wJ6GZLb5RpjVbv9oTq9FUVucd3bTjYMoDdvpIstQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1、切换到MyTest.java文件夹

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpRia0EsM8LqDic5WmnebibSE1qoynFU5G7KicVhnRD7XuNnGcDPyMy9Jl9Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2、javac.exe编译MyTest.java

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

编译后，发现e:\Blogs 目录多了以class为后缀的文件：A.class,B.class和MyTest.class

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Tip:当javac.exe编译java源代码时，java源代码有几个类，就会编译成一个对应的字节码文件(.class文件)

其中，字节码文件的文件名就是每个类的类名。需要注意的是，类即使不在源文件中定义，但被源文件引用，编译后，也会编程相应的字节码文件。

如类A引用类C，但类C不定义在类A的源文件中，编译后，类C也被编译成对应的字节码文件C.class

Tips：关注微信公众号：Java后端，每日获取技术博文推送。

**三、执行java源文件**

执行java源文件，用java.exe执行即可

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

到现在，java源程序基本执行结果，并正确打印我们期望的结果，那么，如上的步骤，我们可以总结如下：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如上总结，已经抽象化了在JVM中的执行。接下来，我们将分析字节码文件（.class文件）如何在虚拟机中一步一执行的。

**四、JVM如何执行字节码文件**

1、装载字节码文件

当 .java 源码被 javac.exe 编译器编译成 .class 字节码文件后，接下来的工作就交给JVM处理。

JVM首先通过类加载器(ClassLoader)，将class文件和相关Java API加载装入JVM，以供JVM后续处理。

在该阶段中，涉及到如下一些基本概念和知识。

1）JDK,JRE和JVM关系

- JDK（Java Development Kit），Java开发工具包，主要用于开发，在JDK7前，JDK包括JRE
- JRE（Java Runtime Environment），Java程序运行的核心环境，包括JVM和一些核心库
- JVM（Java Virtual Machine），VM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的，是JRE核心模块。

2）JVM

JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

Java虚拟机的主要任务是装载class文件，并执行其中的字节码，不同的Java虚拟机中，执行引擎可能有不同的实现。

大致有如下几种引擎：

-    一次性解释字节码引擎
- ​    即时编译引擎
-    自适应优化器

关于虚拟机的实现方式，采用软件方式、硬件方式和软件硬件结合方式，这个要根据具体厂商而定。

3）什么是ClassLoader

虚拟机的主要任务是装载class文件并执行其中的字节码，而class文件是由虚拟机的类加载器(ClassLoader)完成的，在一个Java虚拟机中有可能存在多个类加载器。

任何java运用程序，可能会使用两种类加载器，即启动类加载器(bootstrap)和用户自定义类加载器。     

启动类加载器是Java虚拟机唯一实现的一部分，它又可分为原始类装载器，系统类装载器或默认类装载器。它的主要作用是从操作系统的磁盘装载相应的类，如Java API类等。

用户自定义装载类，即按照用户自定义的方式来装载类。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2、将字节码文件存储在JVM内存区

当JAVA虚拟机运行一个程序时，它需要内存来存储许多东西。

比如如字节码，程序创建的对象，传递给方法的参数，返回值，局部变量以及运算的中间结果等，这些相关信息被组织到“运行时数据区”。

根据厂商的不同，在Java虚拟机中，运行时数据区也有所不同。有些运行时数据区由线程共享，有些只能由某个特定线程共享。

运行时数据区大致可分几个区：方法区，堆区，栈区，PC寄存器区和本地方法栈区。

在该阶段中，涉及到如下基本概念和知识。

1）方法区

方法区用来存储解析被加载的class文件的相关信息。

当虚拟装载一个class文件后，它会从这个class文件包含的二进制数据中解析类型信息，然后将该相关信息存储到方法区中。

2）堆

堆是用来存储相关引用类型的，如new对象。当程序运行时，虚拟机会把所有该程序在运行时创建的对象都放到堆中。

3）PC寄存器

PC寄存器主要用来存储线程。当新创建一个线程时，该线程都将得到一个自己的PC寄存器(程序计数器)以及一个java栈。

Java虚拟机没有寄存器，其指令集使用Java栈来存储中间数据。

4）栈区

栈区主要用来存储值类型的，如基本数据类型。需要注意的是，String为引用类型，是存在堆中的。

Java栈是由许多栈帧组成的，一个栈帧包含一个Java方法调用的状态，当线程调用一个方法时，虚拟机压入一个新的栈帧到该线程的Java栈中，当该方法返回时，这个栈帧从Java栈中弹出。

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpGaaK4xRerFd6fRLcszeCicrcUrvsh7clDFezRBia3x3kXbGpllgX8HZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、执行引擎与运行时数据区交互

运行时数据区为执行引擎提供了执行环境和相关数据，执行引擎通过与运行时数据区交互，从而获取执行时需要的相关信息，存储执行的中间结果等

 ![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpujh1Amts9zrX1kp3iaNQx0C1ZAJ4BgUTQY8cr3mITh5dXF89UcePkxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、执行引擎与本地方法接口

当要执行本地方法时，执行引擎将调用本地方法接口来获取相关OS本地方法。

需要注意的是，本地方法与操作系统强耦合的。

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpekQwqGa6bqxWkubcmeu0fE0NtW2dnfJBXMqzUIia5tGsqia9RGFM2I6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5、JVM在具体操作系统上执行

JVM通过调用本地接口来获取本地方法，从而实现在具体的平台上执行。比如在Linux系统上执行，在Window系统上执行和在Unix系统上执行。

![img](https://mmbiz.qpic.cn/mmbiz_png/vnOqylzBGCTgZM7VBsyv3O2na0ibggMGpgcm487W56CcLTClkle6QxibDSy91m2lAZnm1ia5XFIRzd2bqr5mHvyjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**五、参考文献**

1、深入Java虚拟机（原书第2版）(美)Bill Venners 著  

2、Core Java Volume I - Fundamententals(10th Edition) 

3、Core Java Volume I - Advanced Features(10th Edition) 