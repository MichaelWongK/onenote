# Docker部署SpringBoot项目

# 一、环境

（1）首先需要准备linux服务器，没有的话可以安装VMware虚拟机代替
（2）在虚拟机里面安装centos系统
（3）docker的安装，可查看<p><a href="https://github.com/MichaelWongK/onenote/blob/master/study/Docker/Docker linux安装.md">Docker linux安装</a></p>

# 二、部署

部署方式，个人知道的有两种，一种是自己手动将springboot项目通过maven打包成jar包后，自己手动的将jar上传到服务器，通过手动Dockerfile文件的方式的方式部署；还有一种是用idea的dokcer插件，直接通过idea上传到服务器进行部署。

这里先用第一种方式进行部署，首先确定自己的虚拟机里面已经安装了docker,并且保证已经从镜像仓库里面拉取了java镜像和mysql镜像，在已经安装了Centos系统和dokcer的虚拟机里面输入`docker image ls`

![](http://www.micheal.wang:10020/mongo/read/5f32b67c9c5fd27c4ff5d872)

- 如上图需要先拉取java镜像

  一行命令搞定`docker pull 要拉取的镜像`

- 将项目用maven打包成一个jar包

![](http://www.micheal.wang:10020/mongo/read/5f32b7449c5fd27c4ff5d874)

- 创建Dockerfile文件

  ```
  FROM  java:8
  VOLUME  /tmp
  ADD file-mongo-1.0.0.jar filemongo.jar
  EXPOSE 8088
  ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/filemongo.jar"]
  ```

  FROM java：表示安装的java镜像
  VOLUME /tmp：创建/tmp目录并持久化到Docker数据文件夹
  ADD file-mongo-1.0.0.jar  filemongo.jar：将生成的jar包复制并重命名到filemongo.jar
  EXPOSE 8088：对外暴露的接口，也是访问的接口，接口的端口号需要根据springboot项目配置的端口号来设置
  ENTRYPOINT：固定写法，用来运行jar包，相当于java -jar xxx.jar一样

  最后将Dockerfile文件和打包生成的jar包（file-mongo-0.0.1-SNAPSHOT.jar）一起上传到虚拟机里面的Centos系统里面，并且处于同一目录下

![](http://www.micheal.wang:10020/mongo/read/5f32b82b9c5fd27c4ff5d876)

- 构建镜像

  进入Dockerfile文件和打包生成的jar包所在的文件夹，并且输入构建镜像的命令`docker build -t filemongo .`**注意命令后面还有一个小数点**，filemongo是Dockerfile文件里面`ADD file-mongo-SNAPSHOT.jar filemongo.jar`的filemongo，镜像即可构建成功

<img src="http://www.micheal.wang:10020/mongo/read/5f32b8b59c5fd27c4ff5d878" alt="image-20200811232625237" style="zoom:80%;" />

- 查看构建好的镜像

![](http://www.micheal.wang:10020/mongo/read/5f32b67c9c5fd27c4ff5d872)

- 运行镜像

  运行镜像，输入命令 docker  run -d --name filemongo -p 10020:8088 filemongo关于运行镜像命令这里就不解释了。跑起来之后就可以在虚拟机外面通过虚拟机里Centos系统的ip来访问了

  docker ps 查看镜像运行中

![image-20200811233039343](http://www.micheal.wang:10020/mongo/read/5f32b9b89c5fd27c4ff5d87a)





