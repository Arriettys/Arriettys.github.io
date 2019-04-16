---
title: Docker-Compose
date: 2019-04-13 14:15:46
tags: Docker
categories: 虚拟化
---
前面我们使用 Docker 的时候，定义 Dockerfile 文件，然后使用 docker build、docker run 等命令操作容器。然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，那么效率之低，维护量之大可想而知

使用 Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具
<!--more-->

## 安装 Docker Compose
安装 Docker Compose 可以通过下面命令自动下载适应版本的 Compose，并为安装脚本添加执行权限

	$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
	$ sudo chmod +x /usr/local/bin/docker-compose

查看安装是否成功

	docker-compose -v

## Docker-Compose.yml 文件详解
先来看一份 docker-compose.yml 文件，不用管这是干嘛的，只是有个格式方便后文解说：

```
version: '2'
services:
  web:
    image: dockercloud/hello-world
    ports:
      - 8080
    networks:
      - front-tier
      - back-tier
 
  redis:
    image: redis
    links:
      - web
    networks:
      - back-tier
 
  lb:
    image: dockercloud/haproxy
    ports:
      - 80:80
    links:
      - web
    networks:
      - front-tier
      - back-tier
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 
 
networks:
  front-tier:
    driver: bridge
  back-tier:
driver: bridge
```
以看到一份标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分，下面先来看 services 的书写规则。
### service
#### image

```
services:
  web:
    image: hello-world
```
在 services 标签下的第二级标签是 web，这个名字是用户自己自定义，它就是服务名称。
image 则是指定服务的镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
例如下面这些格式都是可以的：

```
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```
#### build
服务除了可以基于指定的镜像，还可以基于一份 Dockerfile，在使用 up 启动之时执行构建任务，这个构建标签就是 build，它可以指定 Dockerfile 所在文件夹的路径。Compose 将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器。

	build: /path/to/build/dir
也可以是相对路径，只要上下文确定就可以读取到 Dockerfile。

	build: /path/to/build/dir
设定上下文根目录，然后以该目录为准指定 Dockerfile。
```
build:
  context: ../
  dockerfile: path/of/Dockerfile
```
注意 build 都是一个目录，如果你要指定 Dockerfile 文件需要在 build 标签的子级标签中使用 dockerfile 标签指定，如上面的例子。
如果你同时指定了 image 和 build 两个标签，那么 Compose 会构建镜像并且把镜像命名为 image 后面的那个名字。

```
build: ./dir
image: webapp:tag
```
既然可以在 docker-compose.yml 中定义构建任务，那么一定少不了 arg 这个标签，就像 Dockerfile 中的 ARG 指令，它可以在构建过程中指定环境变量，但是在构建成功后取消，在 docker-compose.yml 文件中也支持这样的写法：

```
build:
  context: .
  args:
    buildno: 1
    password: secret
```
与 ENV 不同的是，ARG 是允许空值的。同样构建时的命令，这里都能作为标签用，如command。
这样构建过程可以向它们赋值。
ps.YAML 的布尔值（true, false, yes, no, on, off）必须要使用引号引起来（单引号、双引号均可），否则会当成字符串解析。

#### container_name
前面说过 Compose 的容器名称格式是：<项目名称><服务名称><序号>
虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定：

	container_name: app
这样容器的名字就指定为 app 了。

### depends_on
在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。
例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。
例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务：

```
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
先就写这些，等有空在整理，有兴趣的可以去看[官方文档](https://docs.docker.com/compose/compose-file/#service-configuration-reference)