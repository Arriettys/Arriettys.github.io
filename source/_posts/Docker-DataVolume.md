---
title: Docker-数据卷
date: 2019-04-09 10:29:34
tags: Docker
categories: 虚拟化
---
docker的镜像是由多个只读的文件系统叠加在一起形成的。当我们在我启动一个容器的时候，docker会加载这些只读层并在这些只读层的上面(栈顶)增加一个读写层。这时如果修改正在运行的容器中已有的文件，那么这个文件将会从只读层复制到读写层。该文件的只读版本还在，只是被上面读写层的该文件的副本隐藏。当删除docker,或者重新启动时，之前的更改将会消失。在Docker中，只读层及在顶部的读写层的组合被称为Union File System（联合文件系统）。

为了很好的实现数据保存和数据共享，Docker提出了Volume这个概念，简单的说就是绕过默认的联合文件系统，而以正常的文件或者目录的形式存在于宿主机上。又被称作数据卷。
<!--more-->
## Volume

+ 通过数据卷可以在容器之间实现共享和重用
+ 对数据卷的修改会立马生效(非常适合作为开发环境)
+ 对数据卷的更新,不会影响镜像
+ 卷会一直存在,直到没有容器使用

### 初始化Volume
在使用docker run的时候我们可以通过 -v 来创建一个数据卷并挂载到容器上，在一次run中多次使用可以挂载多个容器。

如果使用Dockerfile方式进行初始化时可以使用 VOLUME 来添加一个或者多个新的卷到由该镜像创建的任意容器。

#### 创建一个数据卷

	docker run -p 8080:80 -d --name volume-test1 -v /usr/share/nginx/html nginx
上面的命令的意思是，我们创建了一个名称为volume-test1的容器，将本机的8080端口映射到容器中nginx服务器的默认web访问端口80下，创建一个数据卷，并挂载到容器的/usr/share/nginx/html目录下。

这时我们就可以绕过联合文件系统，直接在主机上操作该目录了，任何在该镜像/usr/share/nginx/html下的文件都会被复制到Volume。

我们可以通过docker inspect指令找到Volume在主机上的存储位置

	docker inspect shanlei-nginx
docker inspect指令后面的参数可以跟容器名称。通过这个命令我们可以获得容器所有的相关信息。我们需要看这一部分

```
...
...

"Mounts": [
            {
                "Type": "volume",
                "Name": "057f911105d4c77d2cfe16ee6acb7f5a43f2643d571708da40f5db55e27b1155",
                "Source": "/var/lib/docker/volumes/057f911105d4c77d2cfe16ee6acb7f5a43f2643d571708da40f5db55e27b1155/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
],
...
...
```
这说明docker把本机的“Source”指向目录，也就是

	/var/lib/docker/volumes/057f911105d4c77d2cfe16ee6acb7f5a43f2643d571708da40f5db55e27b1155/_data
在Linux中我们可以ls“Source”指向的路径，可以看到nginx的index.html文件。

只要将主机的目录挂载到容器的目录上，那改变就会立即生效。在Dockerfile中我们可以通过VOLUME达到相同的目的

	FROM nginx
	VOLUME /usr/share/nginx/html
	
#### 目录作为数据卷
通过-v标识可以将主机的目录挂载到容器中去

	sudo docker run -d -p 8080:80 --name volume-test2 -v $PWD/html:/usr/share/nginx/html nginx
上面的命令将本机的$PWD/html目录挂载到容器的/usr/share/nginx/html目录。$PWD在是一个系统环境变量，指代当前目录环境。这个功能在进行测试的时候十分方便,比如用户可以放置一些程序到本地目录中,来查看容器是否正常工作。本地目录的路径必须是绝对路径,如果目录不存在 Docker 会自动为你创建它。
ps.Dockerfile中不支持这种语法.
	
docker挂载数据卷的默认权限是读写，当然我们可以通过指令:ro指定为只读。

	sudo docker run -d -p 8080:80  --name volume-test2 -v $PWD/html:/usr/share/nginx/html:ro nginx
在我的本机主机的$PWD/html中有一个index.html文件，内容如下：

	this is old
本机下对其修改，通过8080端口访问可以直接观察到。

#### 文件作为数据卷
我们也可以使用-v指令从主机挂载单个文件到容器中去

	sudo docker run -d -p 8080:80  --name volume-test2 -v $PWD/html/index.html:/usr/share/nginx/html:ro nginx
当修改本机$PWD/html/index.html时,容器中的/usr/share/nginx/html/index.html也会随之变化。效果和上面相同，在这里就不做赘述了。

ps.如果直接挂载一个文件,很多文件编辑工具,包括 vi 或者 sed --in-place ,可能会造成文件 inode的改变,从 Docker 1.1 .0起,这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。

### 数据共享
如果想要实现容器间的数据共享，那么需要授权一个容器访问另一个容器的Volume。我们可以在使用docker run时使用-volumes-from参数来进行指定。



	docker run -it  --volumes-from volume-test2 ubuntu /bin/bash
![](https://s2.ax1x.com/2019/04/09/AISaUP.png)

## 数据卷容器
如果我们有一些持续更新的数据需要在容器之间共享,最好创建数据卷容器。常见的使用场景是使用纯数据容器来持久化数据库、配置文件或者数据文件等。

数据卷容器，其实就是一个正常的容器,专门用来提供数据卷供其它容器挂载的。

首先我们需要先创建一个数据卷容器

	docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres

然后通过--volumes-from指令参数来挂载 dbdata 容器中的数据卷。

	docker run -d --volumes-from dbdata --name db1 training/postgres
当然，我们也可以使用多个 --volumes-from 参数来从多个容器挂载多个数据卷。 也可以从其他已经挂载了数据卷的容器
来挂载数据卷。
	
	docker run -d --name db3 --volumes-from db1 training/postgres
现在我们进入容器中查看数据卷容器是否挂载成功

	docker exec -it db1 /bin/bash
![](https://s2.ax1x.com/2019/04/09/AIpNM4.png)
说明数据卷容器挂载已经成功了。

ps.如果删除了挂载的容器(包括 dbdata、db1 和 db2等),数据卷并不会被自动删除。如果要删除一个数据卷,必须在删除最后一个还挂载着它的容器时使用 docker rm -v 命令来指定同时删除关联的容器。

### 备份和恢复
我们可以利用数据卷对其中的数据进行进行备份、恢复和迁移。
首先使用 --volumes-from 标记来创建一个加载 dbdata 容器卷的容器,并从本地主机挂载当前到容器的 /backup 目录。命令如下:

	docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
容器启动后,使用了 tar 命令来将 dbdata 卷备份为本地的 /backup/backup.tar 。

如果要恢复数据到一个容器,首先创建一个带有数据卷的容器 dbdata2。

	docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
然后创建另一个容器,挂载 dbdata2 的容器,并使用 untar 解压备份文件到挂载的容器卷中。

	docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf　/backup/backup.tar
	
转载自：
**https://www.cnblogs.com/lishanlei/p/9503596.html**
