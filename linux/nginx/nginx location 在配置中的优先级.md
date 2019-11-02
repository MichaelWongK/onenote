### location表达式类型

- ~ 表示执行一个正则匹配，区分大小写
- ~* 表示执行一个正则匹配，不区分大小写
- ^~ 表示普通字符匹配。使用前缀匹配。如果匹配成功，则不再匹配其他location。
- = 进行普通字符精确匹配。也就是完全匹配。
- @ 它定义一个命名的 location，使用在内部定向时，例如 error_page, try_files







### location优先级说明

在nginx的location和配置中location的顺序没有太大关系。正location表达式的类型有关。相同类型的表达式，字符串长的会优先匹配。

以下是按优先级排列说明：

1. 等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。
2. ^~类型表达式。一旦匹配成功，则不再查找其他匹配项。
3. 正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。
4. 常规字符串匹配类型。按前缀匹配。



### location优先级示例

配置项如下:

```

```

1. `location = / {`
2. `# 仅仅匹配请求 /`
3. `[ configuration A ]`
4. `}`
5. `location / {`
6. `# 匹配所有以 / 开头的请求。`
7. `# 但是如果有更长的同类型的表达式，则选择更长的表达式。`
8. `# 如果有正则表达式可以匹配，则优先匹配正则表达式。`
9. `[ configuration B ]`
10. `}`
11. `location /documents/ {`
12. `# 匹配所有以 /documents/ 开头的请求。`
13. `# 但是如果有更长的同类型的表达式，则选择更长的表达式。`
14. `# 如果有正则表达式可以匹配，则优先匹配正则表达式。`
15. `[ configuration C ]`
16. `}`
17. `location ^~ /images/ {`
18. `# 匹配所有以 /images/ 开头的表达式，如果匹配成功，则停止匹配查找。`
19. `# 所以，即便有符合的正则表达式location，也不会被使用`
20. `[ configuration D ]`
21. `}`
22. `location ~* \.(gif|jpg|jpeg)$ {`
23. `# 匹配所有以 gif jpg jpeg结尾的请求。`
24. `# 但是 以 /images/开头的请求，将使用 Configuration D`
25. `[ configuration E ]`
26. `}`

请求匹配示例

```

```

1. `/ -> configuration A`
2. `/index.html -> configuration B`
3. `/documents/document.html -> configuration C`
4. `/images/1.gif -> configuration D`
5. `/documents/1.jpg -> configuration E`

注意，以上的匹配和在配置文件中定义的顺序无关。



原文：http://seanlook.com/2015/05/17/nginx-location-rewrite/作者： sean



## 1. location正则写法



一个示例：

```

```

1. `location = / {`
2. `# 精确匹配 / ，主机名后面不能带任何字符串`
3. `[ configuration A ]`
4. `}`
5. `location / {`
6. `# 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求`
7. `# 但是正则和最长字符串会优先匹配`
8. `[ configuration B ]`
9. `}`
10. `location /documents/ {`
11. `# 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索`
12. `# 只有后面的正则表达式没有匹配到时，这一条才会采用这一条`
13. `[ configuration C ]`
14. `}`
15. `location ~ /documents/Abc {`
16. `# 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索`
17. `# 只有后面的正则表达式没有匹配到时，这一条才会采用这一条`
18. `[ configuration CC ]`
19. `}`
20. `location ^~ /images/ {`
21. `# 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。`
22. `[ configuration D ]`
23. `}`
24. `location ~* \.(gif|jpg|jpeg)$ {`
25. `# 匹配所有以 gif,jpg或jpeg 结尾的请求`
26. `# 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则`
27. `[ configuration E ]`
28. `}`
29. `location /images/ {`
30. `# 字符匹配到 /images/，继续往下，会发现 ^~ 存在`
31. `[ configuration F ]`
32. `}`
33. `location /images/abc {`
34. `# 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在`
35. `# F与G的放置顺序是没有关系的`
36. `[ configuration G ]`
37. `}`
38. `location ~ /images/abc/ {`
39. `# 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用`
40. `[ configuration H ]`
41. `}`
42. `location ~* /js/.*/\.js`

- 已`=`开头表示精确匹配
  如 A 中只匹配根目录结尾的请求，后面不能带任何字符串。
- `^~` 开头表示uri以某个常规字符串开头，不是正则匹配
- ~ 开头表示区分大小写的正则匹配;
- ~* 开头表示不区分大小写的正则匹配
- / 通用匹配, 如果没有其它匹配,任何请求都会匹配到

