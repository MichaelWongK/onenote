# Solr Study

## Solr的常见命令：

- 启动：

```
./solr start
windows 
```

- 关闭：

```
./solr stop -all
```

- 重启：

```
./solr restart
```

- 创建Core：

```
./solr create -c YourCoreName -d _default
```

- 删除Core：

```
./solr delete -c YourCoreName
```



## 相关链接：

- Solr 官网

[https://lucene.apache.org/solr/](https://links.jianshu.com/go?to=https%3A%2F%2Flucene.apache.org%2Fsolr%2F)

- Solr API官方文档

http://lucene.apache.org/solr/7_7_1/solr-solrj/index.html

- Solr 中文文档（译版）

https://www.w3cschool.cn/solr_doc/solr_doc-t3642fkr.html



## **创建Core**

Solr `Core`的创建有两种方式，第一种是通过命令行的方式，第二种是在Solr `Web` 首页中创建。

> 个人比较推荐第一种，原因有二，一是方便快捷在命令行一句话搞定，`Ctrl+c Ctrl+v`齐活儿，不用在页面上点来点去还得敲文字，二是所在公司大牛在实践过后说是如果采用的第二种方式创建出的`Core`会有一些问题。本着*听人话，吃饱饭*的态度就采用第一种吧~ ~*大家可以实践一下然后分享给我哦。*当然两种方式还是要介绍下的。

- 使用命令行创建`Core`

我们只需要进入到上述的`bin`目录，复制以下命令即可创建`Core`，命令中的`TestCore`为名称可以自行替换：

```
./solr create -c TestCore -d _default
```

 

## **配置IK-Analyzer中文分词**

介绍性的内容还是不罗列了，大家自己百度吧。Ik-Analyzer分词据说是国内最好用的中文分词，大部分人都用这个。目前Solr-7.7.1也自带了一个中文分词，具体的对比我没做过，等装完IK后大家回来可以进行下对比。*同时欢迎分享给我哦。*

- IK分词GitHub：

[https://github.com/magese/ik-analyzer-solr7](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmagese%2Fik-analyzer-solr7)

- 动态词库自动加载：

[https://github.com/liang68/ik-analyzer-solr6](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fliang68%2Fik-analyzer-solr6)

[http://www.cnblogs.com/liang1101/articles/6395016.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2Fliang1101%2Farticles%2F6395016.html)













