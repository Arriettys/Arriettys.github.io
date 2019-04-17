---
title: Hadoop-分布式模式
date: 2019-04-16 11:24:07
tags: Hadoop
categories: 分布式集群
---
最近参照网上的教程搭建Hadoop集群，在自己动手操作时出现了一些教程上没有出现的错误，头痛了好几天，索性得遇大神指点才找出问题所在，这里分享我的历程
<!--more-->
以下教程参考自：
https://www.cnblogs.com/zhangyinhua/p/7647686.html
https://www.yiibai.com/hadoop/hadoop_enviornment_setup.html

# Hadoop的三种运行模式（启动模式）
运行的前提是Hadoop环境配置完毕，这里就不多说，想了解的[看这里](https://www.yiibai.com/hadoop/hadoop_enviornment_setup.html)
## 单机模式（独立模式）（Local或Standalone  Mode）
单机模式其实很简单，没出什么问题，我这里就简单介绍下。

+ 默认情况下，Hadoop即处于该模式，用于开发和调式。
+ 不对配置文件进行修改。
+ 使用本地文件系统，而不是分布式文件系统。
+ Hadoop不会启动NameNode、DataNode、JobTracker、TaskTracker等守护进程，Map()和Reduce()任务作为同一个进程的不同部分来执行的。
+ 用于对MapReduce程序的逻辑进行调试，确保程序的正确。

## 伪分布式模式（Pseudo-Distrubuted Mode）
**重点来了**

+ Hadoop的守护进程运行在本机机器，模拟一个小规模的集群　
+ 在一台主机模拟多主机。
+ Hadoop启动NameNode、DataNode、JobTracker、TaskTracker这些守护进程都在同一台机器上运行，是相互独立的Java进程。
+ 在这种模式下，Hadoop使用的是分布式文件系统，各个作业也是由JobTraker服务，来管理的独立进程。在单机模式之上增加了代码调试功能，允许检查内存使用情况，HDFS输入输出，以及其他的守护进程交互。类似于完全分布式模式，因此，这种模式常用来开发测试Hadoop程序的执行是否正确。
+ 修改3个配置文件：core-site.xml（Hadoop集群的特性，作用于全部进程及客户端）、hdfs-site.xml（配置HDFS集群的工作属性）、mapred-site.xml（配置MapReduce集群的属性）
+ 格式化文件系统

### 配置文件
这里我们之为实验需求而设置，具体配置文件作用，参数意义不多说，想看详细解析的，来{% post_link Hadoop-ConfigurationFile%}
这些文件都在hadoop-3.1.2/etc/hadoop/中

#### hadoop-env.sh

	export JAVA_HOME=${JAVA_HOME}改成export JAVA_HOME=/opt/jdk
改为你电脑上Java的局对路径
![](https://s2.ax1x.com/2019/04/17/Az92Y6.png)
ps.在配置文件中有提示我们怎么设置，我们一般不删除，而是选择注释它

#### core-site.xml　

```
<configuration>
                <property>
                    <name>fs.defaultFS</name>
                    <value>hdfs://1.0.0.5:9000</value>   
                </property>
 </configuration>
```
1.0.0.5是你主节点所在主机的ip，而9000为端口
注意，第一个问题来了，如果你是使用云服务器做实验，碰巧用的还是阿里云，那你会在最后测试时出现9000端口拒绝服务但是通过natstat -nplt 并没有看到9000有被占用的情况。

解决：阿里云服务器有公网ip，和私网ip，两个节点之间的通信通过公网ip进行，但是开启namenode，9000要用内网ip
在core-site.xml的配里应如下设定：

```
<property>
                <name>fs.defaultFS</name>
                <value>hdfs://内网ip:9000</value>
</property>
```
同理 hdfs-site.xml的配置中对于50070或9870端口也要用内网IP。
ps.若你后面要更改，一定要全部重启，且重新对HDFS集群进行格式化，否则还是会拒绝连接
参考自：
https://blog.csdn.net/weixin_43322061/article/details/84194325

#### hdfs-site.xml　　

创建相关目录

```
$ sudo mkdir -p /data/hadoop/hdfs/nn
$ sudo mkdir -p /data/hadoop/hdfs/dn
$ sudo mkdir -p /data/hadoop/hdfs/snn
$ sudo mkdir -p /data/hadoop/yarn/nm
```
ps.如果使用sudo启动hadoop的相关进程，这几目录的权限可以不用管。如果是使用当前的用户启动相关进程，对于opt目录，当前用户得有读写权限，对于/data目录也需要读写权限。

```
<configuration>
                <property>
                    <name>dfs.nameservices</name>
                    <value>hadoop-cluster</value>
                </property>
                <property>
                    <name>dfs.namenode.name.dir</name>
                    <value>file:///data/hadoop/hdfs/nn</value>
                </property>
                <property>
                    <name>dfs.namenode.ch
                    eckpoint.dir</name>
                    <value>file:///data/hadoop/hdfs/snn</value>
                </property>
                <property>
                    <name>dfs.namenode.checkpoint.edits.dir</name>
                    <value>file:///data/hadoop/hdfs/snn</value>
                </property>
                <property>
                    <name>dfs.datanode.data.dir</name>
                    <value>file:///data/hadoop/hdfs/dn</value>
                </property>
</configuration>
```

#### mapred-site.xml
如果你使用的版本没有这个文件，有一个mapred-site.xml.template文件，将该文件复制一份为mapred-site.xml

	$ cp mapred-site.xml.template mapred-site.xml
	
```
<configuration>
                <property>
                    <name>mapreduce.framework.name</name>
                    <value>yarn</value>
                </property>
</configuration>
```

#### yarn-site.xml

```
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
```
### 启动集群
1.对HDFS集群进行格式化，HDFS集群是用来存储数据的。

	hdfs namenode -format
2.启动HDFS集群　

	hadoop-daemon.sh start namenode 启动主节点
　　hadoop-daemon.sh start datanode 启动从节点
ps.好像Hadoop3Hadoop-daemon命令被hdfs,yarn等取代了，3以上版本用下面的命令

	hdfs --daemon start namenode
	hdfs --daemon start datanode
3.启动YARN集群

	yarn-daemon.sh start resourcemanager
　　yarn-daemon.sh start nodemanager
3以上版本：

	yarn --daemon start resourcemanager
	yarn --daemon start nodemanager
4.启动作业历史服务器

	mr-jobhistory-daemon.sh start historyserver
3以上版本：
	
	mapred --daemon start historyserver
5.jps命令查看是否启动成功
![](https://s2.ax1x.com/2019/04/17/AzieoR.png)

其实使用命令可以一起启动的

	$ start-all.sh
但我使用时，使用jps查看只启动了Jps和ResourceManager两项，具体原因我也不清楚，只好老老实实一条条启动

关闭合集（重启时先关闭再启动）：

```
hadoop-daemon.sh stop namenode 
hadoop-daemon.sh stop datanode 
yarn-daemon.sh stop resourcemanager
yarn-daemon.sh stop nodemanager
mr-jobhistory-daemon.sh stop historyserver
||
hdfs --daemon stop namenode
hdfs --daemon stop datanode
yarn --daemon stop resourcemanager
yarn --daemon stop nodemanager
mapred --daemon stop historyserver
```
6.HDFS集群的简单操作命令

	hdfs dfs -ls /
如果出现9000拒绝连接，且你使用的是阿里云服务器
![](https://s2.ax1x.com/2019/04/17/AzkaM8.png)
可能是你core-site.xml设置的是外网ip，改为内网ip，使用ifconfig查看内网ip

### WEB监控页面
HDFS：http://ip:50070
ps.3版本默认端口由50070改为9870
![](https://s2.ax1x.com/2019/04/17/AzAJw4.png)
如果拒绝访问，则可以在hdfs-site.xml中，更改开放端口的绑定IP：

```
<property>
                <name>dfs.http.address</name>
                <value>ip:50070</value> 
                <!--<value>ip:9870</value>-->
 </property>
```
ps.阿里云服务器的ip需为内网ip


YARN：http://ip:8088
![](https://s2.ax1x.com/2019/04/17/AzAy0e.png)