顺序不等于优先级：

> (location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)

上面的匹配结果。按照上面的location写法，以下的匹配示例成立：

- / -> config A
  精确完全匹配，即使/index.html也匹配不了
- /downloads/download.html -> config B
  匹配B以后，往下没有任何匹配，采用B
- /images/1.gif -> configuration D
  匹配到F，往下匹配到D，停止往下
- /images/abc/def -> config D
  最长匹配到G，往下匹配D，停止往下
  你可以看到 任何以/images/开头的都会匹配到D并停止，FG写在这里是没有任何意义的，H是永远轮不到的，这里只是为了说明匹配顺序
- /documents/document.html -> config C
  匹配到C，往下没有任何匹配，采用C
- /documents/1.jpg -> configuration E
  匹配到C，往下正则匹配到E
- /documents/Abc.jpg -> config CC
  最长匹配到C，往下正则顺序匹配到CC，不会往下到E



### 实际使用建议

所以实际使用中，个人觉得至少有三个匹配规则定义，如下：

```

```

1. `#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。`
2. `#这里是直接转发给后端应用服务器了，也可以是一个静态首页`
3. `# 第一个必选规则`
4. `location = / {`
5. `proxy_pass http://tomcat:8080/index`
6. `}`
7. `# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项`
8. `# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用`
9. `location ^~ /static/ {`
10. `root /webroot/static/;`
11. `}`
12. `location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {`
13. `root /webroot/res/;`
14. `}`
15. `# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器`
16. `# 非静态文件请求就默认是动态请求，自己根据实际把握`
17. `# 毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了`
18. `location / {`
19. `proxy_pass http://tomcat:8080/`
20. `}`

参考：

- http://tengine.taobao.org/book/chapter_02.html
- http://nginx.org/en/docs/http/ngx_http_rewrite_module.html



## 2. Rewrite规则

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如`http://seanlook.com/a/we/index.php?id=1&u=str` 只对`/a/we/index.php`重写。语法`rewrite regex replacement [flag];`

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。

表明看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里，它们的执行顺序是：

1. 执行server块的rewrite指令
2. 执行location匹配
3. 执行选定的location中的rewrite指令

如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误。



### 2.1 flag标志位

- `last` : 相当于Apache的[L]标记，表示完成rewrite
- `break` : 停止执行当前虚拟主机的后续rewrite指令集
- `redirect` : 返回302临时重定向，地址栏会显示跳转后的地址
- `permanent` : 返回301永久重定向，地址栏会显示跳转后的地址

因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。这里 last 和 break 区别有点难以理解：

1. last一般写在server和if中，而break一般使用在location中
2. last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
3. break和last都能组织继续执行后面的rewrite指令



### 2.2 if指令与全局变量



#### if判断指令

语法为`if(condition){...}`，对给定的条件condition进行判断。如果为真，大括号内的rewrite指令将被执行，if条件(conditon)可以是如下任何内容：

- 当表达式只是一个变量时，如果值为空或任何以0开头的字符串都会当做false
- 直接比较变量和内容时，使用`=`或`!=`
- `~`正则表达式匹配，`~*`不区分大小写的匹配，`!~`区分大小写的不匹配

`-f`和`!-f`用来判断是否存在文件
`-d`和`!-d`用来判断是否存在目录
`-e`和`!-e`用来判断是否存在文件或目录
`-x`和`!-x`用来判断文件是否可执行

例如：

```

```

1. `if ($http_user_agent ~ MSIE) {`
2. `rewrite ^(.*)$ /msie/$1 break;`
3. `} //如果UA包含"MSIE"，rewrite请求到/msid/目录下`
4. `if ($http_cookie ~* "id=([^;]+)(?:;|$)") {`
5. `set $id $1;`
6. `} //如果cookie匹配正则，设置变量$id等于正则引用部分`
7. `if ($request_method = POST) {`
8. `return 405;`
9. `} //如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302`
10. `if ($slow) {`
11. `limit_rate 10k;`
12. `} //限速，$slow可以通过 set 指令设置`
13. `if (!-f $request_filename){`
14. `break;`
15. `proxy_pass http://127.0.0.1;`
16. `} //如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查`
17. `if ($args ~ post=140){`
18. `rewrite ^ http://example.com/ permanent;`
19. `} //如果query string中包含"post=140"，永久重定向到example.com`
20. `location ~* \.(gif|jpg|png|swf|flv)$ {`
21. `valid_referers none blocked www.jefflei.com www.leizhenfang.com;`
22. `if ($invalid_referer) {`
23. `return 404;`
24. `} //防盗链`
25. `}`



