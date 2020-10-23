## 启动：

每个SpringBoot程序都有一个主入口，也就是main方法，main里面调用SpringApplication.run()启动整个spring-boot程序，该方法所在类需要使用@SpringBootApplication注解，以及@ImportResource注解(if need)，@SpringBootApplication包括三个注解，功能如下：

**@EnableAutoConfiguration**：SpringBoot根据应用所声明的依赖来对Spring框架进行自动配置。

**@SpringBootConfiguration(内部为@Configuration)**：被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有bean事务，提供了一个spring的上下文环境。

**@ComponentScan**：组件扫描，可自动发现和装配Bean，默认扫描SpringApplication的run方法里的MqMailApplication.class所在的包路径下文件，所以最好将该启动类放到根包路径下。

![image-20200925143026306](http://www.micheal.wang:10020/mongo/read/5f6d8edbb40e335bf463037f)

![image-20200925143123373](http://www.micheal.wang:10020/mongo/read/5f6d8ef4b40e335bf4630381)

### 首先进入main方法

![image-20200925143609290](http://www.micheal.wang:10020/mongo/read/5f6d8fe3b40e335bf4630383)