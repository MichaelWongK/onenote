| title                 | date       | categories | tags   |
| --------------------- | ---------- | ---------- | ------ |
| Spring 源码——循环依赖 | 2020-04-14 | 框架相关   | spring |

# Spring 源码——循环依赖

> Spring 源码学习，一方面提升读源码能力、向大师学习；一方面便于更深入地使用、拓展 Spring。

- 本人呢，还是喜欢把学的记录下来，写的过程，也是加深理解的过程。毕竟你可能只是感觉会了。
- 编译了 Spring 源码，学着就更方便了

## 循环依赖是什么？为什么学循环依赖的源码？

> 循环依赖很常见，众多的业务场景，两块业务相互依赖也是很常见的。 如果 Spring 不支持循环依赖，那出现循环依赖场景时，项目就无法启动了——依赖注入死循环。 （当然，单例情况下，Spring默认是支持循环依赖的，你也可以通过其他方式关闭循环依赖）

- 假设有 A、B、C 三个类，他们之间项目依赖
  - A 依赖 B 和 C
  - B 依赖 A 和 C
  - C 依赖 A 和 B
- 此时你觉得它们的初始化完成顺序是什么样的呢？
- 源码见附录

## 怎么学习呢？当然是看本文了 ^_^

> 路神视频与博客如下

- [B站——2020年最新spring源码解析](https://www.bilibili.com/video/BV1VJ41167Ec)
- [CSDN——子路老师博客](https://me.csdn.net/java_lyvee)

## 简述循环依赖过程

1. `.java` 编译成 `.class` 文件，ClassLoader 加载 `.class` 文件
2. Spring 容器实例化
3. 扫描符合规则的类（比如添加了@Controller、@Service等等注解的类、@Bean注解的方法）
4. 解析这些类（比如拿到这些类的@Scope、@Lazy等等）
5. 拿到了这些解析的信息放哪呢？BeanDefinition（下边简称BD）闪亮登场，实例化BD，写入解析的信息
6. 将BD信息放入Map中缓存起来，ConcurrentHashMap<beanName, BeanDefinition> beanDefinitionMap
7. 调用bean工厂后置处理器
8. 验证BD，如果是Lazy、ProtoType、Abstract的，就不用执行下边的了。
9. 重点来了，Bean的生命周期 ↓↓↓↓↓↓
10. 首先推断A的构造方法，通过反射创建对象，此时还是空的对象
    - 空对象创建完，往Set集合里放入正在创建的beanName，后边有用 singletonsCurrentlyInCreation
11. 判断是否允许循环依赖，允许的话
    - 就将当前对象封装成 ObjectFactory，放进二级缓存
    - 如果对象需要AOP，那就将代理后的对象封装成 ObjectFactory，放进二级缓存 singletonFactories
12. 接着就是依赖注入了，是循环依赖的核心
    - 拿到@Resource、@Autowired注解的类，进行注入
    - 初始化A，需要注入B和C。
      - A注入B，就要getSingleton(b)，因为b未创建，所以拿到的肯定是空
      - 此时就会走创建B的生命周期(10~18)，此时应该能加深理解递归、以及栈的概念了
      - B走到当前这步，需要注入A，C。
        - 注入A，getSingleton(a)，单例池拿A，肯定拿不到，判断A是否在创建中（第10步存的），
        - A在创建中，就从二级缓存中拿到由A封装的ObjectFactory，getObject() 拿到A对象
        - 如果拿到A需要AOP，此时拿到的是CGLIB代理的A对象，A对象注入完成
        - 注入C，和B差不多，走Bean的生命周期，反射创建对象，完成依赖注入，此时A、B都能拿到了
        - C 最先初始化成Bean，开始依次出栈
      - B第二初始化成Bean，继续出栈
    - A注入B完成
    - A注入C，getSingleton(c)，C已经是完整的Bean了，注入C完成
    - 至此，A、B、C初始化完成
13. 依赖注入完成了，开始走Bean的生命周期回调
14. 生命周期回调1——回调实现ApplicationContextAware接口的方法setApplicationContext
15. 生命周期回调2——回调被@PostConstruct注解的方法
16. 生命周期回调3——回调实现InitializingBean接口的方法afterPropertiesSet
17. 如果Bean需要AOP，此处完成相应AOP代理
18. 将初始化完成的Bean放入单例池，也就是一级缓存 singletonObjects

![image-20200804220514873](..\..\imageFiles\image-20200804220514873.png)

## 附录

- `A.java` 如下（`B.java`、`C.java`类似）

```
@Component
public class A {

	@Resource
	private B b;
	@Resource
	private C c;

	public A() {
		System.out.println("init " + getClass().getName());
	}

	@PostConstruct
	public void callback() {
		System.out.println("\t" + getClass().getSimpleName() + " Init Finish!");
	}

}
```

- 启动类 `SecondApplication.java`

```
@ComponentScan("com.amos.cr.second")
public class SecondApplication {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext context =
				new AnnotationConfigApplicationContext(SecondApplication.class);
		System.out.println("Spring Bean 默认初始化完成");
	}

}
```

- 运行结果

```
init com.amos.cr.second.service.A
init com.amos.cr.second.service.B
init com.amos.cr.second.service.C
	C Init Finish!
	B Init Finish!
	A Init Finish!
Spring Bean 默认初始化完成
```