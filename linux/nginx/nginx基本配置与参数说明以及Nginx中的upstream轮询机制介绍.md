## 一.nginx简介

​        Nginx (发音为[engine x])专为性能优化而开发，其最知名的优点是它的稳定性和低系统资源消耗，以及对并发连接的高处理能力(单台物理服务器可支持30000～50000个并发连接)， 是一个高性能的 HTTP 和反向代理服务器，也是一个IMAP/POP3/SMTP 代理服。



## 二.nginx基本配置与参数说明

\#运行用户



```undefined
user nobody;
```

\#启动进程,通常设置成和cpu的数量相等



```undefined
worker_processes  1;
```

\#全局错误日志及PID文件

```cs
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
```

\#工作模式及连接数上限

events {



​    \#仅用于linux2.6以上内核,可以大大提高nginx的性能  



```php
use   epoll;
```

​    \#**单个后台worker process进程**最大并发链接数  



```undefined
worker_connections  1024;
```

​    \# 并发总数是 worker_processes 和 worker_connections 的乘积



​    \# 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么



​    \# 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000



​    \# 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数



​    \# 我们来看看360M内存的VPS可以打开的文件句柄数是多少：



​    \# 输出 34336



​    \# 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置



​    \# 其实质也就是根据主机的物理CPU和内存进行配置



​    \# ulimit -SHn 65535



\#设定http服务器



​    \#**设定mime类型,类型由mime.type文件定义**   



```php
include    mime.types;

default_type  application/octet-stream;
```

​    \#**设定日志格式**



```swift
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

access_log  logs/access.log  main;
```

​    \#sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，



​    \#如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，





```vbscript
sendfile     on;
```

​    \#tcp_nopush     on;

​    \#连接超时时间





```vbscript
keepalive_timeout  65;
tcp_nodelay     on;
 #`FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度`。下面参数看字面意思都能理解。
```

``

```undefined
fastcgi_connect_timeout 300;
fastcgi_send_timeout 300;
fastcgi_read_timeout 300;
fastcgi_buffer_size 64k;
fastcgi_buffers 4 64k;
fastcgi_busy_buffers_size 128k;
fastcgi_temp_file_write_size 128k;
```

​      \#upstream的负载均衡，（以权重方式分发），weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。



```vbscript
upstream blog.nginx.com {
    server 192.168.80.121:80 weight=3;
    server 192.168.80.122:80 weight=2;
    server 192.168.80.123:80 weight=3;
}
```

​      \#upstream的负载均衡，（以nginx热备方式分发），其它所有的非backup Server down或者忙的时候，请求backup机器。所以**这台机器压力会最轻**。

```vbscript
upstream blog.nginx.com {
    server 192.168.80.121:80;
    server 192.168.80.122:80;
    server 192.168.80.123:80 backup;
}<span style="font-size:18px;"></span>
```

​    \#开启gzip压缩   



```vbscript
gzip  on;
gzip_disable "MSIE [1-6].";
```

​    \#设定请求缓冲   



```undefined
client_header_buffer_size    128k;
large_client_header_buffers  4 128k; 
```

​    \#设定虚拟主机配置



​        \#**侦听80端口**    



```perl
listen    80;
```

​        \#定义访问地址，域名可以有多个，用空格隔开      



```css
server_name www.nginx.cn nginx.cn ;
```

​        \#定义服务器的默认网站根目录位置  



```undefined
root html;
```

​        \#设定本虚拟主机的访问日志    



```vbscript
access_log  logs/nginx.access.log  main; 
```

​        \#默认请求       



```perl
location / {            
   #定义首页索引文件的名称
   index index.php index.html index.htm;   
}
```

​        \#对 “/” 启用反向代理

```php
location / {
    proxy_pass http://127.0.0.1:88;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #以下是一些反向代理的配置，可选。
    proxy_set_header Host $host;
    client_max_body_size 10m; #允许客户端请求的最大单文件字节数
    client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
    proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
    proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
    proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
    proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
    proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
    proxy_temp_file_write_size 64k;
    #设定缓存文件夹大小，大于这个值，将从upstream服务器传
}
```

​       \#设定查看Nginx状态的地址



```cs
location /NginxStatus {
    stub_status on;
    access_log on;
    auth_basic “NginxStatus”;
    auth_basic_user_file conf/htpasswd;
    #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
}
```

​       \#本地动静分离反向代理配置

​        \#所有jsp的页面均交由tomcat或resin处理



```php
location ~ .(jsp|jspx|do)?$ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://127.0.0.1:8080;
}
```

​    \#所有静态文件由nginx直接读取不经过tomcat或resin



```ruby
location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt
                |pdf|xls|mp3|wma)$ { 
    expires 15d; 
}
location ~ .*.(js|css)?$ { 
    expires 1h; 
}
```

​        \# 定义错误提示页面      



```undefined
error_page   500 502 503 504 /50x.html;
location = /50x.html {
} 
```

​        \#静态文件缓存时间设置，nginx自己处理     



```php
location ~ ^/(images|javascript|js|css|flash|media|static)/ {            
   #过期30天，静态文件不怎么更新，过期可以设大一点，
   #如果频繁更新，则可以设置得小一点。
   expires 30d;
}
```

​        \#PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置. 



```ruby
location ~ .php$ {
   fastcgi_pass 127.0.0.1:9000;
   fastcgi_index index.php;
   fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
   include fastcgi_params;
}
```

​        \#禁止访问 .htxxx 文件     



```xml
location ~ /.ht {
   deny all;
}<span style="font-size:18px;">    </span>
```

​    }

