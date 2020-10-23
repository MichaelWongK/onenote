 

## **1. 查找Docker Hub上的elasticsearch镜像**

命令：docker search elasticsearch

实例：

![image-20200929164820266](http://www.micheal.wang:10020/mongo/read/5f72f4eab40e335bf463038f)

 

## **2. 拉取官方的镜像**

命令：docker pull elasticsearch:7.9.0（7.9.0是版本号，如果不写版本号，默认拉取latest版本）

实例：

![image-20200929164941986](http://www.micheal.wang:10020/mongo/read/5f72f53ab40e335bf4630391)

 

## **3. 查看本地镜像列表**

命令：docker images | grep elasticsearch

示例：

![image-20200929165052420](http://www.micheal.wang:10020/mongo/read/5f72f583b40e335bf4630393)

  

##  **4. 运行镜像**

命令：docker run -v /etc/localtime:/etc/localtime:ro --name elasticsearch -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.9.0

实例：

![image-20200929165233375](http://www.micheal.wang:10020/mongo/read/5f72f5dbb40e335bf4630395)

 

### 命令说明：

　　--name : 镜像的名称

　　-p 对外的端口

　　-e "discovery.type=single-node" 集群，单个节点

```
# 测试es是否成功
[root@VM-0-8-centos ~]# curl localhost:9200
{
  "name" : "2d70e7fbf624",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "p5rxBNrwSOiOIvBq5u9pjQ",
  "version" : {
    "number" : "7.9.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "a479a2a7fce0389512d6a9361301708b92dff667",
    "build_date" : "2020-08-11T21:36:48.204330Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

# 查看docker stats
```

![微信截图_20200930124513](D:\workspace\git\onenote\imageFiles\_20200930124513.png)

```
# 关闭es，增加内存限制，修改配置文件 -e环境配置修改
docker run  -v /etc/localtime:/etc/localtime:ro --name elasticsearch -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx1536m" elasticsearch:7.9.0
# 查看docker stats
```

![image-20200930125332800](D:\workspace\git\onenote\imageFiles\image-20200930125332800.png)



## 安装es-head插件

es-head插件支持es几个版本。

- Elasticsearch 5.x: `docker run -p 9100:9100 mobz/elasticsearch-head:5`

安装好以后，访问9100端口。此时无法访问：

![image-20200930150409249](D:\workspace\git\onenote\imageFiles\image-20200930150409249.png)

### 配置跨域

使用`docker exec -it <your CONTAINER ID or CONTAINER NAME> bash` 进入ElasticSearch容器。
```
# 配置elasticsearch.yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 重启es服务器，然后再次连接

![image-20200930151945942](D:\workspace\git\onenote\imageFiles\image-20200930151945942.png)



## kibana连接es

![image-20200930125527848](D:\workspace\git\onenote\imageFiles\image-20200930125527848.png)

### kibana.yml

```
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true

# 需要修改elasticsearch.hosts值对应到elasticsearch服务ip
```



### 启动kibana

```bash
docker run -d -it --restart=always --privileged=true --name=kibana -p 5601:5601 -v/data/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.9.0
方式二：
docker run --name kibana -e ELASTICSEARCH_URL=http://micheal.wang:9200 -p 5601:5601 -d kibana:7.9.0
```

```
补充 kibana.yml详细配置:
elasticsearch.hosts: ['http://172.16.10.202:9200','http://172.16.10.202:9202', 'http://172.16.10.202:9203']
#发给es的查询记录 需要日志等级是verbose=true 
elasticsearch.logQueries: true
#连接es的超时时间 单位毫秒
elasticsearch.pingTimeout: 30000
elasticsearch.requestTimeout: 30000
#是否只能使用server.host访问服务
elasticsearch.preserveHost: true
#首页对应的appid
kibana.defaultAppId: "home"
kibana.index: '.kibana'
#存储日志的文件设置
logging.dest: /usr/share/kibana/logs/kibana.log
logging.json: true
#是否只输出错误日志信息
logging.quiet: false
logging.rotate:
  enabled: true
  #日志文件最大大小
  everyBytes: 10485760
  #保留的日志文件个数
  keepFiles: 7
logging.timezone: UTC
logging.verbose: true
monitoring.kibana.collection.enabled: true
xpack.monitoring.collection.enabled: true
#存储持久化数据的位置
path.data: /usr/share/kibana/data
#访问kibana的地址和端口配置 一般使用可访问的服务器地址即可
server.host: 172.16.10.202
#端口默认5601
server.port: 5601
server.name: "kibana"
#配置页面语言
i18n.locale: zh-CN
```

[参考]: https://blog.csdn.net/u010361276/article/details/107695787?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&amp;depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param



## 安装IK分词器

1. 我这es的版本为7.9.0的，所以我下载的ik分词器的版本也是7.9.0,如果版本不是这个的，可以自行从[https://github.com/medcl/elasticsearch-analysis-ik/release](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.5.0/elasticsearch-analysis-ik-6.5.0.zip)s这个地址自己查找，那么我这里使用的就是6.5.0的了
2. 进入es容器 docker exec -it es1 bash
3. 执行下载 wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.0/elasticsearch-analysis-ik-7.9.0.zip
4. 下载完成后，进入plugins文件夹，创建文件ik mkdir ik
5. 将下载的elasticsearch-analysis-ik插件移动到ik文件夹,执行解压,解压完成后，将源文件删除
6. 如果你下载的版本和es的版本不对的话，可以通过更改ik分词器中的 plugin-descriptor.properties来使其OK
7. 重启es容器，至此ik分词器安装成功

```
容器中文乱码：
# 临时解决
export  LANG="en_US.UTF-8"
```



## ES-head 查询406

head 连接Elasticsearch7.9.0时 【数据浏览模块不能显示数据了】 
看一下网络发现406 错误

{"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}

解决方法:

1、进入head安装目录；

2、cd _site/

3、编辑vendor.js 共有两处

```
1. 6886行 /contentType: "application/x-www-form-urlencoded
改成  contentType: "application/json;charset=UTF-8"

2. 7574行 var inspectData = s.contentType === "application/x-www-form-urlencoded" &&
改成  var inspectData = s.contentType === "application/json;charset=UTF-8" &&
```



## docker 容器内部没有 vim 命令

```
apt-get update
apt-get install vim
```

