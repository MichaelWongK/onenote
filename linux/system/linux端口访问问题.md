# CentOS7 防火墙关闭但是远程端口无法访问问题

今天发现对于一台新使用的安装了CentOS7.3 系统服务器，即使关闭防火墙，除了22端口，其他端口只能在本地访问，无法被外部访问，找了诸多解决办法，最后找到了通过firewall可以启用其他端口的远程访问权限。

* 先开启firewalld

  ```
  systemctl start firewalld
  ```

* 添加80端口的访问权限，这里添加后永久生效

  ```
  firewall-cmd --zone=public --add-port=80/tcp --permanent
  ```

* 重新加载配置文件

  ```
  firewall-cmd --reload
  ```

> 此时测试，端口已经能够访问了，如果不需要firewall，可以再关闭，已放通端口不受影响

* 查看80端口访问权限情况

  ```
  firewall-cmd --zone= public --query-port=80/tcp
  ```

* 关闭80访问权限

  ```
  firewall-cmd --zone= public --remove-port=80/tcp --permanent
  ```



注：firewall依赖本机python版本，如果自己升级了python版本，需要修改firewall配置文件（实际版本号以本机实际为准，我的为2.7）：
1、vim /usr/bin/firewall-cmd， 将#！/usr/bin/python -Es 改为 #！/usr/bin/python2.7 -Es
2、vim /usr/sbin/firewalld, 将#！/usr/bin/python -Es 改为 #！/usr/bin/python2.7 -Es



### 另附firewall的其他命令操作

```
启动： systemctl start firewalld
查看状态： systemctl status firewalld 
停止： systemctl disable firewalld
禁用： systemctl stop firewalld
启动服务：systemctl start firewalld.service
关闭服务：systemctl stop firewalld.service
重启服务：systemctl restart firewalld.service
服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed
查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
更新防火墙规则： firewall-cmd --reload
查看区域信息:  firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic
```