**简单配置示例**

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
error_log  /app/nginx-web/logs/error.log  debug;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /app/nginx-web/logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    #keepalive_timeout  65;

    #gzip  on;

#include /app/nginx-web/conf/conf.d/*.conf;
    server {
	listen	9100;
	server_name	dountui-std;
	location /  {
		root    /app/nginx-web/html/dountui-std/;
		index  	index.html index.htm;
		#include conf.d/global_proxy_server.conf;
    		#expires -1;

	}
	error_page   500 502 503 504  /50x.html;
         location = /50x.html {
             root   html;
         }

	location /prod-api/{
                #proxy_pass http://47.100.93.46:8088/sweet-heart/;
                #include conf.d/global_proxy_server.conf;
		#expires -1;
        }

    }
}
```

补充：

```
1.使用include可以引用外部配置文件 
 /app/nginx-web/conf/conf.d/*.conf;
```



## 三.nginx配置超时时间

​      http://my.oschina.net/xsh1208/blog/199674 

 



Nginx中upstream有以下几种方式：





```vbscript
upstream bakend {
    server 192.168.1.10;
    server 192.168.1.11;
}
```

2、weight 
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
 如果后端服务器down掉，能自动剔除。 
 比如下面配置，则1.11服务器的访问量为1.10服务器的两倍。



```vbscript
upstream bakend {
    server 192.168.1.10 weight=1;
    server 192.168.1.11 weight=2;
}
```

3、ip_hash 
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session不能跨服务器的问题。 
如果后端服务器down掉，要手工down掉。



```vbscript
upstream resinserver{
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

4、fair（第三方插件） 
按后端服务器的响应时间来分配请求，响应时间短的优先分配。



```vbscript
upstream resinserver{
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    fair;
}
```

5、url_hash（第三方插件） 
 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存服务器时比较有效。 
 在upstream中加入hash语句，hash_method是使用的hash算法



```vbscript
upstream resinserver{
   server 192.168.1.10:8080;
   server 192.168.1.11:8080;
   hash $request_uri;
   hash_method crc32;
}
```

设备的状态有: 
1.down 表示单前的server暂时不参与负载 
2.weight 权重,默认为1。 weight越大，负载的权重就越大。 
3.max_fails 允许请求失败的次数默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误 
4.fail_timeout max_fails次失败后，暂停的时间。 
5.backup 备用服务器, 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

 





```perl
<span style="font-size:14px;">启动nginx:
nginx -c /path/to/nginx.conf

重启nginx：
nginx -s reload  ：修改配置后重新加载生效
nginx -s reopen  ：重新打开日志文件
nginx -t -c /path/to/nginx.conf 测试nginx配置文件是否正确

关闭nginx：
nginx -s stop  :快速停止nginx
         quit  ：完整有序的停止nginx

其他的停止nginx 方式：

ps -ef | grep nginx

kill -QUIT 主进程号     ：从容停止Nginx
kill -TERM 主进程号     ：快速停止Nginx
pkill -9 nginx          ：强制停止Nginx

平滑重启nginx：
kill -HUP 主进程号</span>
```