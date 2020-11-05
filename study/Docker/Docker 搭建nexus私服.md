# Docker 搭建nexus私服

# **一、概述**

安装Docker （略）

# **二、安装nexus**

## **拉取镜像**

```javascript
docker pull sonatype/nexus3
```

## **持久化目录**

```javascript
mkdir -p /data/nexus/data
chmod 777 -R /data/nexus/data
```

## **启动镜像**

```javascript
docker run -d -p 8081:8081 --name nexus -v /data/nexus/data:/nexus-data sonatype/nexus3
```

**指定数据卷后启动，可能会报一些权限错误，导致启动不起来。可能会需要修改文件夹权限**

```
chmod 777 -R /data/nexus/data
```

### **查看日志**

```javascript
docker logs -f nexus
```

输出：

```javascript
2020-11-02 06:11:50,879+0000 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@bf295bd{HTTP/1.1, (http/1.1)}{0.0.0.0:8081}
2020-11-02 06:11:50,879+0000 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.Server - Started @85847ms
2020-11-02 06:11:50,882+0000 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer - 
-------------------------------------------------
Started Sonatype Nexus OSS 3.28.1-01
-------------------------------------------------
```

等待几分钟时间，出现 `Started Sonatype Nexus OSS` 表示启动好了。

# **三、访问nexus**

打开浏览器，访问 `http://micheal.wang:8081/`

![image-20201102144309137](D:\workspace\git\onenote\imageFiles\image-20201102144309137.png)

点击右侧的登录

![image-20201102144333270](D:\workspace\git\onenote\imageFiles\image-20201102144333270.png)

查看管理员`admin`密码

```javascript
# cat /data/nexus/data/admin.password
```

登录

![image-20201102144410501](D:\workspace\git\onenote\imageFiles\image-20201102144410501.png)

开始设置

![image-20201102144739351](D:\workspace\git\onenote\imageFiles\image-20201102144739351.png)

修改密码

![image-20201102144831652](D:\workspace\git\onenote\imageFiles\image-20201102144831652.png)

确认配置

![image-20201102144853388](D:\workspace\git\onenote\imageFiles\image-20201102144853388.png)

# **四、添加阿里云maven代理**

点击`settings->Repository->Repositories`

点击Create repositoty按钮

![image-20201102144946378](D:\workspace\git\onenote\imageFiles\image-20201102144946378.png)

选择maven2 (proxy)

![image-20201102145012612](D:\workspace\git\onenote\imageFiles\image-20201102145012612.png)

填写如下字段，分别是代理库的名称，所代理的上层库的url。阿里云url为：http://maven.aliyun.com/nexus/content/groups/public/

![image-20201102145113180](D:\workspace\git\onenote\imageFiles\image-20201102145113180.png)

滚动到页面最下方，点击“Create repositoty”按钮。

![image-20201102145202269](D:\workspace\git\onenote\imageFiles\image-20201102145202269.png)

可以看到刚刚新建的代理库已经存在了。

![image-20201102145544494](D:\workspace\git\onenote\imageFiles\image-20201102145544494.png)

重新配置maven-public组，使其包含新建的aliyun-maven。在如上页面，点击maven-public，进入到配置页面。按下图进行修改。把aliyun-maven移至右侧，并向上移至第一位。然后点击保存。

![image-20201102145616932](D:\workspace\git\onenote\imageFiles\image-20201102145616932.png)

点击左侧菜单Repositoty>Repositories，进入到仓库列表页面，点击maven-public一行的copy按钮，然后复制弹出的url，后面配置maven时需要使用。

![image-20201102145646098](D:\workspace\git\onenote\imageFiles\image-20201102145646098.png)

# **五、配置maven**

### 1.首先修改maven的setting.xml文件，添加用户信息，以便jar包上传私服时进行身份认证,修改内容如下：



```javascript
<servers>
    <server>
          <id>maven-releases</id>
          <username>admin</username>
          <password>112233</password>
    </server>
    <server>
      <id>maven-snapshots</id>
      <username>admin</username>
      <password>112233</password>
    </server>
</servers>
```

> `id` : 为nexus的仓库名称
>
> `username` : nexus用户名
>
> `password` : nexus密码

注意：修改为自己设置的密码。

增加mirrors

```javascript
<mirror>
  <id>maven-public</id>
  <name>maven-public</name>
    <url>http://micheal.wang:8081/repository/maven-public/</url>
  <mirrorOf>*</mirrorOf>
</mirror>
    
```

注意：修改ip地址为服务器ip

### 2、创建maven项目，修改pom.xml

增加发布管理节点

```xml
<distributionManagement>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <name>maven-snapshots-repository</name>
            <url>http://micheal.wang:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>maven-releases</id>
            <name>maven-releases-repository</name>
            <url>http://micheal.wang:8081/repository/maven-releases/</url>
        </repository>
</distributionManagement>
```

> `snapshotRepository` : 快照仓库
>
> `repository` : 发行仓库
>
> `id` : 与上面的server的id一致
>
> `name` : 随便
>
> `url` : 仓库地址，从nexus中可以找到

### 3、项目打包发布

maven会根据`<version/>`中是否含有`SANPSHOT`来选择是发布到快照仓库，还是发行版仓库

```
项目打包
mvn clean package -Dmaven.test.skip=true
项目发布
mvn deploy
```



# 六、jar包安装到nexus私服

有时项目开发时，一些maven依赖下载不下来，一直报错。这时，可以手动下载jar包，将其安装到nexus私服，再从nexus解决依赖问题。下面以安装`kaptcha`为例：

```
mvn deploy:deploy-file -DgroupId=com.google.code.kaptcha -DartifactId=kaptcha -Dversion=2.3 -Dpackaging=jar -Dfile=D:\kaptcha-2.3.2.jar -Durl=http://micheal.wang:8081/repository/third/ -DrepositoryId=third
```

> `DgroupId` : jar包的groupId
>
> `Dversion` : jar包的版本
>
> `Dfile` : jar包所在位置
>
> `Durl` : 仓库地址
>
> `DrepositoryId` : 仓库名

这里新建了名为`third`的第三方仓库，注意要在`setting.xml`增加一个`server`节点，配置用户名和密码。同时要将`third`仓库加到`maven-public`组中，因为第十步要依赖的是`maven-public`组。将 `third`加入`maven-pulic`组后，只要依赖`maven-public`，便可取到`third`中的jar包。

### 七、从nexus下载依赖

在`pom.xml`中增加如下仓库配置：

```
<repositories>
        <repository>
            <id>nexus</id>
            <name>Nexus Repository</name>
            <url>http://micheal.wang:8081/repository/maven-public/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>Nexus Plugin Repository</name>
            <url>http://micheal.wang:8081/repository/maven-public/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </pluginRepository>
    </pluginRepositories>
```

###### 可以看到 上面的`url`节点填写的都是`maven-public`组的url，而`maven-releases`,`maven-snapshots`,`third`都包含在`maven-public`中，所以能取到三个仓库的内容。