#### 全局变量

下面是可以用作if判断的全局变量

- `$args` ： #这个变量等于请求行中的参数，同`$query_string`
- `$content_length` ： 请求头中的Content-length字段。
- `$content_type` ： 请求头中的Content-Type字段。
- `$document_root` ： 当前请求在root指令中指定的值。
- `$host` ： 请求主机头字段，否则为服务器名称。
- `$http_user_agent` ： 客户端agent信息
- `$http_cookie` ： 客户端cookie信息
- `$limit_rate` ： 这个变量可以限制连接速率。
- `$request_method` ： 客户端请求的动作，通常为GET或POST。
- `$remote_addr` ： 客户端的IP地址。
- `$remote_port` ： 客户端的端口。
- `$remote_user` ： 已经经过Auth Basic Module验证的用户名。
- `$request_filename` ： 当前请求的文件路径，由root或alias指令与URI请求生成。
- `$scheme` ： HTTP方法（如http，https）。
- `$server_protocol` ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
- `$server_addr` ： 服务器地址，在完成一次系统调用后可以确定这个值。
- `$server_name` ： 服务器名称。
- `$server_port` ： 请求到达服务器的端口号。
- `$request_uri` ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
- `$uri` ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
- `$document_uri` ： 与$uri相同。

例：`http://localhost:88/test1/test2/test.php`

```

```

1. `$host：localhost`
2. `$server_port：88`
3. `$request_uri：http://localhost:88/test1/test2/test.php`
4. `$document_uri：/test1/test2/test.php`
5. `$document_root：/var/www/html`
6. `$request_filename：/var/www/html/test1/test2/test.php`



### 2.3 常用正则

- `.` ： 匹配除换行符以外的任意字符
- `?` ： 重复0次或1次
- `+` ： 重复1次或更多次
- `*` ： 重复0次或更多次
- `\d` ：匹配数字
- `^` ： 匹配字符串的开始
- `$` ： 匹配字符串的介绍
- `{n}` ： 重复n次
- `{n,}` ： 重复n次或更多次
- `[c]` ： 匹配单个字符c
- `[a-z]` ： 匹配a-z小写字母的任意一个

小括号`()`之间匹配的内容，可以在后面通过`$1`来引用，`$2`表示的是前面第二个`()`里的内容。正则里面容易让人困惑的是`\`转义特殊字符。



### 2.4 rewrite实例



#### 例1：

```

```

1. `http {`
2. `# 定义image日志格式`
3. `log_format imagelog '[$time_local] ' $image_file ' ' $image_type ' ' $body_bytes_sent ' ' $status;`
4. `# 开启重写日志`
5. `rewrite_log on;`
6. ``
7. `server {`
8. `root /home/www;`
9. ``
10. `location / {`
11. `# 重写规则信息`
12. `error_log logs/rewrite.log notice;`
13. `# 注意这里要用‘’单引号引起来，避免{}`
14. `rewrite '^/images/([a-z]{2})/([a-z0-9]{5})/(.*)\.(png|jpg|gif)$' /data?file=$3.$4;`
15. `# 注意不能在上面这条规则后面加上“last”参数，否则下面的set指令不会执行`
16. `set $image_file $3;`
17. `set $image_type $4;`
18. `}`
19. ``
20. `location /data {`
21. `# 指定针对图片的日志格式，来分析图片类型和大小`
22. `access_log logs/images.log mian;`
23. `root /data/images;`
24. `# 应用前面定义的变量。判断首先文件在不在，不在再判断目录在不在，如果还不在就跳转到最后一个url里`
25. `try_files /$arg_file /image404.html;`
26. `}`
27. `location = /image404.html {`
28. `# 图片不存在返回特定的信息`
29. `return 404 "image not found\n";`
30. `}`
31. `}`

对形如`/images/ef/uh7b3/test.png`的请求，重写到`/data?file=test.png`，于是匹配到`location /data`，先看`/data/images/test.png`文件存不存在，如果存在则正常响应，如果不存在则重写tryfiles到新的image404 location，直接返回404状态码。



#### 例2：

```

```

1. `rewrite ^/images/(.*)_(\d+)x(\d+)\.(png|jpg|gif)$ /resizer/$1.$4?width=$2&height=$3? last;`

对形如`/images/bla_500x400.jpg`的文件请求，重写到`/resizer/bla.jpg?width=500&height=400`地址，并会继续尝试匹配location。