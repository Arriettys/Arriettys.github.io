---
title: Docker-网络
date: 2019-04-04 10:16:43
tags: Docker
categories: 虚拟化
---
![](https://s2.ax1x.com/2019/04/11/AHF8bT.md.jpg)
<!--more-->
## 网络基础
### Docker网络命令
docker network命令是配置和管理容器网络的主要命令
```
$ docker network
```
![](https://s2.ax1x.com/2019/04/04/AgTbxH.png)
命令输出显示如何使用该命令以及所有docker network子命令。从输出中可以看出，该命令允许创建新网络，列出现有网络，检查网络和删除网络。它还允许从网络连接和断开容器。

### 列出网络
运行docker network ls命令以查看当前Docker主机上的现有容器网络。
```
$ docker network ls
```
![](https://s2.ax1x.com/2019/04/04/Ag7PzQ.png)
上面的输出显示了作为Docker标准安装的一部分创建的容器网络。
你创建的新网络也将显示在docker network ls命令的输出中。

你可以看到，每一个网络获得一个唯一的ID和NAME。每个网络还与单个驱动程序相关联。请注意，“网桥”网络和“主机”网络与其各自的驱动程序具有相同的名称。

## 检查网络
该docker network inspect命令用于查看网络配置详细信息。这些细节包括; 名称，ID，驱动程序，IPAM驱动程序，子网信息，连接容器等。

在docker host 中使用docker network inspect <network>去查看容器的网络配置的详细信息。以下命令显示了所调用网络bridge的详细信息。

```
$ docker network inspect bridge
```
![](https://s2.ax1x.com/2019/04/04/Ag7v6J.png)
ps.该docker network inspect命令的语法是docker network inspect <network>，其中<network>可以是网络名称或网络ID。在上面的示例中，我们显示了名为“bridge”的网络的配置详细信息。不要将它与“桥”驱动程序混淆。

### 列出网络驱动程序插件
该docker info命令显示了许多有关Docker安装的有趣信息。
运行该docker info命令并找到网络插件列表。

```
$ docker info
```
![](https://s2.ax1x.com/2019/04/04/AgHV6H.png)

## 桥接网络
###基础知识
每个纯净安装的docker都带有一个bridge预构建网络，可以使用docker network ls 命令查看

```
NETWORK ID          NAME                DRIVER              SCOPE
3430ad6f20bf        bridge              bridge              local
a7449465c379        host                host                local
06c349b9cc77        none                null                local
```
上面的输出显示桥接网络与桥接驱动程序相关联。重要的是要注意网络和驱动程序是连接的，但它们并不相同。在这个例子中，网络和驱动程序具有相同的名称 - 但它们不是一回事！

上面的输出还显示桥接网络是本地范围的。这意味着网络仅存在于此Docker主机上。所有使用网桥驱动程序的网络都是如此- 网桥驱动程序提供单主机网络。

使用网桥驱动程序创建的所有网络都基于Linux网桥（也称为虚拟交换机）。
![](https://s2.ax1x.com/2019/04/11/A73Q9P.png)
Linux虚拟网桥的特点:

 + 可以设置IP地址
 + 相当于拥有一个隐藏的虚拟网卡
 
docker0的地址划分:

+ IP:172.17.42.1 子网掩码: 255.255.0.0
+ MAC: 02:42:ac:11:00:00 到 02:42:ac:11:ff:ff
+ 总共提供65534个地址

docker守护进程在一个容器启动时，实际上它要创建网络连接的两端。一端是在容器中的网络设备，而另一端是在运行docker守护进程的主机上打开一个名为veth*的一个接口，用来实现docker这个网桥与容器的网络通信。
![](https://s2.ax1x.com/2019/04/11/A73JBQ.png)

安装brctl并使用它来列出Docker主机上的Linux桥接器。centos使用yum install bridge-utils -y安装，debian系可以使用apt-get。

列出Docker主机上的网桥

```
$ brct1 show
```
![](https://s2.ax1x.com/2019/04/04/AgqVeA.png)
上面的输出显示了一个名为docker0的 Linux桥。这是为**桥（bridge）**是自动创建的**网桥（bridge network）**

你还可以使用该ip a命令查看docker0网桥的详细信息。

```
$ ip a
```
```
<Snip>
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:52:ed:52:f7 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
```

运行一个docker容器，在容器中查看它的网络设备(如果没有ifconfig命令，通过apt-get install -y net-tools)

```
$ ifconfig
eth0 Link encap:Ethernet HWaddr 02:42:ac:11:00:02 
inet addr:172.17.0.2 Bcast:0.0.0.0 Mask:255.255.0.0
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:145 errors:0 dropped:0 overruns:0 frame:0
TX packets:60 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0 
RX bytes:184985 (184.9 KB) TX bytes:4758 (4.7 KB)

lo Link encap:Local Loopback 
inet addr:127.0.0.1 Mask:255.0.0.0
UP LOOPBACK RUNNING MTU:65536 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0 
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)
```
docker已经自动创建了eth0的网卡，注意观察ip地址和mac地址。不要退出容器，再运行如下查看网桥的状态

```
$ sudo brctl show
bridge name    bridge id           STP    enabled    interfaces
docker0    8000.0242ed943d02       no              　 veth95521e6
```
我们看到在interface中多了一个veth*的这样一个接口。通过ifconfig命令同样可以看到这个网络接口。
**自定义docker0**
修改docker0默认分配的ip地址:

	sudo ifconfig docker0 192.168.200.1 netmask 255.255.255.0
修改完后，重启docker服务 sudo service docker restart. 新运行的容器地址就变成了新的ip地址了。

添加虚拟网桥

	sudo brctl addbr br0
	sudo ifconfig br0 192.168.100.1 netmask 255.255.255.0
更改docker守护进程的启动配置:

vim /etc/default/docker 中添加 DOCKER_OPS的值 -b=br0.
重启docker服务即可。


### 连接容器
新容器的默认网络是网桥，这意味着除非你指定其他网络，否则所有新容器都将连接到网桥。

通过运行docker run -dt ubuntu sleep infinity创建一个新容器。

```
$ docker run -dt ubuntu sleep infinity
```

此命令将基于ubuntu:latest映像创建一个新容器，并将运行该sleep命令以使容器在后台运行。你可以通过运行docker ps验证我们的示例容器已启动。

由于未在docker run命令中指定网络，因此容器将添加到网桥网络中。

再次运行brctl show命令。
![](https://s2.ax1x.com/2019/04/04/AgOE5t.png)
请注意docker0网桥现在如何连接接口。此接口将docker0桥连接到刚刚创建的新容器。

通过运行docker network inspect bridge再次检查桥接网络，以查看连接到它的新容器。
![](https://s2.ax1x.com/2019/04/04/AgOe8f.png)

### 测试网络连接
上一个docker network inspect命令的输出显示新容器的IP地址。在前面的例子中它是“172.17.0.2”，但你的可能会有所不同。

在Docker主机的shell提示符中运行ping -c5 <IPv4 Address>，ping容器的IP地址。

```
ping -c5 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.031 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.034 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.041 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.047 ms

--- 172.17.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4075ms
rtt min/avg/max/mdev = 0.031/0.041/0.055/0.011 ms
```

面的回复表明Docker主机可以通过桥接网络ping容器。但是，我们也可以验证容器是否也可以连接到外部世界。让我们登录容器，安装ping程序，然后ping www.github.com。

首先，我们需要获取上一步中启动的容器的ID。你可以使用docker ps查找。

接下来，让我们通过运行在该ubuntu容器中运行一个shell 

```
$ docker exec -it <CONTAINER ID> /bin/bash。
```
接下来，我们需要安装ping程序。

```
$ apt-get update && apt-get install -y iputils-ping
```
ping www.github.com

```
$ ping -c5 www.github.com
PING www.docker.com (104.239.220.248) 56(84) bytes of data.
64 bytes from 104.239.220.248: icmp_seq=1 ttl=45 time=38.1 ms
64 bytes from 104.239.220.248: icmp_seq=2 ttl=45 time=37.3 ms
64 bytes from 104.239.220.248: icmp_seq=3 ttl=45 time=37.5 ms
64 bytes from 104.239.220.248: icmp_seq=4 ttl=45 time=37.5 ms
64 bytes from 104.239.220.248: icmp_seq=5 ttl=45 time=37.5 ms

--- www.docker.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 37.372/37.641/38.143/0.314 ms
```
最后，让我们通过运行exit来断开我们的shell与容器的连接。

我们也应该停止这个容器，可以通过运行docker stop <CONTAINER ID>来清理这个测试。

这表明新容器可以ping互联网，因此具有有效且可正常工作的网络配置。
ps.使用阿里云的服务器实验记得打开安全组哦

### 为外部连接配置NAT
在此步骤中，我们将启动一个新的NGINX容器，并将Docker主机上的端口8080映射到容器内的端口80。这意味着在端口8080上访问Docker主机的流量将传递到容器内的端口80。
ps.如果从官方NGINX映像启动新容器而未指定要运行的命令，则容器将在端口80上运行基本Web服务器。

通过运行启动基于官方NGINX映像的新容器

```
$ docker run --name web1 -d -p 8080:80 nginx。
```
通过运行docker ps查看容器状态和端口映射。

现在容器正在运行并映射到主机接口上的端口，你可以测试与NGINX Web服务器的连接。只需将Web浏览器指向Docker主机的IP和端口8080即可。此外，如果你尝试在不同的端口号上连接到相同的IP地址，它将失败。

如果由于某种原因你无法从Web浏览器打开会话，则可以使用该curl 127.0.0.1:8080命令从Docker主机进行连接。

```
$ curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<Snip>
<head>
<title>Welcome to nginx!</title>
    <Snip>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 覆盖网络
该步涉及到Swarm的知识点，以后有机会再补上

转载自：
https://training.play-with-docker.com/docker-networking-hol/



