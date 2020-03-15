## Spring 特征

![image-20200312224052952](..\file\Spring 特征.png)

## Spring 核心组件

![image-20200312224358924](..\file\Spring 核心组件.png)

## 开发中主要使用 Spring 的什么技术 ?

> ①. IOC 容器管理各层的组件
> ②. 使用 AOP 配置声明式事务
> ③. 整合其他框架.

![image-20200312225130783](..\file\常用模块.png)

![image-20200312225019761](D:\workspace\git\onenote\Java\Interview Prep\file\Spring常用注解.png)

## 简述 AOP 和 IOC 概念 AOP:

> Aspect Oriented Program, 面向(方面)切面的编程;Filter(过滤器) 也是一种 AOP. 
>
> **AOP** 是一种新的方法论, 是对传统 OOP(Object-Oriented Programming, 面向对象编程) 的补充. AOP 的主要编程对象是切面(aspect), 而切面模块化横切关注点.可以举例通过事务说明.
>
> **IOC**: Invert Of Control, 控制反转. 也成为 DI(依赖注入)其思想是反转 资源获取的方向. 传统的资源查找方式要求组件向容器发起请求查找资源.作为 回应, 容器适时的返回资源. 而应用了IOC 之后, 则是容器主动地将资源推送 给它所管理的组件,组件所要做的仅是选择一种合适的方式来接受资源. 这种行 为也被称为查找的被动形式
>
> **IOC优点**：IOC(Inversion of Control)控制反转，将控制权(创建对象和对象之间的依赖关系的权利)交给spring容器。程序员自己不用再new 对象了，这些都交给容器来做。
>
> 接口驱动设计(Interface Driven Design)的好处，可以灵活提供不同的子类实现(其实就是解耦),提高程序的灵活性、可扩展性和可维护性。

## 在 Spring 中如何配置 Bean ?

> Bean 的配置方式: 通过全类名（反射）、通过工厂方法（静态工厂方法 & 实 例工厂方法）、FactoryBean

## IOC 容器对 Bean 的生命周期:

> ①. 通过构造器或工厂方法创建 Bean 实例
> ②. 为 Bean 的属性设置值和对其他 Bean 的引用
> ③ . 将 Bean 实 例 传 递 给 Bean 后 置 处 理 器 的 postProcessBeforeInitialization 方法 
>
> ④. 调用 Bean 的初始化方法(init-method)  



## Spring bean的作用域和生命周期

> Bean 是 Spring 应用的骨架。它们由 Spring IoC 容器管理。换句话说，Bean 是一个由 Spring IoC 容器初始化、装配和管理的对象。
> 下面是 Spring Bean 的5种作用域：

**singleton：单例模式（多线程下不安全）**

> 1.singleton：单例模式， Spring IoC 容器中只会存在一个共享的 Bean 实例，无论有多少个
> Bean 引用它，始终指向同一对象。 该模式在多线程下是不安全的。 Singleton 作用域是
> Spring 中的缺省作用域，也可以显示的将 Bean 定义为 singleton 模式，配置为：

```
<bean id="userDao" class="com.ioc.UserDaoImpl" scope="singleton"/>
```

**prototype:原型模式每次使用时创建**

> prototype:原型模式，每次通过 Spring 容器获取 prototype 定义的 bean 时，容器都将创建
> 一个新的 Bean 实例，每个 Bean 实例都有自己的属性和状态，而 singleton 全局只有一个对
> 象。根据经验， 对有状态的bean使用prototype作用域，而对无状态的bean使用singleton
> 作用域。

**Request：一次 request 一个实例**

> request：在一次 Http 请求中，容器会返回该 Bean 的同一实例。而对不同的 Http 请求则会
> 产生新的 Bean，而且该 bean 仅在当前 Http Request 内有效,当前 Http 请求结束，该 bean
> 实例也将会被销毁。

```
<bean id="loginAction" class="com.cnblogs.Login" scope="request"/>
```

**session**

>session：在一次 Http Session 中，容器会返回该 Bean 的同一实例。而对不同的 Session 请
求则会创建新的实例，该 bean 实例仅在当前 Session 内有效。 同 Http 请求相同，每一次
session 请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的 session 请求
内有效，请求结束，则实例将被销毁。

```
<bean id="userPreference" class="com.ioc.UserPreference" scope="session"/>
```

**global Session**

> global Session：在一个全局的 Http Session 中，容器会返回该 Bean 的同一个实例，仅在
> 使用 portlet context 时有效。  

## Spring Bean 生命周期

**实例化**

> 实例化一个 Bean， 也就是我们常说的 new。

**IOC 依赖注入**

> 按照 Spring 上下文对实例化的 Bean 进行配置， 也就是 IOC 注入。

**setBeanName 实现**

> 如果这个 Bean 已经实现了 BeanNameAware 接口，会调用它实现的 setBeanName(String)
> 方法，此处传递的就是 Spring 配置文件中 Bean 的 id 值

**BeanFactoryAware 实现**

