## 拉取镜像

```
docker pull jenkins
```

## 启动

```
docker run -d --restart=always --name=jenkins-master \ 
-p 18080:8080 -p 50000:50000 \ 
-v $PWD:/var/jenkins_home \ 
-v /run/docker.sock:/var/run/docker.sock \ 
-v /usr/bin/docker:/usr/bin/docker \ 
-v /var/local/jdk1.8.0_191/bin/java:/usr/bin/jdk1.8.0_191/java \
-v /var/local/jdk1.8.0_191:/var/local/jdk1.8.0_191 \ 
-v /var/local/apache-maven-3.6.0:/var/local/apache-maven-3.6.0 \ 
--env JAVA_OPTS="-Xms256m -Xmx256m -XX:CompressedClassSpaceSize=128m -XX:MetaspaceSize=200m -XX:MaxMetaspaceSize=200m" \ 
-u 0 \ 
jenkins/jenkins:lts

# -p 指定暴露的端口，8080用于对外暴露服务，而50000适用于Jenkins slave的注册
# 第一个 -v 指定挂载的jenkins home中的内容到本地，这里我们指定当前目录$PWD
# 第二个 -v 用于将本地的docker挂载到容器中，实现在容器中使用docker命令
# --env可以指定很多参数，这里我们指定jenkins的contextpath 为 jenkins
# -u 指定启动容器账号为root账号
```

## docker中Jenkins容器启动失败

![image-20201105102020149](D:\workspace\git\onenote\imageFiles\image-20201105102020149.png)

原因是Jenkins镜像内部使用的用户是jenkons，但是我们启动容器时的账号是root，导致没有权限操作内部目录，我们可以稍微改一下上面的命令：

增加启动参数

```
-u 0 
```

这命令的意思是覆盖容器中内置的帐号，该用外部传入，这里传入0代表的是root帐号Id。这样再启动的时候就应该没问题了

## getting start

访问jenkins主页，可以看到在下面配置目录下生成了正确的密码文件

![image-20201105104713952](D:\workspace\git\onenote\imageFiles\image-20201105104713952.png)

注：初始密码文件在docker容器中

![image-20201105105626410](D:\workspace\git\onenote\imageFiles\image-20201105105626410.png)

## java.io.IOException: Jackson 2 API Plugin v2.10.0 failed to load

必须从jenkins v2.60升级到v2.138.
造成这一步的原因是镜像不是使用最新的版本导致的，直接使用如下命令会报错

```shell
docker pull jenkins:latest 这是错误的
```

首先在docker官方仓库网页搜 索jenkins的镜像。注意，要安装的是 jenkins/jenins:lts，不是jenins，后者已经被官方废弃了。
应该安装如下

```shell
docker pull jenkins/jenkins:lts
```

## 根据提示要求完成安装。下图为安装默认推荐插件。

