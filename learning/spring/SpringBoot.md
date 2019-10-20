#  **Spring Boot学习**

## 自动配置的原理

	- Spring Boot 在启动时扫描项目所依赖的 jar 包，寻找包含spring.factories 文件的 jar 包。
	- 根据 spring.factories 配置加载 AutoConfigure 类。
	- 根据 @Conditional 等条件注解 的条件，进行自动配置并将 Bean 注入 Spring IoC 中。

Spring Boot的启动：
第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器
第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块
第三部分是自动化配置模块，该模块作为springboot自动配置核心
@SpringBootApplication注解：
-- @SpringBootConfiguration：SpringBoot根据应用所声明的依赖来对Spring框架进行自动配置
-- @ComponentScan：组件扫描，可自动发现和装配Bean，默认扫描SpringApplication的run方法里的启动方法
所在的包路径下文件，所以最好将该启动类放到根包路径下
-- @EnableAutoConfiguration：被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有
bean事务，提供了一个spring的上下文环境
@EnableAutoConfiguration -> @Import(AutoConfigurationImportSelector.class) ->org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#selectImports
-> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getCandidateConfigurations

参考：

[1.springBoot2启动](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486739&idx=1&sn=174827f0ff5fb6c999e65ee80266bc39&chksm=eb538825dc240133c6eae1c051c2902786fdee84fcebf1df35eaf72809286088daebb3b977fd&scene=21#wechat_redirect)

[2.启动原理解析-平凡希](https://www.cnblogs.com/xiaoxi/p/7999885.html)


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

 


## **SpringBoot 面试题**

1. Spring Boot 是什么？Spring Boot、Spring MVC 和 Spring 有什么区别？
Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手

2. Spring Boot 提供了哪些核心功能？
application配置文件是应用级别的，是当前应用的配置文件。
bootstrap配置文件是系统级别的，用来加载外部配置，如配置中心的配置信息，也可以用来定义系统不会变化的属性。bootstatp文件的加载先于application文件。

3. Spring Boot 有什么优缺点？

4. Spring Boot 中的 Starter 是什么？
Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成 Spring 及其他技术，而不需要到处找示例代码和依赖包。Starters包含了许多项目中需要用到的依赖，它们能快速持续的运行，都是一系列得到支持的管理传递性依赖。

5. Spring Boot 配置加载顺序？

6. @SpringBootApplication核心注解
启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：
@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。
@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，
如关闭数据源自动配置功能：@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
@ComponentScan：Spring组件扫描。

7. 什么是 Spring Boot 自动配置？

8. Spring Boot 支持哪些日志框架？

9. 开启 Spring Boot 特性有哪几种方式？
	继承spring-boot-starter-parent项目
	导入spring-boot-dependencies项目依赖

**面试题**

## *定义*
	Spring Boot:Spring的子项目，提供Spring的引导功能。通过 Spring Boot ，我们开发者可以快速配置 Spring 项目，
	引入各种 Spring MVC、Spring Transaction、Spring AOP、MyBatis 等等框架，而无需不断重复编写繁重的 Spring 配置，降低了 Spring 的使用成本。
Spring Boot 提供了各种 Starter 启动器，提供标准化的默认配置

## *优势*
	pring Boot 的缺点主要是，因为自动配置 Spring Bean 的功能，我们可能无法知道，哪些 Bean 被进行创建了。
	这个时候，如果我们想要自定义一些 Bean ，可能存在冲突，或者不知道实际注入的情况

## *Spring的热启动*
	spring-boot-devtools

## *Spring Boot的配置加载顺序*	

	1. spring-boot-devtools 依赖的 spring-boot-devtools.properties 配置文件。
	2. 单元测试上的 @TestPropertySource 和 @SpringBootTest 注解指定的参数。
	3. 命令行指定的参数。例如 java -jar springboot.jar --server.port=9090 。
	4. 命令行中的 spring.application.json 指定参数。例如 java -Dspring.application.json='{"name":"Java"}' -jar springboot.jar 。
	5. ServletConfig 初始化参数。
	6. ServletContext 初始化参数。
	7. JNDI 参数。例如 java:comp/env 。
	8. Java 系统变量，即 System#getProperties() 方法对应的。
	9. 操作系统环境变量。
	10. RandomValuePropertySource 配置的 random.* 属性对应的值。
	11. Jar 外部的带指定 profile 的 application 配置文件。例如 application-{profile}.yaml 。
	12. Jar 内部的带指定 profile 的 application 配置文件。例如 application-{profile}.yaml 。
	13. Jar 外部 application 配置文件。例如 application.yaml 。
	14. Jar 内部 application 配置文件。例如 application.yaml 。
	15. 在自定义的 @Configuration 类中定于的 @PropertySource 。
	16. 启动的 main 方法中，定义的默认配置。即通过 SpringApplication#setDefaultProperties(Map<String, Object> defaultProperties) 方法进行设置。

## *Spring Boot核心注解*	
	@SpringBootApplication
		@SpringBootConfiguration
		@EnableAutoConfiguration
		@ComponentScan
		参考：
		https://blog.csdn.net/zhou920786312/article/details/84326023
		https://blog.csdn.net/claram/article/details/75125749
