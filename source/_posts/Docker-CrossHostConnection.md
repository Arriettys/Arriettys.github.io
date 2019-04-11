---
title: Docker-跨主机连接
date: 2019-04-11 10:15:35
tags: Docker
categories: 虚拟化
---
![](https://s2.ax1x.com/2019/02/18/kcaap8.jpg)
这里指的是不同宿主机之间的容器连接
## Docker网桥实现跨主机容器连接
docker网桥实现跨主机连接的网络拓扑图如下：
![](https://img-blog.csdn.net/20180528020606566)
在同一个docker主机中，docker容器通过虚拟网桥连接(docker0)，如果将连接容器的网桥docker0也桥接到宿主机提供的网卡上，将docker0分配的IP地址和宿主机的IP地址设置为同一个网段，就相当于将docker容器和宿主机连接到了一起，这样就可以实现跨主机的docker容器通信。

主机端，修改/etc/network/interfaces 文件添加网桥

```
auto br0	//名称
iface br0 inet static	//ip分配方式
address 10.211.55.3	//ip
netmask 255.255.255.0	//子网掩码
gateway 10.211.55.1	//默认的网关
bridge_ports eth0		//指明将本地的物理网卡eth0连接到新建的网桥之上
```
docker端，修改/etc/default/docker 文件

 + **-b**：指定使用宿主机的网桥名
 	-b = br0
  + **--fixed-cidr**：指定对docker容器分配的ip段
  	-fixed-cidr=172.25.11.1/24
 
另一台也是同样方法设置，之后随意进入一个容器ping另一台容器ip测试即可

**网桥配置跨主机容器连接的优点：**

+ 配置简单，不依赖第三方软件
**网桥配置跨主机容器连接的缺点：**

+ 容器和主机在同一网段，划分IP需要特别小心
+ 需要网段控制权，在生产环境中不容易实现
+ 不容易管理，兼容性不佳

## Open vSwitch实现跨主机容器连接
Open vSwitch是一个高质量、多层虚拟交换机。使用Apache2.0许可协议，旨在通过编程扩展，使庞大的网络自动化（配置、管理、维护），同时还支持标准的管理接口和协议。
网络拓扑如下：
![](https://img-blog.csdn.net/20180528020637777)
这张图中蓝色部分就是我们上一节介绍的虚拟网桥br0，容器通过虚拟网桥实现同主机之间的连接。而其上层黄色部分open vSwitch创建的ovs网桥obr0，ovs通过gre隧道协议接口实现跨主机的网络连接。

GRE是通用路由协议封装；隧道技术（Tunneling）是一种通过使用互联网络的基础设施在网络之间传递数据的方式。使用隧道传递的数据（或负载）可以是不同协议的数据帧或包。隧道协议将其它协议的数据帧或包重新封装然后通过隧道发送。新的帧头提供路由信息，以便通过互联网传递被封装的负载数据。
### 具体步骤
A、B两台主机
A：192.168.59.103	docker网段：192.168.1.1
B：192.168.59.103	docker网段：192.168.2.4
以下以A实验
确认安装了Open vSwitch

```
$ apt-get install openvswitch-switch
```
1.建立网桥

```
$ sudo ovs-vsctl add-br obr0
```
2.添加gre接口

```
$ sudo ovs-vsctl add-port obr0 gre0
$ sudo ovs-vsctl set interface gre0 type=gre options:remote_ip=address	//设置远程主机ip，address处填ip
```
3.本机docker容器需要的网桥

```
$ sudo brctl addbr br0
$ sudo ifconfig br0 192.168.1.1/24 netmask 255.255.255.0
$ sudo brctl addif br0 obr0		//添加ovs网桥连接
$ sudo brctl show 				//查看
```
4.修改docker容器网桥，编辑/etc/default/docker

5.开启一个docker容器ping测试
B主机的ip虽然能ping通，但其docker容器的却不能，因为不同网段需要查找路由表确定地址，故还需要将docker容器的网段

6.添加不同Docker容器网段路由
在A中添加B的docker路由，B相反

```
$ sudo ip route add 192.168.2.0/24 via 192.168.59.104(主机) 
```

## 使用weave实现跨主机容器连接
Weave是由weaveworks公司开发的解决Docker跨主机网络的解决方案，它能够创建一个虚拟网络，用于连接多台主机上的Docker容器，这样容器就像被接入了同一个网络交换机，那些使用网络的应用程序不必去配置端口映射和链接等信息。
![](https://img-blog.csdn.net/20180528020700646)
Weave会在主机上创建一个网桥,每一个容器通过 veth pair 连接到该网桥上，同时网桥上有个 Weave router 的容器与之连接，该router会通过连接在网桥上的接口来抓取网络包(该接口工作在Promiscuous模式)。

在每一个部署Docker的主机(可能是物理机也可能是虚拟机)上都部署有一个W(即Weave router)，它本身也可以以一个容器的形式部署。Weave run的时候就可以给每个veth的容器端分配一个ip和相应的掩码。veth的网桥这端就是Weave router容器，并在Weave launch的时候分配好ip和掩码。

Weave网络是由这些weave routers组成的对等端点(peer)构成，每个对等的一端都有自己的名字，其中包括一个可读性好的名字用于表示状态和日志的输出，一个唯一标识符用于运行中相互区别，即使重启Docker主机名字也保持不变，这些名字默认是mac地址。

每个部署了Weave router的主机都需要将TCP和UDP的6783端口的防火墙设置打开，保证Weave router之间控制面流量和数据面流量的通过。控制面由weave routers之间建立的TCP连接构成，通过它进行握手和拓扑关系信息的交换通信。 这个通信可以被配置为加密通信。而数据面由Weave routers之间建立的UDP连接构成，这些连接大部分都会加密。这些连接都是全双工的，并且可以穿越防火墙。

对每一个weave网络中的容器，weave都会创建一个网桥，并且在网桥和每个容器之间创建一个veth pair，一端作为容器网卡加入到容器的网络命名空间中，并为容器网卡配置ip和相应的掩码，一端连接在网桥上，最终通过宿主机上weave router将流量转发到对端主机上。
其基本过程如下：

	1，容器流量通过veth pair到达宿主机上weave router网桥上。
	2，weave router在混杂模式下使用pcap在网桥上截获网络数据包，并排除由内核直接通过网桥转发的数据流量，例如本子网内部、本地容器之间的数据以及宿主机和本地容器之间的流量。捕获的包通过UDP转发到所其他主机的weave router端。
	3，在接收端，weave router通过pcap将包注入到网桥上的接口，通过网桥的上的veth pair，将流量分发到容器的网卡上。

weave默认基于UDP承载容器之间的数据包，并且可以完全自定义整个集群的网络拓扑，但从性能和使用角度来看，还是有比较大的缺陷的：

+ weave自定义容器数据包的封包解包方式，不够通用，传输效率比较低，性能上的损失也比较大。
+ 集群配置比较负载，需要通过weave命令行来手工构建网络拓扑，在大规模集群的情况下，加重了管理员的负担。

Weave优劣势：

+ Weave优势

支持主机间通信加密。

支持container动态加入或者剥离网络。

支持跨主机多子网通信。

+ Weave劣势

只能通过weave launch或者weave connect加入weave网络。


转载自：
https://blog.csdn.net/fsx2550553488/article/details/80474773