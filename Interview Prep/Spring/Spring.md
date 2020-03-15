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
> AOP 是一种新的方法论, 是对传统 OOP(Object-Oriented Programming, 面向对象编程) 的补充. AOP 的主要编程对象是切面(aspect), 而切面模块化横切关注点.可以举例通过事务说明.
>
> IOC: Invert Of Control, 控制反转. 也成为 DI(依赖注入)其思想是反转 资源获取的方向. 传统的资源查找方式要求组件向容器发起请求查找资源.作为 回应, 容器适时的返回资源. 而应用了IOC 之后, 则是容器主动地将资源推送 给它所管理的组件,组件所要做的仅是选择一种合适的方式来接受资源. 这种行 为也被称为查找的被动形式

## 在 Spring 中如何配置 Bean ?

> Bean 的配置方式: 通过全类名（反射）、通过工厂方法（静态工厂方法 & 实 例工厂方法）、FactoryBean

## IOC 容器对 Bean 的生命周期:

> ①. 通过构造器或工厂方法创建 Bean 实例
> ②. 为 Bean 的属性设置值和对其他 Bean 的引用
> ③ . 将 Bean 实 例 传 递 给 Bean 后 置 处 理 器 的 postProcessBeforeInitialization 方法 
>
> ④. 调用 Bean 的初始化方法(init-method)  