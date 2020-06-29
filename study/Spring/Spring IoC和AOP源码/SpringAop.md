# SpringAop

> AOP本质：在不改变原有业务逻辑的情况下增强横切逻辑，横切逻辑代码往往是权限校验代码、⽇志代码、事务控制代码、性能监控代码  



## Spring Aop 两种动态代理方式

> Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式 生成由AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。 

### JDK 动态接口代理

> JDK 动态代理主要涉及到 java.lang.reflect 包中的两个类： Proxy 和 InvocationHandler。
> InvocationHandler 是一个接口，通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态将横切逻辑和业务逻辑编制在一起。 Proxy 利用 InvocationHandler 动态创建一个符合某一接口的实例，生成目标类的代理对象。

### CGLib 动态代理

> CGLib 全称为 Code Generation Library，是一个强大的高性能， 高质量的代码生成类库，可以在运行期扩展 Java 类与实现 Java 接口， CGLib 封装了 asm，可以再运行期动态生成新的 class。和 JDK 动态代理相比较： JDK 创建代理有一个限制，就是只能为接口创建代理实例，而对于没有通过接口定义业务方法的类，则可以通过 CGLib 创建动态代理。  

### 采用规则及区别：

1. 如果目标对象实现了接口，则默认采用JDK动态代理

2. 如果目标对象没有实现接口，则采用Cglib进行动态代
3. 如果目标对象实现了接口，且强制cglib代理，则使用cglib代理

```
@SpringBootApplication
// 强制使用cglib代理
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ProxyDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProxyDemoApplication.class, args);
    }

}
```



## 责任连模式-多个AOP如何叠加



![image-20200625204813706](..\..\..\imageFiles\aop-handle.png)





## Spring Aop 应用源码解析



![image-20200625220349516](..\..\..\imageFiles\aop-源码大纲.png)







