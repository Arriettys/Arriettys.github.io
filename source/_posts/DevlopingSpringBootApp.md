---
title: SpringBoot：开始你的第一个SpringBoot应用
date: 2019-03-19 22:49:26
tags: SpringBoot
categories: Spring
---
![](https://s2.ax1x.com/2019/03/22/A3f1u4.jpg)
<!--more-->
## 依赖
在开始之前，要确定你的电脑是否安装Java和maven（若是使用IDE如eclipse，可直接使用IDE自带的）。安装这里不叙述，终端输入一下命令验证：
```
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```
```
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```
ps.maven由于GWF缘故连接外网下载jar包会很慢，建议将源地址替换为阿里镜像
找到找到 apache-maven-3.5.2\conf 目录中的 **settings.xml** 文件找到**<mirrors>  </ mirrors>**标签，在标签内添加内容如下：
```
<mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>

```
## 纯命令行式
最原始的方法
1.创建一份pom.xml文件
```
$ vim pom.xml
```
终端输入以下（默认继承parent starter构件）：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
	</parent>

	<!-- Additional lines to be added here... -->

</project>
```
个人建议保存一份模板，每次创建时cp就行
可以终端运行（但其实没必要，因为你还没开始编辑你的Java代码）
```
$ mvn package
```
就会自动导入依赖包（导入过程你可以无视“jar will be empty - no content was marked for inclusion!” 警告）。导入成功
![](https://s2.ax1x.com/2019/03/20/Auzahd.png)
并自动生成maven和target文件夹

2.添加类路径依赖
```
$ mvn dependency:tree
```
该命令打印树型结构工程依赖
然后我们在pom.xml中添加
```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```
再使用mvn denpendency:tree，你可以看到跟多的依赖，如tomcat web server

3.写代码
接下来我们创建Java文件
```
$ mkdir -p src/main/java
$ vim src/main/java/Example.java
```
```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example
{

	@RequestMapping("/")
	String home() 
	{
		return "Hello World!";
	}

	public static void main(String[] args) 
	{
		SpringApplication.run(Example.class, args);
	}

}
```
4.运行Example
终端使用：
```
mvn spring-boot:run
```
在浏览器访问 localhost:8080，可以看到 Hello World！
按Ctrl+c退出
5.创建可执行jar
包含所有依赖和你编辑的类，相当于一个硬盘免安装软件。我们需要先在pom.xml中添加**spring-boot-maven-plugin**
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```
终端使用
```
$ mvn package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.1.3.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
target文件夹中会出现**myproject-0.0.1-SNAPSHOT.jar**和**myproject-0.0.1-SNAPSHOT.jar.original**（它是Maven在SpringBoot重新打包之前创建的原始JAR文件）
现在我们运行这个应用
```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.1.3.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```
像之前一样浏览器访问，退出Ctrl+C

## eclipse创建SpringBoot项目
**替换maven源**
我们先自己创建一份setting.xml:
```
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
	<mirrors>
		<!-- mirror | Specifies a repository mirror site to use instead of a given 
			repository. The repository that | this mirror serves has an ID that matches 
			the mirrorOf element of this mirror. IDs are used | for inheritance and direct 
			lookup purposes, and must be unique across the set of mirrors. | -->
		<mirror>
    	<id>huaweicloud</id>
    	<mirrorOf>*</mirrorOf>
    	<url>https://mirrors.huaweicloud.com/repository/maven/</url>
		</mirror>
		<mirror>
     	<id>mirrorId</id>
     	<mirrorOf>central</mirrorOf>
     	<name>Human Readable Name for this Mirror.</name>
     	<url>http://central.maven.org/maven2/</url>
		</mirror>
		<mirror>
     	<id>nexus-aliyun</id>
     	<mirrorOf>*</mirrorOf>
     	<name>Nexus aliyun</name>
     	<url>http://maven.aliyun.com/nexus/content/groups/public</url>
		</mirror>
	</mirrors>

	<profiles>
		<profile>
			<id>default</id>
			<repositories>
				<repository>
					<id>nexus</id>
					<name>local private nexus</name>
					<url>http://maven.oschina.net/content/groups/public/</url>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>false</enabled>
					</snapshots>
				</repository>
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>nexus</id>
					<name>local private nexus</name>
					<url>http://maven.oschina.net/content/groups/public/</url>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>false</enabled>
					</snapshots>
				</pluginRepository>
			</pluginRepositories>
		</profile>
	</profiles>
</settings>
```
再在eclipse中
Eclipse - > [ Windows ] - > [ Preferences ]- > [ Maven ] - > [ User Settings ]
替换为我们创建的setting.xml文件
之后重启eclipse即可
## 方法一 安装STS插件
![](https://s2.ax1x.com/2019/03/20/AK9CfH.png)
![](https://s2.ax1x.com/2019/03/20/AK9l1s.png)

安装插件导向窗口完成后，在eclipse右下角将会出现安装插件的进度，等插件安装完成后重启eclipse生效
新建springboot项目
![](https://s2.ax1x.com/2019/03/20/AK91cn.png)
![](https://s2.ax1x.com/2019/03/20/AK9Gn0.png)
项目启动
![](https://s2.ax1x.com/2019/03/20/AK9JBV.png)

## 方法二 
1.创建Maven项目
![](https://s2.ax1x.com/2019/03/20/AK9ah4.png)
2.选择项目类型
![](https://s2.ax1x.com/2019/03/20/AQvQfJ.png)
这里勾选了Create project...，也可以不够只是更方便，省去之后创建resources文件夹
3.编写项目组和名称即可，Packaging选jar或war都可以
![](https://s2.ax1x.com/2019/03/20/AlSN7V.png)
4.修改pom.xml文件
```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.2.RELEASE</version>
</parent>
```
ps.在导入**spring-boot-starter-parent**出现这种情况：
![](https://s2.ax1x.com/2019/03/20/Al9sdx.png)
可鼠标右键点击工程[Maven] -> [Update Project]
![](https://s2.ax1x.com/2019/03/20/Al9HFf.png)
取消Offline，勾选Force Update of Snapshots/Releases，Ok
编译插件
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

```
依赖，这里实验只用web
```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
</dependencies>
```
5.创建resources文件夹和application.properties文件
![](https://s2.ax1x.com/2019/03/25/AtL8SS.png)
6.写代码
![](https://s2.ax1x.com/2019/03/20/AlCpT0.png)
在src/main/java处创建Example.java，代码依旧是之前写的hello world
之后[Run As] -> [Java Aplication]运行，浏览器查看

