## 多容器管理(docker-compose)：

Dockerfile 用来构建 Docker 镜像，那么 docker-compose 则是用来创建容器的。 

Docker 有三个主要的功能：Build、Ship 和 Run，使用 docker-compose 可以帮我们在 Run 的层面解决很多实际问题。

docker-compose 通过一个 yaml 模板文件来统一管理多个容器的配置，如网络、数据卷、执行指令、环境变量、资源限制等等。

有了 docker-compose 我们便可以一键重启、关闭、删除、监控所有的 docker 服务，只需要一次配置，则可以对容器进行统一管理，那

么此时我们则不必为了每次要运行一堆容器时写大量的命令而头疼。

## 安装 docker-compose：

方式一:

```
curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

## 高速源
curl -L https://get.daocloud.io/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose version # 查看版本号，测试是否安装成功
你可以通过修改URL中的版本，可以自定义您的需要的版本。
```

方式二:

```css
1、安装python-pip

yum -y install epel-release

yum -y install python-pip

2、安装docker-compose

pip install docker-compose

待安装完成后，执行查询版本的命令确认安装成功

docker-compose version

spring.dubbo

application.name

registry.port
```



## 配置 docker-compose.yml 文件(注意: 冒号 -号后必须空格, 各级别必须对其)

```bash
version: '2' # docker 的版本

services: # 配置的容器列表

CONTAINER_NAME: # 容器的名称

image: BASE_IMAGE # 这个一个容器的基础镜像

ports: # 你的容器需不需要做端口映射

- "host_port:container_port"

volumes: # 数据卷配置

- host_dir:container_dir

environment: # 环境变量(map 的配置方式 key: value)

PARAM: VALUE

environments: # 环境变量(数组的配置方式 - key=value)

- PARAM=VALUE

restart: always # 容器的重启策略

dns: # dns 的配置

- 8.8.8.8
```



















