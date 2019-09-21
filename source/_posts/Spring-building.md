---
title: Spring源码分析-环境搭建
date: 2019-07-14 13:15:32
tags: SpringSourceCode
categories: Spring
---

<!--more-->

## Gradle安装

先决条件：已安装Java JDK orJRE且版本为7或者以上，可通过命令$:java -version查看JDK版本。

笔者平台为Ubuntu，Windows的用户[点这里](https://jingyan.baidu.com/article/4d58d541167bc69dd4e9c009.html)

### 下载
下载地址：[点这里](https://gradle.org/releases)
这里我选择的是binary-only

### 解压

```
$ sudo mkdir /opt/gradle
$ sudo unzip -d /opt/gradle  gradle-5.5.1-bin.zip
```

### 配置环境变量

```
$ sudo vim /etc/profile  #打开配置文件
$ export GRADLE_HOME=/opt/gradle/gradle-5.5.1  #添加的内容
$ export PATH=$GRADLE_HOME/bin:$PATH   #添加的内容
```

保存完成后运行命令source /etc/profile 使环境变量生效
验证

```
$ gradle -v
```
![](https://s2.ax1x.com/2019/07/14/Z50fFH.png)

## Spring源码导入eclipse
### Spring源码下载
[github](https://s2.ax1x.com/2019/07/14/Z50fFH.png)
解压源码包

### 使用gradle将项目结构转变为eclipse工程结构
进入解压包目录下，其中就有官方提供的导入手册import-into-eclipse.md
接下来我们按照手册来导入
#### 预构
在解压包目录下，终端执行命令

```
$ ./gradlew :spring-oxm:compileTestJava
```
ps.Windows的朋友在dos中执行
另外项目明确说了 Eclipse需要安装 AspectJ (AspectJ Development Tools) 和 Groovy 两个插件 不然项目可能会报一些错误，BuildShip插件也需要，但是eclipse4版本好像内置此插件
#### 导入eclipse
使用eclipse 将整个文件导入 File -> Import -> Existing Gradle Project -> 找到源码目录 点击finish 开始导入

导入成功后你会发现项目编译错误
![](https://s2.ax1x.com/2019/07/14/Z5s9Xt.png)
打开Build path Libraries中缺少jar包，一般缺少两个，spring-cglib-repack-3.2.2.jar和
spring-objenesis-repack-2.4.jar两个包
**解决方法：**
在工程目录下（本文工程目录是直接导入spring源码解压包，且已转变eclipse工程结构）运行以下命令：

```
$ gradle objenesisRepackJar
$ gradle cglibRepackJar
```
![](https://s2.ax1x.com/2019/07/14/Z54xyD.png)
在refresh一下工程项目
![](https://s2.ax1x.com/2019/07/14/Z54KG6.png)
可以看到这两个包已经导入

另外，部分模块会出现com.sum....包无法导入，这是由于Eclipse的JRE System library中默认包含了一系列的代码访问规则，如果代码中引用了这些访问规则所禁止引用的类，就会报错，如引用了rt.jar包，可以通过添加一条允许访问JAR包中所有类的访问规则来解决

进入项目build path
![](https://s2.ax1x.com/2019/07/14/ZIBnMt.png)
![](https://s2.ax1x.com/2019/07/14/ZIBusP.png)

**CoroutinesUtils 报错，找不到该类。 解决办法：**
![](https://s2.ax1x.com/2019/07/14/ZIBldS.png)
直接找到spring-framework-master\spring-core-coroutines\build\libs 下面的spring-core-coroutines-5.2.0.BUILD-SNAPSHOT.jar包，将这个jar包导入依赖
报错项目右键 properties -> java build path -> add jars -> 找到spring-core-coroutines-5.2.0.BUILD-SNAPSHOT.jar 位置，选择确定， project clean 一下，这个CoroutinesUtils not found 的问题就解决了
![](https://s2.ax1x.com/2019/07/14/ZIBGGj.png)
![](https://s2.ax1x.com/2019/07/14/ZIBJRs.png)