> 如果这个 Bean 已经实现了 BeanFactoryAware 接口，会调用它实现的 setBeanFactory，
> setBeanFactory(BeanFactory)传递的是 Spring 工厂自身（可以用这个方式来获取其它 Bean，
> 只需在 Spring 配置文件中配置一个普通的 Bean 就可以）。

**ApplicationContextAware 实现**

> 如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调用
> setApplicationContext(ApplicationContext)方法，传入 Spring 上下文（同样这个方式也
> 可以实现步骤 4 的内容，但比 4 更好，因为 ApplicationContext 是 BeanFactory 的子接
> 口，有更多的实现方法）

**postProcessBeforeInitialization 接口实现-初始化预处理**

> 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用
> postProcessBeforeInitialization(Object obj, String s)方法， BeanPostProcessor 经常被用
> 作是 Bean 内容的更改，并且由于这个是在 Bean 初始化结束时调用那个的方法，也可以被应
> 用于内存或缓存技术。

**init-method**

> 如果 Bean 在 Spring 配置文件中配置了 init-method 属性会自动调用其配置的初始化方法。

**postProcessAfterInitialization**

> 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用
> postProcessAfterInitialization(Object obj, String s)方法。
> 注： 以上工作完成以后就可以应用这个 Bean 了，那这个 Bean 是一个 Singleton 的，所以一
> 般情况下我们调用同一个 id 的 Bean 会是在内容地址相同的实例，当然在 Spring 配置文件中
> 也可以配置非 Singleton。

**Destroy 过期自动清理阶段**

> 当 Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调
> 用那个其实现的 destroy()方法；

**destroy-method 自配置清理**

> 最后，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的
> 销毁方法  

![image-20200315232841661](..\file\SpringBean生命周期.png)

**bean 标签有两个重要的属性（init-method 和 destroy-method）。用它们你可以自己定制**
**初始化和注销方法。它们也有相应的注解（@PostConstruct 和@PreDestroy） 。**

```
<bean id="" class="" init-method="初始化方法" destroy-method="销毁方法">  
```



## Spring AOP 原理  

> 面向切面编程（AOP）和面向对象编程（OOP）类似，也是一种编程模式。[Spring](http://c.biancheng.net/spring/) AOP 是基于 AOP 编程模式的一个框架，它的使用有效减少了系统间的重复代码，达到了模块间的松耦合目的。

> AOP 的全称是“Aspect Oriented Programming”，即面向切面编程，它将业务逻辑的各个部分进行隔离，使开发人员在编写业务逻辑时可以专心于核心业务，从而提高了开发效率。

> AOP 采取横向抽取机制，取代了传统纵向继承体系的重复性代码，其应用主要体现在事务处理、日志管理、权限控制、异常处理等方面。

> 目前最流行的 AOP 框架有两个，分别为 Spring AOP 和 AspectJ。

> Spring AOP 使用纯 [Java](http://c.biancheng.net/java/) 实现，不需要专门的编译过程和类加载器，在运行期间通过代理方式向目标类植入增强的代码。

> AspectJ 是一个基于 Java 语言的 AOP 框架，从 Spring 2.0 开始，Spring AOP 引入了对 AspectJ 的支持。AspectJ 扩展了 Java 语言，提供了一个专门的编译器，在编译时提供横向代码的植入。

### AOP 核心概念  

| 名称                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| Joinpoint（连接点） | 被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。 |
| Pointcut（切入点）  | 指要对哪些 Joinpoint 进行拦截，即被拦截的连接点。            |
| Advice（通知）      | 指拦截到 Joinpoint 之后要做的事情，即对切入点增强的内容。    |
| Target（目标）      | 指代理的目标对象。                                           |
| Weaving（植入）     | 织入，将通知等织入代理类。Spring AOP是动态织入（运行时织入），AspectJ则是静态织入（编译时织入） |
| Proxy（代理）       | AOP代理对象，由JDK动态代理或CGLIB代理生成                    |
| Aspect（切面）      | 切面，在Spring中意为所有通知方法所在的类                     |

### AOP 两种代理方式  

> ​	Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式 生成由
> AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。 默认的策略是如果目标类是接口，
> 则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。  

**JDK 动态接口代理**

> ​	JDK 动态代理主要涉及到 java.lang.reflect 包中的两个类： Proxy 和 InvocationHandler。
> InvocationHandler 是一个接口，通过实现该接口定义横切逻辑，并通过反射机制调用目标类
> 的代码，动态将横切逻辑和业务逻辑编制在一起。 Proxy 利用 InvocationHandler 动态创建
> 一个符合某一接口的实例，生成目标类的代理对象。

**CGLib 动态代理**

> ​	CGLib 全称为 Code Generation Library，是一个强大的高性能， 高质量的代码生成类库，
> 可以在运行期扩展 Java 类与实现 Java 接口， CGLib 封装了 asm，可以再运行期动态生成新
> 的 class。和 JDK 动态代理相比较： JDK 创建代理有一个限制，就是只能为接口创建代理实例，
> 而对于没有通过接口定义业务方法的类，则可以通过 CGLib 创建动态代理。  