---
title: Docker-Dockerfile命令详解
date: 2019-04-02 09:54:21
tags: Docker
categories: 虚拟化
---
制作Dockerfile为Docker入门学习的第一步（当然，除了环境搭建）
![](https://s2.ax1x.com/2019/04/02/AyfYFA.png)
<!--more-->
## FROM
功能为指定基础镜像，并且必须是第一条指令。
如果不以任何镜像为基础，那么写法为：FROM scratch。
同时意味着接下来所写的指令将作为镜像的第一层开始
语法：

FROM <image>
FROM <image>:<tag>
FROM <image>:<digest> 
三种写法，其中<tag>和<digest> 是可选项，如果没有选择，那么默认值为latest


## FUN
用于指定docker build过程中运行的程序      其可以是任何命令  但是需要基础镜像的shell环境的支持

RUN命令有两种格式

1. RUN <command>
2. RUN ["executable", "param1", "param2"]

第一种后边直接跟shell命令
在linux操作系统上默认 /bin/sh -c
在windows操作系统上默认 cmd /S /C

第二种是类似于函数调用。
可将executable理解成为可执行文件，后面就是两个参数。
两种写法比对：

RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME
RUN ["/bin/bash", "-c", "echo hello"]
注意：多行命令不要写多个RUN，原因是Dockerfile中每一个指令都会建立一层.
 多少个RUN就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错。
RUN书写时的换行符是\

## CMD
CMD指令的首要目的在于为启动的容器指定默认要运行的程序,且其运行结束后,容器也将终止. 不过CMD指定的命令其可以被docker run的命令行选项所覆盖 
CMD命令的执行时间周期 就是容器的生命周期 CMD一旦执行完毕  容器就会立即停止  CMD指令只有最后一个指令生效
CMD指令一般不会单独使用  通常都需要配合 ENTRYPOINT指令来设置
语法有三种写法

1. CMD ["executable","param1","param2"]
2. CMD ["param1","param2"]
3. CMD command param1 param2
第三种比较好理解了，就时shell这种执行方式和写法

第一种和第二种其实都是可执行文件加上参数的形式

举例说明两种写法：

CMD [ "sh", "-c", "echo $HOME" 
CMD [ "echo", "$HOME" ]
补充细节：这里边包括参数的一定要用双引号，就是",不能是单引号。千万不能写成单引号。

原因是参数传递后，docker解析的是一个JSON array

不要把RUN和CMD搞混了。
RUN是构件容器时就运行的命令以及提交运行结果
CMD是容器启动时执行的命令，在构件时并不运行，构件时紧紧指定了这个命令到底是个什么样子

## ENTRYPOINT
主要用来指定shell的  把shell作为容器中第一个启动进程 通过接收CMD命令 把命令作为参数启动指定的子进程
用来指定容器内核启动的第一个进程 只有PID为1的进程才能接收系统发送给容器的系统信号
由于ENTRYPOINT启动的程序不会被docker run命 令行指定的参数所覆盖.而且,这些命令行参数会被当作参数传递给ENTRYPOINT指定指定的程序 
不过,docker run命令的--entrypoint选项的参数可覆盖ENTRYPOINT指令指定的程序 
docker run命令传入的命令参数会覆盖CMD指令的内容并且附加到 ENTRYPOINT命令最后做为其参数使用 
语法：

ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2

与CMD比较说明（这俩命令太像了，而且还可以配合使用）：

1. 相同点：

只能写一条，如果写了多条，那么只有最后一条生效

容器启动时才运行，运行时机相同

2. 不同点：
 ENTRYPOINT不会被运行的command覆盖，而CMD则会被覆盖

 如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD指令不是一个完整的可执行命令，那么CMD指定的内容将会作为ENTRYPOINT的参数
如下：

FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]

如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD是一个完整的指令，那么它们两个会互相覆盖，谁在最后谁生效
如下：

FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ls -al
那么将执行ls -al ,top -b不会执行。

## LABEL
功能是为镜像指定标签
语法：

LABEL <key>=<value> <key>=<value> <key>=<value> ...
 一个Dockerfile种可以有多个LABEL，如下：

LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
 但是并不建议这样写，最好就写成一行，如太长需要换行的话则使用\符号

如下：

LABEL multi.label1="value1" \
multi.label2="value2" \
other="value3"
 
说明：LABEL会继承基础镜像中的LABEL，如遇到key相同，则值覆盖

## MAINTAINER
指定作者
语法：

MAINTAINER <name>

## EXPOSE
功能为暴漏容器运行时的监听端口给外部
语法：

EXPOSE <port>[/<protocol>] [<port>[/<protocol>] ...]   EXPOSE指令可一次指定多个端口  EXPOSE 11211/udp 11211/tcp

但是EXPOSE并不会使容器访问主机的端口
如果想使得容器与主机的端口有映射关系，必须在容器启动的时候加上 -P参数

## ENV指令
用于为镜像定义所需的环境变量,并可被Dockerfile文件中位于其后的其它指令（如ENV、ADD、COPY等）所调用
语法：

ENV <key> <value>     
ENV <key>=<value> ... 

第二种格式可用一次设置多个变量,每个变量为一个"<key>=<value>"的 键值对.如果<value>中包含空格,可以以反斜线(\)进行转义,也可通过对<value>加引号进行标识.另外,反斜线也可用于续行
在docker run的时候可以通过 -e 选项直接覆盖Dockerfile文件中已经指定的ENV或者添加为容器新的ENV变量的

## COPY
从dockerfile的工作目录中 复制指定文件到目标镜像的文件系统中
语法：

COPY <src> ... <dest> 
COPY ["<src>",... "<dest>"] 
<dest>:目标路径,即正在创建的image的文件系统路径;建议为<dest>使用绝对路径   否则,COPY指定则以WORKDIR为其起始路径
注意:  在路径中有空白字符时,通常使用第二种格式 
复制规则：

src的路径不能是当前工作目录之上的目录或者文件 只能是工作目录中的文件或者子目录
如果<src>是目录,则其内部文件或子目录会被递归复制,但<src>目录自身不会被复制  相当于shell中的 cp  -r  /root/dir/*    /tmp
如果指定了多个<src>或在<src>中使用了通配符  则<dest>必须是一个目录,且必须以/结尾 
如果<dest>事先不存在,它将会被自动创建,这包括其父目录路径

## ADD
ADD指令类似于COPY指令    ADD支持使用TAR文件和URL路径 
如果<src>为URL且<dest>不以/结尾,则<src>指定的文件将被下载并直接被 创建为<dest>.如果<dest>以/结尾,则文件名URL指定的文件将被直接下载 并保存为<dest>/<filename> 

如果<src>是一个本地系统上的压缩格式的tar文件,它将被展开为一个目录 ,其行为类似于“tar -x”命令；然而,通过URL获取到的tar文件将不会自动 展开 

如果<src>有多个,或其间接或直接使用了通配符,则<dest>必须是一个以/结 尾的目录路径；如果<dest>不以/结尾,则其被视作一个普通文件,<src>的内 容将被直接写入到<dest>
语法：

ADD <src>... <dest>
ADD ["<src>",... "<dest>"] 

## VOLUME
用于在image中创建一个挂载点目录   以挂载Docker host上的卷或 其它容器上的卷 
语法：

VOLUME <mountpoint>  
VOLUME ["<mountpoint>"] 

如果挂载点目录路径下此前在文件存在,  docker run命令会在卷挂载完成后将此前的所有文件复制到新挂载的卷中
不能指定宿主机上面的目录路径  只能创建docker manage volume

## USER
用于指定运行image时的或运行Dockerfile中任何RUN、CMD或 ENTRYPOINT指令指定的程序时的用户名或UID 
默认情况下container的运行身份为root用户 
语法：

USER daemo
USER UID

需要注意的是<UID>可以为任意数字 但实践中其必须为/etc/passwd中某用户的有效UID  否则docker run命令将运行失败

## WORKDIR
用于为Dockerfile中所有的RUN、CMD、ENTRYPOINT、COPY和 ADD指定设定工作目录
语法：

WORKDIR /path/to/workdir

在Dockerfile文件中,WORKDIR指令可出现多次,其路径也可以为相对路径,不过 ,其是相对此前一个WORKDIR指令指定的路径 
另外 WORKDIR也可调用由ENV指定定义的变量 

## ARG
语法：

ARG <name>[=<default value>]

设置变量命令，ARG命令定义了一个变量，在docker build创建镜像的时候，使用 --build-arg <varname>=<value>来指定参数

如果用户在build镜像时指定了一个参数没有定义在Dockerfile中，那么将有一个Warning

提示如下：

**[Warning] One or more build-args [foo] were not consumed.**

我们可以定义一个或多个参数，如下：

FROM busybox
ARG user1
ARG buildno
...
也可以给参数一个默认值：

FROM busybox
ARG user1=someuser
ARG buildno=1
...
如果我们给了ARG定义的参数默认值，那么当build镜像时没有指定参数值，将会使用这个默认值

## ONBUILD
用于在Dockerfile中定义一个触发器 
Dockerfile用于build映像文件,此映像文件亦可作为base image被另一个Dockerfile用作FROM指令的参数,并以之构建新的映像文件
在后面的这个Dockerfile中的FROM指令在build过程中被执行时,将会“触发”创建其base image的Dockerfile文件中的ONBUILD指令定义的触发器
语法：

ONBUILD [INSTRUCTION]

这个命令只对当前镜像的子镜像生效。
比如当前镜像为A，在Dockerfile种添加：
ONBUILD RUN ls -al
这个 ls -al 命令不会在A镜像构建或启动的时候执行
此时有一个镜像B是基于A镜像构建的，那么这个ls -al 命令会在B镜像构建的时候被执行。

## HEALTHCHECK
 容器健康状况检查命令
docker引擎判定容器是否健康的机制是仅仅判定容器是否处于运行状态
判定运行的容器是否正常运行 并不能单一的检测容器是否正在运行 需要更加具体化 需要检测容器中的主进程是否能正常提供服务才行
需要借助外部命令检测 如检测nginx容器是否正常  可以使用命令请求主页 wget -O - -q a1f2903f6de3 获取返回结果 进行健康检查

语法有两种：

HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE

第一个的功能是在容器内部运行一个命令来检查容器的健康状况
第二个的功能是在基础镜像中取消健康检查命令
[OPTIONS]的选项支持以下三中选项：

    --interval=DURATION 两次检查默认的时间间隔为30秒
    --timeout=DURATION 健康检查命令运行超时时长，默认30秒
    --retries=N 当连续失败指定次数后，则容器被认为是不健康的，状态为unhealthy，默认次数是3
    
注意：
HEALTHCHECK命令只能出现一次，如果出现了多次，只有最后一个生效。
CMD后边的命令的返回值决定了本次健康检查是否成功，具体的返回值如下：

0: success - 表示容器是健康的
1: unhealthy - 表示容器已经不能工作了
2: reserved - 保留值


参考自：
https://www.cnblogs.com/dazhoushuoceshi/p/7066041.html
https://www.cnblogs.com/yxh168/p/9419235.html