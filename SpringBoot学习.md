#  **Spring Boot学习**

###  **配置java环境变量:**
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-1.el7_7.x86_64 
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile
```
###  **配置maven环境变量:**
	http://mirrors.shu.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
	
#### 解压
`tar vxf apache-maven-3.5.2-bin.tar.gz`

#### 移动
```
mv apache-maven-3.5.2 /usr/local/maven3
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

##	**SpringBoot部署docker**

###	**DockerFile构建镜**

在 pom.xml-properties 中添加 Docker 镜像名称
```
<properties>
	<docker.image.prefix>springboot</docker.image.prefix>
</properties>
```
lugins 中添加 Docker 构建插件：
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
		<!-- Docker maven plugin -->
		<plugin>
			<groupId>com.spotify</groupId>
			<artifactId>docker-maven-plugin</artifactId>
			<version>1.0.0</version>
			<configuration>
				<imageName>${docker.image.prefix}/${project.artifactId}</imageName>
				<dockerDirectory>src/main/docker</dockerDirectory>
				<resources>
					<resource>
						<targetPath>/</targetPath>
						<directory>${project.build.directory}</directory>
						<include>${project.build.finalName}.jar</include>
					</resource>
				</resources>
			</configuration>
		</plugin>
		<!-- Docker maven plugin -->
	</plugins>
</build>
```
在目录src/main/docker下创建 Dockerfile 文件，Dockerfile 文件用来说明如何来构建镜像。
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD spring-boot-docker-1.0.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
主要作用：构建 Jdk 基础环境，添加 Spring Boot Jar 到镜像中：
1. FROM ，表示使用 Jdk8 环境 为基础镜像，如果镜像不是本地的会从 DockerHub 进行下载
2. VOLUME ，VOLUME 指向了一个/tmp的目录，由于 Spring Boot 使用内置的Tomcat容器，Tomcat 默认使用/tmp作为工作目录。这个命令的效果是：在宿主机的/var/lib/docker目录下创建一个临时文件并把它链接到容器中的/tmp目录
3. ADD ，拷贝文件并且重命名
4. ENTRYPOINT ，为了缩短 Tomcat 的启动时间，添加java.security.egd的系统属性指向/dev/urandom作为 ENTRYPOINT
###	**DockerFile构建镜**
```
mvn package docker:build
```
###	**运行该镜像**
```
docker run -p 8080:8080 -t springboot/spring-boot-docker
```
###	**停止**
```
docker stop CONTAINERId
```
###	**docker-compose**
启动服务：docker-compose up
使用docker-compose ps查看项目中目前的所有容器
关闭服务docker-compose down
参考： http://www.ityouknow.com/springboot/2018/03/28/dockercompose-springboot-mysql-nginx.html
ocker-compose.yaml 文件详解
```
version: '3'
services:
  nginx:
   container_name: v-nginx
   image: nginx:1.13
   restart: always
   ports:
   - 80:80
   - 443:443
   volumes:
   - ./nginx/conf.d:/etc/nginx/conf.d
    
  mysql:
   container_name: v-mysql
   image: mysql/mysql-server:5.7
   environment:
    MYSQL_DATABASE: test
    MYSQL_ROOT_PASSWORD: root
    MYSQL_ROOT_HOST: '%'
   ports:
   - "3306:3306"
   restart: always
    
  app:
    restart: always
    build: ./app
    working_dir: /app
    volumes:
      - ./app:/app
      - ~/.m2:/root/.m2
    expose:
      - "8080"
    depends_on:
      - nginx
      - mysql
    command: mvn clean spring-boot:run -Dspring-boot.run.profiles=docker
```
### *加载资源*
CommandLineRunner 接口的 Component 会在所有 Spring Beans 都初始化之后，SpringApplication.run() 之前执行，非常适合在应用程序启动之初进行一些数据初始化的工作。

#### *响应式编程Webflux *
Java 注解编程模型
函数式编程模型

	https://www.ibm.com/developerworks/cn/java/spring5-webflux-reactive/index.html


curl的使用：


curl http://localhost:8080/sse/randomNumbers


git 使用：
 git fetch origin dev（dev为远程仓库的分支名）
 git checkout -b dev(本地分支名称) origin/dev(远程分支名称)
 git pull origin dev(远程分支名称)