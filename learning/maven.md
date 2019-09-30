## 打成 jar 包

- cd 项目跟目录（和pom.xml同级）

	`mvn clean package`
	
	排除测试代码后进行打包
	
	`mvn clean package  -Dmaven.test.skip=true`
	
	打包完成后 jar 包会生成到 target 目录下，命名一般是 项目名+版本号.jar

- 启动 jar 包命令
	
	`java -jar  target/spring-boot-scheduler-1.0.0.jar`
	
- 使用在后台运行的方式来启动:
	
	`nohup java -jar target/spring-boot-scheduler-1.0.0.jar &`
	
- 也可以在启动的时候选择读取不同的配置文件
	
	`java -jar app.jar --spring.profiles.active=dev`
	
- 也可以在启动的时候设置 jvm 参数
	
	`java -Xms10m -Xmx80m -jar app.jar &`
	
	
	
# jenkins部署

[jenkins安装使用](http://www.ityouknow.com/springboot/2017/11/11/spring-boot-jenkins.html)