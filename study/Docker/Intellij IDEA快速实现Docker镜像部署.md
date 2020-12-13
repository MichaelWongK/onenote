# Intellij IDEA快速实现Docker镜像部署

## Docker开启远程访问

```

[root@VM-0-8-centos ~]# vim /lib/systemd/system/docker.service

#修改ExecStart这行
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

改成
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
```

```
重新加载配置文件
[root@VM-0-8-centos ~]# systemctl daemon-reload    
#重启服务
[root@VM-0-8-centos ~]# systemctl restart docker.service 
#查看端口是否开启
[root@VM-0-8-centos ~]# netstat -nlpt
#直接curl看是否生效
[root@VM-0-8-centos ~]# curl http://127.0.0.1:2375/info
```

**注：**阿里云/腾讯云/华为云等云服务器需要开启安全组 2375 端口

**如果远程还是无法访问2375端口：**

```
防火墙开放2375端口号
[root@VM-0-8-centos ~]# firewall-cmd --zone=public --add-port=2375/tcp --permanent
重启防火墙
[root@VM-0-8-centos ~]# firewall-cmd --reload 

或者关闭防火墙
[root@VM-0-8-centos ~]# chkconfig iptables off
```



补充：

***\*Firewall\**\**关闭常见端口命令：\****

```
firewall-cmd --zone=public --remove-port=2375/tcp --permanent
```



## **Intellij IDEA安装Docker插件**

### 打开小扳手（setting）找到docker，输入虚拟机ip，不出意外会连接成功的。

![docker-idea-settings.png](http://www.micheal.wang:10020/mongo/read/5fcde3da420630475f381463)



### Springboot项目添加docker-maven-plugin插件

```
<!--使用docker-maven-plugin插件-->
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>

    <!--将插件绑定在某个phase执行-->
    <executions>
        <execution>
            <id>build-image</id>
            <!--将插件绑定在package这个phase上。也就是说，用户只需执行mvn package ，就会自动执行mvn docker:build-->
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!--指定生成的镜像名-->
        <imageName>micheal.wang/${project.artifactId}</imageName>
        <!--指定标签-->
        <imageTags>
            <imageTag>latest</imageTag>
        </imageTags>
        <!-- 指定 Dockerfile 路径-->
        <dockerDirectory>${project.basedir}/docker</dockerDirectory>

        <!--指定远程 docker api地址-->
        <dockerHost>http://micheal.wang:2375</dockerHost>

        <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <!--jar 包所在的路径  此处配置的 即对应 target 目录-->
                <directory>${project.build.directory}</directory>
                <!-- 需要包含的 jar包 ，这里对应的是 Dockerfile中添加的文件名　-->
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```



### 创建docker文件夹和Dockerfile文件，

```
Dockerfile要与pom.xml中configuration配置的路径一致
<dockerDirectory>${project.basedir}/docker</dockerDirectory>
```



### Dockerfile的内容

```
FROM openjdk:8-jre

VOLUME /tmp/myshop/provider-admin-service:/tmp

ENV APP_VERSION 0.0.1-SNAPSHOT

 # 设置时区
RUN echo "Asia/Shanghai" > /etc/timezone

RUN mkdir /app

COPY provider-admin-service-$APP_VERSION.jar /app/app.jar

ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar", "--spring.profiles.active=prod"]

EXPOSE 20880
```



## **创建Docker镜像**

###  将工程打包，打开**idea Terminal**

> 执行mvn clean package命令进行编译打包，打包后会在target目录下生成jar包。
>
> 生成jar包后，自动生成镜像推送至远程docker

​	执行成功后，可以远程docker上看到这个镜像：

![docker-images](http://www.micheal.wang:10020/mongo/read/5fcdf81b420630475f381465)



### 编写docker-compose.yml文件

```
version: '3.1'
services:
  provider-admin-service:
    restart: always
    image: micheal.wang/provider-admin-service
    container_name: provider-admin-service
    ports:
      - 20880:20880
```



### 上传至服务器并执行

```
[root@VM-0-8-centos docker]# docker-compose up -d
```



### 执行docedocker ps可以看到，镜像已经生产容器开始运行：

![docker-ps](http://www.micheal.wang:10020/mongo/read/5fcdf89d420630475f381467)





