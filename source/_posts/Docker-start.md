---
title: Docker-Hello World
date: 2019-03-31 21:48:39
tags: Docker
categories: 虚拟化
---
![](https://s2.ax1x.com/2019/04/01/AsMalQ.jpg)
<!--more-->
## 要想入门Docker，首先你需要理解Docker！
Docker，可以说是一个终端命令行的虚拟机，但更准确的说法，其实应该是一个虚拟环境。比如，你想要在PC上无缝使用Linux么？那么虚拟机并不是你唯一的出路，你还有Docker！我更愿意称Docker为一个容器，当然这只是Docker的一个狭义解释，Docker不止是一个容器。Docker包含3个重要概念：

+ 一个，是镜像（Image），镜像是静态的、可以被用户互相分享的文件。我们玩过双系统和虚拟机的人都知道，首先你需要一个.iso镜像，才能安装系统。Docker中的镜像也是这个东西，镜像是静态的，你不能对他操作，只能pull别人的镜像或者push自己的镜像。
+ 还有一个，是容器（Container），前面说过，镜像是静态不可操作的，只能被分享和下载，那什么是能被操作的呢？就是容器里！容器可以理解为镜像的动态状态，也就是我们虚拟机中系统装好后的状态，其实这么说是不对的，容器最符合的描述应该是Linux的iso文件的Live CD模式，比如我们玩双系统时都进入过Live CD模式，不安装系统而直接进入系统，很神奇是吧，Docker的容器就是这个概念，只不过更加轻量更加迅速便捷。但是Live CD的害处就是你关机后作出的修改安装的软件全部gg，容器也是一样，一旦被直接推出，之前安装的gcc啊vim啊啥的就会全部gg掉。如果要保存修改，就需要将当前容器封装成一个新的镜像，这样下次启动这个新的镜像后之前作出的修改还都在。
+ 最后，是仓库（Repository）。各位在前面看到我写的pull和push什么的，有没有晕？不知道各位对于git熟悉不熟悉，Docker中的仓库很像git的代码仓库，你可以pull自己之前push到自己仓库的镜像到本地，也可以pull别人push到公共仓库的镜像到自己本地。说白了就是百度云盘，你可以上传（push）自己做好环境的Docker上去，也可以下载（pull）自己云端的镜像到本地。同时，我们知道百度云最大的特点就是分享（你懂的嘿嘿嘿），类比Docker，如果你得到百度云分享链接（别人的镜像名字、标签和别人的用户名），你还可以下载（pull）别人分享的镜像到自己的本地，别人也可以下载（pull）你的镜像，因为Docker仓库都是公共的。当然，每个免费用户有一个名额把自己的一个镜像设为私有，也就是禁止被分享给别人，类比百度云上你自己保存的而没有被生成分享链接的小姐姐。

## Docker的安装
### CentOS Docker 安装
Docker支持以下的CentOS版本：

+ CentOS 7 (64-bit)
+ CentOS 6.5 (64-bit) 或更高的版本
#### 前提条件
目前，CentOS 仅发行版本中的内核支持 Docker。
Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。
Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。

#### 使用 yum 安装（CentOS 7下）
Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 uname -r 命令查看你当前的内核版本
```
uname -r 3.10.0-327.el7.x86_64
```
![](https://s2.ax1x.com/2019/03/31/AD7Bid.png)

##### 安装Docker
```
yum -y install docker
```
ps. 安装之前可以 yum update一下
安装完成
![](https://s2.ax1x.com/2019/03/31/AD7IWn.png)
启动docker服务
```
service docker start
```
测试运行hello-world
```
docker run hello-world
```
![](https://s2.ax1x.com/2019/03/31/AD7LeU.png)
ps.第一次运行，会花些时间下载hello-world镜像

### Ubuntu Docker 安装
#### 前提条件
Docker 要求 Ubuntu 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的 Ubuntu 版本是否支持 Docker。

通过 uname -r 命令查看你当前的内核版本

#### 使用脚本安装 Docker
1、获取最新版本的 Docker 安装包
```
sudo apt-get update
sudo apt-get install -y docker.io
```
2、启动docker 后台服务
```
sudo service docker start
```

## Docker使用
### Docker Hello World
首先，下载容器镜像Docker官方网站专门有一个页面来存储所有可用的镜像，网址是： index.docker.io。你可以通过浏览这个网页来查找你想要使用的镜像，或者使用命令行的工具来检索。
```
docker search ubuntu15.10
```
![](https://s2.ax1x.com/2019/03/31/ADHWX6.png)
通过docker命令下载ubuntu镜像
ps.执行pull命令的时候要写完整的名字，比如"ubuntu或i386/ubuntu"
```
docker pull ubuntu
```
镜像查看命令：docker images

Docker 允许你在容器内运行应用程序， 使用 docker run 命令来在容器内运行一个应用程序。
```
docker run ubuntu /bin/echo "Hello World"
```
![](https://s2.ax1x.com/2019/03/31/ADbmEF.png)

各个参数解析：

+ docker: Docker 的二进制执行文件
+ run:与前面的 docker 组合来运行一个容器
+ ubuntu指定要运行的镜像
+ /bin/echo "Hello world": 在启动的容器里执行的命令

#### 运行交互式的容器
我们通过docker的两个参数 -i -t，让docker运行的容器实现"对话"的能力
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -i -t ubuntu /bin/bash
root@5a1be259f672:/# 
```
各个参数解析：

+ -t:在新容器内指定一个伪终端或终端
+ -i:允许你对容器内的标准输入 (STDIN) 进行交互

此时我们已进入一个 ubuntu15.10系统的容器
我们尝试在容器中运行命令ls查看当前系统的版本信息和当前目录下的文件列表
![](https://s2.ax1x.com/2019/03/31/ADbJHO.png)
我们可以通过运行exit命令或者使用CTRL+D来退出容器

#### 启动容器（后台模式）
使用以下命令创建一个以进程方式运行的容器
![](https://s2.ax1x.com/2019/03/31/Arhhhq.png)

在输出中，我们没有看到期望的"hello world"，而是一串长字符
这个长字符串叫做容器ID，对每个容器来说都是唯一的，我们可以通过容器ID来查看对应的容器发生了什么。

首先，我们需要确认容器有在运行，可以通过 docker ps 来查看
```
docker ps
```
![](https://s2.ax1x.com/2019/03/31/Arhx9x.png)

**CONTAINER ID**:容器ID
**NAMES**:自动分配的容器名称

在容器内使用docker logs命令，查看容器内的标准输出
![](https://s2.ax1x.com/2019/03/31/Ar4ZCt.png)

#### 停止容器
我们使用 **docker stop** 命令来停止容器:
![](https://s2.ax1x.com/2019/03/31/Ar4MDg.png)

通过docker ps查看，容器已经停止工作

