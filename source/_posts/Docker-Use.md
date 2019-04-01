---
title: Docker-使用
date: 2019-04-01 09:00:07
tags: Docker
categories: 虚拟化
---
![](https://s2.ax1x.com/2019/04/01/AsQyDA.jpg)
<!--more-->
## Docker 容器使用
### Docker 客户端
docker 客户端非常简单 ,我们可以直接输入 docker 命令来查看到 Docker 客户端的所有命令选项。
#### 运行一个web应用
前面我们运行的容器并没有一些什么特别的用处。
接下来让我们尝试使用 docker 构建一个 web 应用程序。
我们将在docker容器中运行一个 Python Flask 应用来运行一个web应用。
```
docker run -d -P training/webapp python app.py
```
![](https://s2.ax1x.com/2019/04/01/AsCY79.png)

ps.training/webapp镜像若你本地没有，会自动pull官网上的
参数说明:

+ **-d**:让容器在后台运行。
+ **-P**:将容器内部使用的网络端口映射到我们使用的主机上。
使用 docker ps 来查看我们正在运行的容器可以看到多了端口信息
Docker 开放了 5000 端口（默认 Python Flask 端口）映射到主机端口 32768 上。
这时我们可以通过浏览器访问WEB应用

![](https://s2.ax1x.com/2019/04/01/Asi8o9.png)

我们也可以指定 -p 标识来绑定指定端口。
```
docker run -d -p 5000:5000 training/webapp python app.py
```

#### 网络端口的快捷方式
通过docker ps 命令可以查看到容器的端口映射，docker还提供了另一个快捷方式：docker port,使用 docker port 可以查看指定 （ID或者名字）容器的某个确定端口映射到宿主机的端口号。
上面我们创建的web应用容器ID为:b570473f8bf5 名字为：cranky_brown
![](https://s2.ax1x.com/2019/04/01/AsiTFs.png)

#### 查看WEB应用程序日志
docker logs [ID或者名字] 可以查看容器内部的标准输出。
![](https://s2.ax1x.com/2019/04/01/AsiXOU.png)

#### 检查WEB应用程序
使用 docker inspect 来查看Docker的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息

![](https://s2.ax1x.com/2019/04/01/AsFk6K.png)

#### 停止WEB应用容器
```
docker stop b570473f8bf5
```

#### 重启WEB应用容器
```
docker start b570473f8bf5
```
docker ps -l 来查看正在运行的容器
正在运行的容器，我们可以使用 docker restart 命令来重启

#### 移除WEB应用容器
我们可以使用 docker rm 命令来删除不需要的容器
```
docker ps -a 查看所有容器，包括停止的
```
删除容器时，容器必须是停止状态，否则会报如下错误

![](https://s2.ax1x.com/2019/04/01/AsFJ0g.png)

### Docker 镜像使用
当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。

下面我们来学习：

+ 管理和使用本地 Docker 主机镜像
+ 创建镜像

#### 列出镜像列表
我们可以使用 docker images 来列出本地主机上的镜像
![](https://s2.ax1x.com/2019/04/01/AsF5jK.png)
各个选项说明:

+ **REPOSTITORY**：表示镜像的仓库源
+ **TAG**：镜像的标签
+ **IMAGE ID**：镜像ID
+ **CREATED**：镜像创建时间
+ **SIZE**：镜像大小

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSTITORY:TAG 来定义不同的镜像。
所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -t -i ubuntu:15.10 /bin/bash 
root@d77ccb2e5cca:/#
```
如果要使用版本为14.04的ubuntu系统镜像来运行容器时，命令如下：
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -t -i ubuntu:14.04 /bin/bash 
root@39e968165990:/#
```
如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。

#### 获取一个新的镜像
我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为：https://hub.docker.com/
我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个httpd的镜像来作为我们的web服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。
```
[root@izuf60l85kxuws81mq1immz ~]# docker search httpd
```
![](https://s2.ax1x.com/2019/04/01/AskZvV.png)

NAME:镜像仓库源的名称

DESCRIPTION:镜像的描述

OFFICIAL:是否docker官方发布

我们决定使用上图中的httpd 官方版本的镜像，使用命令 docker pull 来下载镜像。
![](https://s2.ax1x.com/2019/04/01/AskHMV.png)

#### 创建镜像
当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

+ 从已经创建的容器中更新镜像，并且提交这个镜像
+ 使用 Dockerfile 指令来创建一个新的镜像

#### 更新镜像
更新镜像之前，我们需要使用镜像来创建一个容器。
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -t -i ubuntu /bin/bash
root@e2532447b13a:/# 
```
在运行的容器内使用 apt-get update 命令进行更新。
在完成操作之后，输入 exit命令来退出这个容器。
使用docker ps -l 查找ID
![](https://s2.ax1x.com/2019/04/01/AsEgAg.png)
此时ID为e2532447b13a的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit来提交容器副本。
![](https://s2.ax1x.com/2019/04/01/AsVpDK.png)
各个参数说明：

+ **-m**:提交的描述信息
+ **-a**:指定镜像作者
+ **e2532447b13a**：容器ID
+ **ubuntu:v2**:指定要创建的目标镜像名

使用我们的新镜像 w3cschool/ubuntu 来启动一个容器
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -t -i ubuntu:v2 /bin/bash
root@f44fe837e095:/# 
```
这其实也是保存容器修改的方法

#### 构建镜像
我们使用命令 docker build ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

Dockerfile文件
```
FROM    centos:6.7
MAINTAINER      Arrietty "Arrietty.top"

RUN     /bin/echo 'root:123456' | chpasswd
RUN     useradd arrietty
RUN     /bin/echo 'arrietty:123456' | chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```
每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
第一条FROM，指定使用哪个镜像源
RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。
然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
第一条FROM，指定使用哪个镜像源
RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。
然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
![](https://s2.ax1x.com/2019/04/01/AsndyR.png)

参数说明：

+ -t ：指定要创建的目标镜像名
+ . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径
使用docker images 查看创建的镜像已经在列表中存在,镜像ID为6b874d8f824c
![](https://s2.ax1x.com/2019/04/01/Asn2SH.png)

#### 设置镜像标签
我们可以使用 docker tag 命令，为镜像添加一个新的标签。
docker tag 镜像ID，这里是 6b874d8f824c ,用户名称、镜像源名(repository name)和新的标签名(tag)。
使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。
![](https://s2.ax1x.com/2019/04/01/Asu9tU.png)

### Docker 容器连接
前面我们实现了通过网络端口来访问运行在docker容器内的服务。下面我们来实现通过端口连接到一个docker容器

#### 网络端口映射
我们创建了一个 python 应用的容器。
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -d -P training/webapp python app.py
2678861473c82550489f3b0de5e96e69e64e5c64fc50e327016997f6dac89a1d
```
另外，我们可以指定容器绑定的网络地址，比如绑定 127.0.0.1。
端口绑定的两种方式：

+ -P :是容器内部端口随机映射到主机的高端口。
+ -p : 是容器内部端口绑定到指定的主机端口。
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -d -p 5000:5000 training/webapp python app.py
2bbb8b715e40c853baa4f41cac91b141c93d11119ace0502c830baeb348e9b6c
```

另外，我们可以指定容器绑定的网络地址，比如绑定127.0.0.1。
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -d -p 127.0.0.1:5001:5002 training/webapp python app.py
95c6ceef88ca3e71eaf303c2833fd6701d8d1b2572b5613b5a932dfdfe8a857c
```
![](https://s2.ax1x.com/2019/04/01/AsKF58.png)

这样我们就可以通过访问127.0.0.1:5001来访问容器的5002端口。

上面的例子中，默认都是绑定 tcp 端口，如果要绑定 UPD 端口，可以在端口后面加上 /udp。
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

docker port 命令可以让我们快捷地查看端口的绑定情况。
```
[root@izuf60l85kxuws81mq1immz ~]# docker port adoring_stonebraker 5002
127.0.0.1:5001
```

#### Docker容器连接
端口映射并不是唯一把 docker 连接到另一个容器的方法。
docker有一个连接系统允许将多个容器连接在一起，共享连接信息。
docker连接会创建一个父子关系，其中父容器可以看到子容器的信息。

**容器命名**
当我们创建一个容器的时候，docker会自动对它进行命名。另外，我们也可以使用--name标识来命名容器，例如：
```
[root@izuf60l85kxuws81mq1immz ~]# docker run -d -P --name yourname training/webapp python app.py
```





