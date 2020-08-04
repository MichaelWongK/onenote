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



## Docker安装mysql

1.查找镜像：

```
 docker search mysql
```

![image-20200730110331358](..\..\imageFiles\image-20200730110331358.png)

> OFFICIAL ok 代表官方镜像

2.下载镜像（如上一步，可以指定想要的版本，不指定则为最新版）：

```
docker pull mysql
```

3.通过镜像创建容器并运行：

```
docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql --lower_case_table_names=1
```

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=jshx123@! -d -v /data/mysql/:/var/lib/mysql mysql --lower_case_table_names=1
```

- **-p 3306:3306**：将容器的 3306 端口映射到主机的 3306 端口。

- **-v -v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。

  ```mysql需要忽略大小写敏感则修改my.cnf 
  注:参数顺序一定要对，--lower_case_table_names=1要加在镜像名后面，镜像名前面是参数，后面是mysql配置，不然会报错
   lower_case_table_names=1 只能在初始化时配置，不然会报
  查看MySQL官方文档，有记录：
  lower_case_table_names can only be configured when initializing the server. Changing the lower_case_table_names setting after the server is initialized is prohibited.
  只有在初始化的时候设置  lower_case_table_names=1 才有效
  ```

- **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。

- **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。

- **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。



### navicat for mysql 无法连接问题解决

​	此时，用navicat for mysql连接mysql发现报错：Client does not support authentication protocol requested by server。。。

![image-20200730115313591](..\..\imageFiles\image-20200730115313591.png)



**需要开启mysql远程访问权限**

```
docker exex -it container /bin/sh
```

1、登陆mysql

```
mysql -u root -p
```

2、授权远程登录权限

​	修改mysql库的user表，将host项，从localhost改为%。%这里表示的是允许任意host访问，如果只允许某一个ip访问，则可改为相应的ip，比如可以将localhost改为192.168.77.123，这表示只允许局域网的192.168.77.123这个ip远程访问mysql。

```
mysql> use mysql;

mysql> update user set host = '%' where user = 'root';

or

mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'YOUR_PASSWORD' WITH GRANT OPTION;

mysql> select host, user from user;//查看权限

mysql> flush privileges; // 刷新配置
```



## Docker安装mongodb

### 下载镜像

```shell
docker pull registry.docker-cn.com/library/mongo
```

### 创建本地数据文件夹

```shell
mkdir /data/mongodb0
```

### 启动MongoDB容器

```shell
docker run --name mongodb-server -v /data/mongodb0:/data/db -p 27017:27017 -d 镜像ID --auth
```

- -v后面的参数表示把数据文件挂载到宿主机的路径
- -p把mongo端口映射到宿主机的指定端口
- --auth表示连接mongodb需要授权

### 为MongoDB添加管理员用户

- 进入MongoDB控制台

```shell
docker exec -it mongodb-server mongo admin
```

<img src="..\..\imageFiles\image-20200730204530909.png" alt="image-20200730204530909" style="zoom: 75%;" />

- 添加用户命令

```shell
db.createUser({ user: 'micheal', pwd: 'mingkai13', roles: [{role: "userAdminAnyDatabase", db: "admin"}]});

db.createUser({ user: 'micheal.wang', pwd: 'mingkai13', roles: [{role: "readWrite", db: "storage"}]});
```

<img src="..\..\imageFiles\image-20200730204642242.png" alt="image-20200730204642242" style="zoom: 55%;" />

### MongoDB用户权限

#### 内建的角色

1. 数据库用户角色：read、readWrite;
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
7. 内部角色：__system

> 角色说明：
>  **Read**：允许用户读取指定数据库
>  **readWrite**：允许用户读写指定数据库
>  **dbAdmin**：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
>  **userAdmin**：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
>  **clusterAdmin**：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
>  **readAnyDatabase**：只在admin数据库中可用，赋予用户所有数据库的读权限
>  **readWriteAnyDatabase**：只在admin数据库中可用，赋予用户所有数据库的读写权限
>  **userAdminAnyDatabase**：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
>  **dbAdminAnyDatabase**：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
>  **root**：只在admin数据库中可用。超级账号，超级权限

### 副本集形式部署

如果你想单点连接，那么从这里往下不需要看了，如果你想搭建MongoDB副本集，那么请移除MongoDB的所有docker容器，重新按照下面的方式启动容器和设置

#### 0. 创建用于auth的keyfile

```shell
mkdir -p /data/mongodb0_conf
cd /data/mongodb0_conf
openssl rand -base64 741 > mongodb-keyfile
chmod 600 mongodb-keyfile
chown 999 mongodb_keyfile
```

#### 1. 启动三个mongodb进程

```shell
docker run --name mongodb-server0 \
--restart always \
-v /data/mongodb0:/data/db \
-v /data/mongodb0_conf:/opt/keyfile \
-p 27017:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--replSet exuehui-mongo-set

docker run --name mongodb-server1 \
--restart always \
-v /data/mongodb1:/data/db \
-v /data/mongodb1_conf:/opt/keyfile \
-p 27018:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--replSet exuehui-mongo-set

docker run --name mongodb-server0 \
--restart always \
-v /data/mongodb0:/data/db \
-v /data/mongodb0_conf:/opt/keyfile \
-p 37017:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--replSet exuehui-mongo-set
```

#### 2 进入 mongodb docker

```shell
docker run -it --name mongo-client mongo /bin/bash
```

#### 3 进入要作为master数据库的 mongodb shell

```shell
mongo 192.168.31.206:27017/admin
```

#### 4 初始化副本集, _id和启动时设置的replSet参数相同

```shell
rs.initiate({ _id:"exuehui-mongo-set", members:[
{_id:0,host:"home.lemonsoft.vip:27017"}, {_id:1,host:"home.lemonsoft.vip:27018"}, {_id:2,host:"home.lemonsoft.vip:37017"}
]})
```

#### 5 查看副本集状态

```shell
rs.status()
```

#### 6.创建权限用户

```shell
use admin;
db.createUser({ user: '1iURI', pwd: 'rootroot', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

use exuehui
db.createUser({ user: '1iURI-exuehui', pwd: 'rootroot', roles: [ { role: "readWrite", db: "exuehui" } ] });
```

#### 7. 重新运行容器，并添加--auth

```shell
docker run --name mongodb-server0 \
--restart always \
-v /data/mongodb0:/data/db \
-v /data/mongodb0_conf:/opt/keyfile \
-p 27017:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--auth \
--replSet exuehui-mongo-set

docker run --name mongodb-server1 \
--restart always \
-v /data/mongodb1:/data/db \
-v /data/mongodb1_conf:/opt/keyfile \
-p 27018:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--auth \
--replSet exuehui-mongo-set

docker run --name mongodb-server0 \
--restart always \
-v /data/mongodb0:/data/db \
-v /data/mongodb0_conf:/opt/keyfile \
-p 37017:27017 \
-d d22 \
--smallfiles \
--keyFile /opt/keyfile/mongodb-keyfile \
--auth \
--replSet exuehui-mongo-set
```

































