# Linux系统下安装nginx
![img](https://github.com/MichaelWongK/onenote/blob/master/linux/nginx/images/20190316103605490.png)

## 前言：

在实际开发项目中有可能会经常用到nginx，你们也许会拿它做负载均衡，或者转发请求做动静分离，用来加载前端项目；或者解决跨域的问题等等，今天这篇文章就讲解下在Linux下如何安装nginx。

## 正文：

### 一、安装前准备

我们先检验下系统是否安装g++、gcc、openssl-devel、pcre-devel和zlib-devel，这些安装nginx所需要的依赖包。

```
yum list installed | grep gcc //这个指令既可以检查gcc又可以检查g++
yum list installed | grep openssl-devel
yum list installed | grep pcre-devel
yum list installed | grep zlib-devel
```



#### 知识补充：

因为Linux下软件安装的方式很多，没有一个通用的方式查看软件包是否安装，所以总结下来就是这几类。

rpm包安装的，可以用 rpm -qa 看到，如果要查找某软件包是否安装，用 rpm -qa | grep "软件或者包的名字"
 以deb包安装的，可以用 dpkg -l 看到。如果是查找指定软件包，用 dpkg -l | grep "软件或者包的名字"
yum方法安装的，可以用 yum list installed 查找，如果是查找指定包，用 yum list installed | grep "软件名或者包名"
如果是以源码包自己编译安装的，例如.tar.gz或者tar.bz2形式的，这个只能看可执行文件是否存在了。
我经常用的安装方式是yum安装依赖包，tar去安装软件包，比如安装nginx。

运行结果如下图：

![img](https://img-blog.csdnimg.cn/20190315224710236.png)

如果没有安装的话，就先安装一下。

#### 1、gcc安装

安装nginx需要将nginx的源码进行编译，编译依赖gcc环境，所以需要安装gcc，指令：

```
yum install gcc-c++
```



#### 2、 pcre pcre-devel 安装

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库，指令：

```
yum install -y pcre pcre-devel
```



#### 3、 zlib 安装

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库，指令：

```
yum install -y zlib zlib-devel
```



#### 4、 openSSL 安装

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库，指令：

```
yum install -y openssl openssl-devel
```



### 二、准备好上面的依赖环境就可以正式安装啦

#### 1、把安装包通过sftp上传到服务器

![Image text](https://github.com/MichaelWongK/onenote/blob/master/linux/nginx/images/1572590074246.png)

#### 2、 解压

```
tar -zxvf nginx-1.8.1.tar.gz
```


 知识补充：

源码的安装一般由3个步骤组成：配置（configure）、编译（make）、安装（make install）。

执行这几个命令时一定要到解压后的nginx文件夹下执行（这里是../nginx-1.8.1下）

#### 3、配置

--prefix是指定安装的位置，执行编译、安装（make，make install）成功后，生成的配置文件（conf文件夹）和启动文件（sbin文件夹）等会放到生成的nginx这个文件夹里（nginx文件夹的名字是可以自定义的），这样做的好处就是方便以后的维护。因为如果不设置的话，直接执行./configure，会把文件生成到/usr/local/nginx下（但有的时候你可能没有对这个（usr）文件夹下的文件有操作权限，所以建议指定到你有操作权限的文件夹(dada），指令：

```
./configure --prefix=/dada/nginx-1.8.1/nginx
```



#### 4、编译，指令：

```
make
```



#### 5、安装，指令：

```
make install
```



#### 6、执行命令结束

你在指定的文件夹里的找到了生成的conf和sbin文件，就说明基本安装成功。（_temp是启动后生成的文件）

![1572590521561](C:\Users\User\AppData\Roaming\Typora\typora-user-images\1572590521561.png)

#### 7、修改配置文件

然后进到conf里改下配置文件nginx.conf，如果你有root权限，端口可以设置成80（nginx默认端口号），但是如果你没有root权限建议把端口改成，1024以上的端口，比如9090。（在Linux下，默认端口号在1024以下的程序是要在root下才能使用的，在其他用户下，如果尝试使用将会报错）

![img](https://img-blog.csdnimg.cn/20190316103605490.png)

指令：

```
cd /dada/nginx-1.8.1/nginx/conf
//编辑配置文件
vi nginx.conf 
i
esc->:wq
```

#### 8、启动nginx

进到sbin文件后，启动nginx就可以啦，指令：

```
./nginx
```



#### 9、检查nginx状态

启动完成后，通过以下指令查看是否启动成功

```
ps -ef|grep nginx
```


出现如下图的，就证明启动成功啦！

![1572590180943](C:\Users\User\AppData\Roaming\Typora\typora-user-images\1572590180943.png)

#### 10、查看主页

想要本地看到nginx的主页，需要让运维映射下外网ip和端口或者配一下windows下的跳板机（一般情况下，工作中的服务器是内网地址，无法直接访问，所以要做下映射），这样就可以通过浏览器访问nginx的主页。

![1572590267570](C:\Users\User\AppData\Roaming\Typora\typora-user-images\1572590267570.png)

参考文章：

https://blog.csdn.net/huangyimo/article/details/80760908

https://www.cnblogs.com/hdnav/p/7941165.html

https://www.cnblogs.com/yuanqiangfei/p/8033000.html

## 常见安装中出现的问题：

一、如果出现下面的问题，就是说相应的安装包缺失，用yum安装相应的包就行。

![img](https://img-blog.csdnimg.cn/20190316105450546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pka193YW5ndGFpZGE=,size_16,color_FFFFFF,t_70)

这里缺失的是pcre-devel

二、如果出现下面的问题，要不是端口被占用，要不就是权限不够不能用1024下的端口（nginx默认80）

![img](https://img-blog.csdnimg.cn/201903161059004.png)

## 总结：

其实安装软件一般没有太多的坑，基本上就是三大类：①依赖包缺失②权限不够（工作中开发很少有root权限）③配置文件配置不对。
