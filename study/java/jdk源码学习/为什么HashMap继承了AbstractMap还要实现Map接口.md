# 为什么HashMap继承了AbstractMap还要实现Map接口

**jdk中HashMap的声明如下**

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```

**AbstractMap的声明如下**

```
public abstract class AbstractMap<K,V> implements Map<K,V>
```

这就让人很奇怪了，AbstractMap是实现了Map接口的，HashMap的声明中为什么还要写上实现Map接口（类似的还有很多啊，Java集合类中很多都是这样）？

目前看过几种答案，觉得就下面两个站得住脚：
1、添加Map接口声明是为了Class类的getInterfaces这个方法能够直接获取到Map接口；
2、这就是一个写法上的错误，并没有深意。

**说维护性的**，这样做并没有提升什么维护性。有人说可能以后中间的抽象类会废弃，废弃一个抽象类有什么用，直接把方法实现都去掉不就行了。就算是废弃了也不影响啊，废弃了我在这里写个非public的空的抽象类，估计就改下import就完事了。除非是整个集合类框架大改，否则这写抽象类是不会被废弃了。

**可读性的**，这样反而降低了可读性，你看到这个可能会考虑下HashMap中的put方法到底是哪个的，是AbstraceMap的，还是Map的，实际上这样的层次效果跟单独继承Map一样。

Class类的getInterfaces这个方法的确不能获取到父类实现的接口，如果不写上实现Map接口，这个方法返回的数组中就没有Map.class。
这个理由说得通，因为这样的确在实际中能发挥用，但是我本人还没想到这样做的用处。想想getInterfaces这个方法在哪里用到，判断类型会用isInstance或者instance of，不用接口遍历啊，我自己写的代码中也就jdk动态代理中会用到这个方法，为了动态代理Map而写上implements Map的确可以。不过后来发现这么做可能性不大 ，因为HashMap等集合类是Java1.2中有的，使用jdk动态代理的两个核心类java.lang.reflect.Proxy和java.lang.reflect.InvacationHandler是jdk1.3才有的。所以说第一个答案虽然有理，但是也不怎么有理。

第二个答案，这个倒是找到了依据，具体可以看下这个
http://stackoverflow.com/questions/2165204/why-does-linkedhashsete-extend-hashsete-and-implement-sete
得票最高的答案的回答者说他问了当初写这段代码的 Josh Bloch，得知这就是一个写法错误，我了个去

平时不要像这么写，因为的确没有用，除非你真在某些方面及其依赖Class类getInterefaces方法的结果