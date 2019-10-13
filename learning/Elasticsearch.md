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

远程访问：curl -X GET http://120.27.224.128:9200