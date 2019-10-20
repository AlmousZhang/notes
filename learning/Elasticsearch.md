## elasticsearch安装

    安装教程：https://www.cnblogs.com/jianxuanbing/p/9410800.html（貌似不行）	
	拉取镜像：docker pull docker.elastic.co/elasticsearch/elasticsearch:6.3.2
	启动容器并添加映射：docker run -d --name es -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m"  -e "xpack.security.enabled=false" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2
	
	进入容器：docker exec -it es /bin/bash 运行失败的原因？
	解决方法:{
		sudo vi /etc/sysctl.conf
		vm.max_map_count=262144
		sudo sysctl -p
	}
	
	docker run -p 9301:9200 -e ES_JAVA_OPTS="-Xms256m -Xmx256m"  -e "xpack.security.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:5.6.2
	
- ##liunx安装es##

下载安装：
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.zip
unzip elasticsearch-5.5.1.zip
cd elasticsearch-5.5.1/ 
```
启动：
```
./bin/elasticsearch
```
错误：
```
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x0000000085330000, 2060255232, 0) failed; error='Cannot allocate memory' (errno=12)
```
解决：
配置elasticsearch下的jvm.options：
```
 vi /etc/elasticsearch/jvm.options 
	-Xms4g                    ##启用如下两项
	-Xmx4g
##-Xms2g                     ##关闭如下两项
##-Xmx2g
```
错误：
```
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
```
解决：
```
adduser esUser ;passwd esUser 密码：esUser
chmod 777 esUser
chown -R esUser /opt/elasticsearch-5.5.1
```
重新启动:
```
./bin/elasticsearch
```
测试：
```
curl -X GET http://localhost:9200
curl -i -XGET 'localhost:9200/'
chown -R esUser /opt/elasticsearch-5.5.1
```

安装分词插件
下载安装：
```
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.4.0/elasticsearch-analysis-ik-5.4.0.zip
unzip elasticsearch-analysis-ik-5.4.0.zip
mv elasticsearch ik
```
重启es服务器


sudo chown -R esUser /opt/elasticsearch-5.5.1/plugins/ik

远程访问：curl -X GET http://120.27.224.128:9200

## elasticsearch概念
Elasticsearch 是一个分布式可扩展的实时搜索和分析引擎,一个建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎.当然 Elasticsearch 并不仅仅是 Lucene 那么简单，它不仅包括了全文搜索功能，还可以进行以下工作:

- 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
- 实时分析的分布式搜索引擎。
- 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

1. Node 与 Cluster
	Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。

2. Index
	Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。所以，Elastic 数据管理的顶层单位就叫做 Index（索引）。它是单个数据库的同义词。每个 Index （即数据库）的名字必须是小写。
3. Type
	Document的分组

关系数据库     =》 数据库      =》表   		   =》 行              =》列(Columns)

Elasticsearch  =》索引(Index)  =》类型(type)   =》文档(Docments)   =》字段(Fields)  

原理：

1. 为什么搜索是 近 实时的？
2. 为什么文档的 CRUD (创建-读取-更新-删除) 操作是 实时 的?
3. Elasticsearch 是怎样保证更新被持久化在断电时也不丢失数据?
4. 为什么删除文档不会立刻释放空间？
5. refresh, flush, 和 optimize API 都做了什么, 你什么情况下应该使用他们？

笼统的来说，b-tree 索引是为写入优化的索引结构。当我们不需要支持快速的更新的时候，可以用预先排序等方式换取更小的存储空间，更快的检索速度等好处，其代价就是更新慢。
原理参考：
[elasticsearch原理]{:(https://www.cnblogs.com/dreamroute/p/8484457.html)
[时间序列数据库的秘密 ](https://www.infoq.cn/article/database-timestamp-02/?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk
)

**为什么 Elasticsearch/Lucene 检索可以比 mysql 快?**
	Mysql 只有 term dictionary 这一层，是以 b-tree 排序的方式存储在磁盘上的。检索一个term
需要若干次的 random access 的磁盘操作。而 Lucene 在 term dictionary 的基础上添加了 term index
来加速检索，term index 以树的形式缓存在内存中。从 term index 查到对应的 term dictionary 的 
block 位置之后，再去磁盘上找 term，大大减少了磁盘的 random access 次数。
erm index 在内存中是以 FST（finite state transducers）的形式保存的，其特点是非常节省内存。
Term dictionary 在磁盘上是以分 block 的方式保存的，一个 block 内部利用公共前缀压缩，比如
都是 Ab 开头的单词就可以把 Ab 省去。这样 term dictionary 可以比 b-tree 更节约磁盘空间。

