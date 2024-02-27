## Docker安装mysql

### 1.查找镜像：

```
 docker search mysql
```

![image-20200730110331358](D:/workspace/github/onenote/imageFiles/image-20200730110331358.png)

> OFFICIAL ok 代表官方镜像

### 2.下载镜像（如上一步，可以指定想要的版本，不指定则为最新版）：

```
docker pull mysql
```

### 3.通过镜像创建容器并运行：

```
docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql --lower_case_table_names=1
```

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mingkai13 -d -v /data/mysql/:/var/lib/mysql mysql --lower_case_table_names=1 --default_authentication_plugin=mysql_native_password
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

- **--default_authentication_plugin=mysql_native_password：**设置默认加密规则



### navicat for mysql 无法连接问题解决

​	此时，用navicat for mysql连接mysql发现报错：Client does not support authentication protocol requested by server。。。

![image-20200730115313591](D:/workspace/github/onenote/imageFiles/image-20200730115313591.png)



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

方式二：创建新用户，采用naviat_password 连接方式。赋予新用户root管理员的全部权限。

CREATE USER 'finley'@'%' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON *.* TO 'finley'@'%' WITH GRANT OPTION;