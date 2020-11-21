# linux安装Docker

**Docker的三大核心概念：镜像、容器、仓库**

镜像：类似虚拟机的镜像、用俗话说就是安装文件。

容器：类似一个轻量级的沙箱，容器是从镜像创建应用运行实例，可以将其启动、开始、停止、删除、而这些容器都是相互隔离、互不可见的。

仓库：类似代码仓库，是Docker集中存放镜像文件的场所。

**前置条件：**

- 64-bit 系统
- kernel 3.10+

**1.检查内核版本，返回的值大于3.10即可。**

```
$ uname -r
```

![](..\..\imageFiles\image-20200730091701685.png)

**2.使用 sudo 或 root 权限的用户登入终端。**

**3.确保yum是最新的**

```
$ yum update
```

**4.安装docker**

- 安装依赖包

```kotlin
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 
```

- 设置阿里云镜像源

```csharp
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```

- 安装 Docker-CE

```undefined
sudo yum install docker-ce
```

- 若报错

```ruby
--> Processing Conflict: 1:docker-ce-cli-18.09.6-3.el7.x86_64 conflicts docker
--> Processing Conflict: 1:docker-ce-cli-18.09.6-3.el7.x86_64 conflicts docker-io
--> Processing Conflict: 3:docker-ce-18.09.6-3.el7.x86_64 conflicts docker
--> Processing Conflict: 3:docker-ce-18.09.6-3.el7.x86_64 conflicts docker-io
--> Finished Dependency Resolution
Error: docker-ce conflicts with 2:docker-1.13.1-96.gitb2f74b2.el7.centos.x86_64
Error: docker-ce-cli conflicts with 2:docker-1.13.1-96.gitb2f74b2.el7.centos.x86_64
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

- 解决办法

```csharp
# 1、查看安装过的docker：
yum list installed | grep docker
docker.x86_64                           2:1.13.1-74.git6e3bb8e.el7.centos
docker-client.x86_64                    2:1.13.1-74.git6e3bb8e.el7.centos
docker-common.x86_64                    2:1.13.1-74.git6e3bb8e.el7.centos
# 2、卸载docker：
sudo yum remove -y docker-ce.x86_64 docker-client.x86_64 docker-common.x86_64
# 3、删除容器镜像：
sudo rm -rf /var/lib/docker
# 4、 重新安装docker
sudo yum install docker-ce
```

- 安装docker

```
yum -y install docker
```

- 启动docker

```bash
# 开机自启
sudo systemctl enable docker 
# 启动docker服务  
sudo systemctl start docker
```

- 添加docker用户组（可选）

```bash
# 1. 建立 Docker 用户组
sudo groupadd docker
# 2.添加当前用户到 docker 组
sudo usermod -aG docker $USER
```

- 镜像加速配置

```bash
# 加速器地址 ：
# 阿里云控制台搜索容器镜像服务
# 进入容器镜像服务， 左侧最下方容器镜像服务中复制加速器地址
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["你的加速器地址"]
}
EOF
# 重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 拉取镜像

- 镜像拉取地址：

  dockerhub 

  ![img](https:////upload-images.jianshu.io/upload_images/9494436-2a2035d70223703e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



```objectivec
# 下载镜像：docker pull <镜像名:tag>    如：下载centos镜像
docker pull centos
docker pull sameersbn/redmine:latest
# 查看已下载镜像
docker images
# 删除容器
docker rm <容器名 or ID>
# 查看容器日志
docker logs -f <容器名 or ID>
# 查看正在运行的容器
docker ps
# 查看所有的容器，包括已经停止的。
docker ps -a 
# 删除所有容器
docker rm $(docker ps -a -q)
# 停止、启动、杀死指定容器
docker start <容器名 or ID> # 启动容器
docker stop <容器名 or ID> # 启动容器
docker kill <容器名 or ID> # 杀死容器
# 后台运行 docker run -d <Other Parameters>
docker run -d -p 127.0.0.1:33301:22 centos6-ssh
# 暴露端口： 一共有三种形式进行端口映射
docker -p ip:hostPort:containerPort # 映射指定地址的主机端口到容器端口
# 例如：docker -p 127.0.0.1:3306:3306 映射本机3306端口到容器的3306端口
docker -p ip::containerPort # 映射指定地址的任意可用端口到容器端口
# 例如：docker -p 127.0.0.1::3306 映射本机的随机可用端口到容器3306端口
docer -p hostPort:containerPort # 映射本机的指定端口到容器的指定端口
# 例如：docker -p 3306:3306 # 映射本机的3306端口到容器的3306端口
# 映射数据卷
docker -v /home/data:/opt/data # 这里/home/data 指的是宿主机的目录地址，后者则是容器的目录地址
```



## 导出导入镜像

此处以mysql为例（docker安装mysql）

* 找一台已经安装mysql的linux服务 

  **执行命令：docker images**

<img src="..\..\imageFiles\image-20200730144620164.png" alt="image-20200730144620164" style="zoom: 70%;" />

​		**执行导出命令**

```
	// docker save -o 导出得镜像名称 .gz docker镜像名称
	// 导出得镜像名称 docker镜像名称中间有空格，不能有点比如：mysql5.7.gz）
	docker save -o mysql-temp.gz mysql
```

​		**导出完成后文件会生成在根目录下或者root下**

<img src="..\..\imageFiles\image-20200730144933735.png" alt="image-20200730144933735" style="zoom:80%;" />

​		**将生成的mysql-temp.gz 文件导入需要安装的linux**

- 执行导入命令 进入mysql-temp.gz所在目录执行命令 docker load -i mysql-temp.gz

  **导入后执行 docker images 查看镜像，导入成功**

  <img src="..\..\imageFiles\image-20200730145408718.png" alt="image-20200730145408718" style="zoom:80%;" />

> 启动请参考[Docker安装mysql](Docker安装mysql)

## GUI管理

这里推荐使用 Portainer 作为容器的 GUI 管理方案。
 官方地址：[https://portainer.io/install.html](https://links.jianshu.com/go?to=https%3A%2F%2Fportainer.io%2Finstall.html)
 安装命令：

```jsx
docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

访问你的 IP:9000 即可进入容器管理页面。

![img](..\..\imageFiles\aaaaaaaaaa.png)





































