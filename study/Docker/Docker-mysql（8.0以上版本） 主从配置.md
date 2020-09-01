# 1. docker下mysql（8.0以上版本） 主从配置

请自行先安装完docker和docker-compose
mysql8.0以后的版本与5.7的配置有一点点不同，网上不少都是5.X版本的配置方法，并不适用于8.0的，在这里总结一下我配置成功的过程。
 我的安装环境：

![img](https:////upload-images.jianshu.io/upload_images/20140387-4692f187fd35dd41.png?imageMogr2/auto-orient/strip|imageView2/2/w/339/format/webp)

在这里插入图片描述

1. 在主机上安装docker之后，从官网拉取最新镜像

```css
docker pull mysql:latest
```

安装的mysql最新版本为：

![img](https:////upload-images.jianshu.io/upload_images/20140387-dc2cae94adfcd803.png?imageMogr2/auto-orient/strip|imageView2/2/w/505/format/webp)

在这里插入图片描述

### 编写docker-compose.yml，

以及两个数据库配置文件master.my.cnf,  slave.my.cnf，这里先建个目录，在目录下创建需要的三个文件：

```
mkdir mysql_docker_file && cd mysql_docker_file
touch docker-compose.yml master.my.cnf slave.my.cnf
```

 docker-compose.yml内容如下：

 docker-compose.yml内容如下：

**主要将mysql-master和mysql-slave配置在172.18.2.0/24同一个网段，
 然后做了个存储空间的映射，将两个容器的数据内容都映射到宿主机的/root/data目录下**

```shell
version: '2'
services:
  mysql-master:
   image: mysql
   networks:
       mysql_net:
         ipv4_address: 172.17.2.2
   volumes:
     - /root/data/mysql-master:/var/lib/mysql
     - /data/docker/mysql-master-slave/master.my.cnf:/etc/mysql/my.cnf
   ports:
     - "33067:3306"
   environment:
     - MYSQL_DATABASE=root
     - MYSQL_ROOT_PASSWORD=123456
  mysql-slave:
   image: mysql
   networks:
       mysql_net:
         ipv4_address: 172.17.2.3
   volumes:
     - /root/data/mysql-slave:/var/lib/mysql
     - /data/docker/mysql-master-slave/slave.my.cnf:/etc/mysql/my.cnf
   ports:
     - "33068:3306"
   environment:
     - MYSQL_DATABASE=root
     - MYSQL_ROOT_PASSWORD=123456
networks:
  mysql_net:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.17.2.0/24
```

### 新建master.my.cnf，内容如下：

 只在默认的文件加了三条：
 server-id=111 #id号
 innodb_flush_log_at_trx_commit=2
 sync_binlog=1
 [innodb_flush_log_at_trx_commit和sync_binlog请参考点击这篇文章，讲得很清楚](https://www.jianshu.com/p/74b03a792ff8)

```shell
#copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
## 1.1. This program is free software; you can redistribute it and/or modify
## 1.2. it under the terms of the GNU General Public License as published by
## 1.3. the Free Software Foundation; version 2 of the License.
##
## 1.4. This program is distributed in the hope that it will be useful,
## 1.5. but WITHOUT ANY WARRANTY; without even the implied warranty of
## 1.6. MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
## 1.7. GNU General Public License for more details.
##
## 1.8. You should have received a copy of the GNU General Public License
## 1.9. along with this program; if not, write to the Free Software
## 1.10. Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
#
##
## 1.11. The MySQL  Server configuration file.
##
## 1.12. For explanations see
## 1.13. http://dev.mysql.com/doc/mysql/en/server-system-variables.html
#
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

lower_case_table_names=1

default_authentication_plugin=mysql_native_password

## 1.14. Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

#
## 1.15. Custom config should go here
!includedir /etc/mysql/conf.d/

server-id=111
innodb_flush_log_at_trx_commit=2  
sync_binlog=1  
```

新建slave.my.cnf，内容如下：
 在默认文件下添加了6条
 server-id=222
 innodb_flush_log_at_trx_commit=2
 sync_binlog=1
 relay_log_recovery=0

lower_case_table_names=1

default_authentication_plugin=mysql_native_password

```shell
#copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
## 1.16. This program is free software; you can redistribute it and/or modify
## 1.17. it under the terms of the GNU General Public License as published by
## 1.18. the Free Software Foundation; version 2 of the License.
##
## 1.19. This program is distributed in the hope that it will be useful,
## 1.20. but WITHOUT ANY WARRANTY; without even the implied warranty of
## 1.21. MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
## 1.22. GNU General Public License for more details.
##
## 1.23. You should have received a copy of the GNU General Public License
## 1.24. along with this program; if not, write to the Free Software
## 1.25. Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
#
##
## 1.26. The MySQL  Server configuration file.
##
## 1.27. For explanations see
## 1.28. http://dev.mysql.com/doc/mysql/en/server-system-variables.html
#
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
## 1.29. Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

lower_case_table_names=1

default_authentication_plugin=mysql_native_password

#
## 1.30. Custom config should go here
!includedir /etc/mysql/conf.d/
server-id=222
innodb_flush_log_at_trx_commit=2  #
sync_binlog=1  #开启binlog日志同步功能
relay_log_recovery=0
```

master.my.cnf和slave.my.cnf的server-id一定不能一样，而innodb_flush_log_at_trx_commit，sync_binlog，relay_log_recovery等参数的设置会影响到主从复制的性能，有兴趣的可以进行更多的了解。

1. 全部文件都写完后，终于可以创建容器了，执行：

   ```
   docker-compose up -d
   ```

   ![img](https:////upload-images.jianshu.io/upload_images/20140387-cf7fb24dc3d05c34.png?imageMogr2/auto-orient/strip|imageView2/2/w/506/format/webp)

   在这里插入图片描述

   执行

   ```
   docker ps -a
   ```

   查看是否创建容器成功，失败的话请docker logs 容器ID 查看日志并解决，第一列为容器ID

   ![img](https:////upload-images.jianshu.io/upload_images/20140387-9be2303ccb1d2546.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

   在这里插入图片描述

2. 主数据库操作流程如下：
    进入master容器：`docker exec -it 容器的ID号 /bin/bash`
    登录数据库:`mysql -uroot -p123456`
    创建账号：`create user 'slave'@'172.17.2.3' identified by '123456';`, 注意填从库的IP
    后授权：`grant replication slave on *.* to 'slave'@'172.18.2.3' with grant option;`
    刷新下权限：`flush privileges;`
    查看master状态：`show master status;`,记录这里的file和Position，后面要用
    锁库：`flush tables with read lock;`
    退出数据库：`exit`
    导出master整个数据库内容：`mysqldump -uroot -p123456 -A --events > backup.sql`
    将生成的backup.sql想办法弄到slave容器，这里通过宿主机做中介：`mv backup.sql /var/lib/mysql`
    退出容器：`exit`

3. 回到宿主机后，进入/root/data/mysql-master，将backup.sql移动到/root/data/mysql-slave

4. 从数据库操作流程如下：
    进入slave容器：`docker exec -it 容器的ID号 /bin/bash`
    导入master的数据库数据：`mysql < /var/lib/mysql/backup.sql -uroot -p123456`
    登录数据库:`mysql -uroot -p123456`
    可以看到有slave账号：`select user from mysql.user;`
    连接master的设置：注意填主库ip,上面记录的file和position



```shell
change master to \
master_host='172.17.2.2',
master_port=3306,
master_user='slave',
master_password='123456',
master_log_file='binlog.000003',
master_log_pos=892,
get_master_public_key=1;
```

开启从服务：`start slave;`
 执行`show slave status\G;`，看到后面两个yes就成立

![image-20200901220728071](D:\workspace\git\onenote\imageFiles\image-20200901220728071.png)



### 再次登录主库：

进入master容器：

```
docker exec -it 容器的ID号 /bin/bash
```

登录数据库:

```
mysql -uroot -p123456
```

解锁：

```
unlock tables；
```

到了这里就大功告成了！！！

### 测试一下：

在主库创建一个名为test的数据库

```
create database test;
```

去从库执行：

```
show databases;
```

可以看到也有也test库

![image-20200901220842020](C:\Users\wangm\AppData\Roaming\Typora\typora-user-images\image-20200901220842020.png)

