---
title: Hadoop-配置文件
date: 2019-04-17 09:26:36
tags: Hadoop
categories: 分布式集群
---
最近在看Hadoop，实验分布集群的时候经常要配一些配置文件，但通常对这些文件参数都是不都意思的，直接照抄教程，有种死记硬背的赶脚。网上查找了下资料，这边就总结下
<!--more-->
## etc/hadoop/core-site.xml
顾名思义，是全局配置文件

参数	|	属性值	|	解释
------	|	------	|	------
fs.defaultFS	|	NameNode URI	|	hdfs://host:port/
io.file.buffer.size	|	131072		|	SequenceFiles文件中.读写缓存size设定

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.1.100:9000</value>
        <description>192.168.1.100为服务器IP地址，其实也可以使用主机名，不能填localhost</description>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
        <description>该属性值单位为KB，131072KB即为默认的64M</description>
    </property>
</configuration>
```
ps.以上只是列出了常用的几个，完整的参数解释[看这里](https://hadoop.apache.org/docs/r3.1.2/hadoop-project-dist/hadoop-common/core-default.xml)，后面的配置文件也是如此

## etc/hadoop/hdfs-site.xml
对hdfs的局部配置
###配置NameNode

参数	|	属性值	|	解释
------	|	------	|	------
dfs.namenode.name.dir	|	在本地文件系统所在的NameNode的存储空间和持续化处理日志	|	如果这是一个以逗号分隔的目录列表，然后将名称表被复制的所有目录，以备不时之需。
dfs.namenode.hosts/dfs.namenode.hosts.exclude	|	Datanodes permitted/excluded列表		|	如有必要，可以使用这些文件来控制允许 数据节点的列表
dfs.blocksize	|	268435456	|	大型的文件系统HDFS块大小为256MB
dfs.namenode.handler.count	|	100		|	设置更多的namenode线程，处理从 datanode发出的大量RPC请求

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
        <description>分片数量，伪分布式将其配置成1即可</description>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/tmp/namenode</value>
        <description>命名空间和事务在本地文件系统永久存储的路径，ps.file://为开头</description>
    </property>
    <property>
        <name>dfs.namenode.hosts</name>
        <value>datanode1, datanode2</value>
        <description>datanode1, datanode2分别对应DataNode所在服务器主机名</description>
    </property>
    <property>
        <name>dfs.blocksize</name>
        <value>268435456</value>
        <description>大文件系统HDFS块大小为256M，默认值为64M</description>
    </property>
    <property>
        <name>dfs.namenode.handler.count</name>
        <value>100</value>
        <description>更多的NameNode服务器线程处理来自DataNodes的RPCS</description>
    </property>
</configuration>
```

### 配置DataNode

参数	|	属性值	|	解释
------	|	------	|	------
dfs.datanode.data.dir	|	逗号分隔的一个DataNode上，它应该保存它的块的本地文件系统的路径列表	|	如果这是一个以逗号分隔的目录列表，那么数据将被存储在所有命名的目录，通常在不同的设备

```
<configuration>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/tmp/datanode</value>
        <description>DataNode在本地文件系统中存放块的路径</description>
    </property>
</configuration>
```

完整参数[看这里](https://hadoop.apache.org/docs/r3.1.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)

## etc/hadoop/yarn-site.xml
配置yarn，yarn是一种新的 Hadoop 资源管理器，职能就是将资源调度和任务调度分开。资源管理器ResourceManager全局管理所有应用程序计算资源的分配，每一个job的ApplicationMaster负责相应任务的调度和协调。

+ ResourceManager做的事情是负责协调集群上计算资源的分配。调度、启动每一个 Job 所属的 ApplicationMaster、另外监控 ApplicationMaster 的存在情况。
+ NodeManager 功能比较专一，根据要求启动和监视集群中机器的计算容器container。负责 Container 状态的维护，并向 RM 保持心跳汇报该节点资源使用情况。
+ ApplicationMaster 负责一个 Job 生命周期内的所有工作。注意每一个Job都有一个 ApplicationMaster。它和MapReduce任务一样在容器中运行。AM通过与RM交互获取资源，然后然后通过与NM交互，启动计算任务。
+ 容器是由ResourceManager进行统一管理和分配的。有两类container：一类是AM运行需要的container；另一类是AP为执行任务向RM申请的。
### 配置ResourceManager

参数	|	属性值	|	解释
------	|	------	|	------
yarn.resourcemanager.address	|	客户端对ResourceManager主机通过 host:port 提交作业	|	host:port
yarn.resourcemanager.scheduler.address	|	ApplicationMasters 通过ResourceManager主机访问host:port跟踪调度程序获资源	|	host:port
yarn.resourcemanager.resource-tracker.address	|	NodeManagers通过ResourceManager主机访问host:port	|	host:port
yarn.resourcemanager.admin.address	|	管理命令通过ResourceManager主机访问host:port	|	host:port
yarn.resourcemanager.webapp.address	|	ResourceManager web页面host:port	|	host:port
yarn.resourcemanager.scheduler.class	|	ResourceManager 调度类（Scheduler class）	|	CapacityScheduler（推荐），FairScheduler（也推荐），orFifoScheduler
yarn.scheduler.minimum-allocation-mb	|	每个容器内存最低限额分配到的资源管理器要求	|	以MB为单位
yarn.scheduler.maximum-allocation-mb	|	资源管理器分配给每个容器的内存最大限制	|	以MB为单位
yarn.resourcemanager.nodes.include-path/yarn.resourcemanager.nodes.exclude-path	|	NodeManagers的permitted/excluded列表	|	如有必要，可使用这些文件来控制允许NodeManagers列表

```
<configuration>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>192.168.1.100:8081</value>
        <description>IP地址192.168.1.100也可替换为主机名</description>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>192.168.1.100:8082</value>
        <description>IP地址192.168.1.100也可替换为主机名</description>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>192.168.1.100:8083</value>
        <description>IP地址192.168.1.100也可替换为主机名</description>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>192.168.1.100:8084</value>
        <description>IP地址192.168.1.100也可替换为主机名</description>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>192.168.1.100:8085</value>
        <description>IP地址192.168.1.100也可替换为主机名</description>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.class</name>
        <value>FairScheduler</value>
        <description>常用类：CapacityScheduler、FairScheduler、orFifoScheduler</description>
    </property>
    <property>
        <name>yarn.scheduler.minimum</name>
        <value>100</value>
        <description>单位：MB</description>
    </property>
    <property>
        <name>yarn.scheduler.maximum</name>
        <value>256</value>
        <description>单位：MB</description>
    </property>
    <property>
        <name>yarn.resourcemanager.nodes.include-path</name>
        <value>nodeManager1, nodeManager2</value>
        <description>nodeManager1, nodeManager2分别对应服务器主机名</description>
    </property>
</configuration>
```

### 配置NodeManager

参数	|	属性值	|	解释
------	|	------	|	------
yarn.nodemanager.resource.memory-mb	|	givenNodeManager即资源的可用物理内存，以MB为单位	|	定义在节点管理器总的可用资源，以提供给运行容器
yarn.nodemanager.vmem-pmem-ratio	|	最大比率为一些任务的虚拟内存使用量可能会超过物理内存率	|	每个任务的虚拟内存的使用可以通过这个比例超过了物理内存的限制。虚拟内存的使用上的节点管理器任务的总量可以通过这个比率超过其物理内存的使用
yarn.nodemanager.local-dirs	|	数据写入本地文件系统路径的列表用逗号分隔	|	多条存储路径可以提高磁盘的读写速度
yarn.nodemanager.log-dirs	|	本地文件系统日志路径的列表逗号分隔	|	多条存储路径可以提高磁盘的读写速度
yarn.nodemanager.log.retain-seconds	|	10800	|	如果日志聚合被禁用。默认的时间（以秒为单位）保留在节点管理器只适用日志文件
yarn.nodemanager.remote-app-log-dir	|	logs	|	HDFS目录下的应用程序日志移动应用上完成。需要设置相应的权限。仅适用日志聚合功能
yarn.nodemanager.remote-app-log-dir-suffix	|	logs	|	后缀追加到远程日志目录。日志将被汇总到yarn.nodemanager.remote­app­logdir/{user}/${thisParam} 仅适用日志聚合功能。
yarn.nodemanager.aux-services	|	mapreduce-shuffle	|	Shuffle service 需要加以设置的Map Reduce的应用程序服务

```
<configuration>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>256</value>
        <description>单位为MB</description>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-pmem-ratio</name>
        <value>90</value>
        <description>百分比</description>
    </property>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/usr/local/hadoop/tmp/nodemanager</value>
        <description>列表用逗号分隔</description>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>/usr/local/hadoop/tmp/nodemanager/logs</value>
        <description>列表用逗号分隔</description>
    </property>
    <property>
        <name>yarn.nodemanager.log.retain-seconds</name>
        <value>10800</value>
        <description>单位为S</description>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce-shuffle</value>
        <description>Shuffle service 需要加以设置的MapReduce的应用程序服务</description>
    </property>
</configuration>
```
完整参数[看这里](https://hadoop.apache.org/docs/r3.1.2/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)

## etc/hadoop/mapred-site.xml
MapReduce的局部配置，MapReduce是一个分布式的计算框架，核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算框架，并发运行在hadoop集群上。引入MapReduce框架后，开发人员可以将绝大部分的工作集中于业务逻辑上的开发，具体的计算只需要交给框架就可以。
### 配置mapreduce

参数	|	属性值	|	解释
------	|	------	|	------
mapreduce.framework.name	|	yarn	|	执行框架设置为 Hadoop YARN.
mapreduce.map.memory.mb	|	1536	|	对maps更大的资源限制的.
mapreduce.map.java.opts		|	-Xmx2014M	|	maps中对jvm child设置更大的堆大小
mapreduce.reduce.memory.mb	|	3072	|	设置 reduces对于较大的资源限制
mapreduce.reduce.java.opts	|	-Xmx2560M	|	reduces对 jvm child设置更大的堆大小
mapreduce.task.io.sort.mb	|	512	|	更高的内存限制，而对数据进行排序的效率
mapreduce.task.io.sort.factor	|100	|	在文件排序中更多的流合并为一次
mapreduce.reduce.shuffle.parallelcopies	|	50	|	通过reduces从很多的map中读取较多的平行 副本

```
<configuration>
    <property>
        <name> mapreduce.framework.name</name>
        <value>yarn</value>
        <description>执行框架设置为Hadoop YARN</description>
    </property>
    <property>
        <name>mapreduce.map.memory.mb</name>
        <value>1536</value>
        <description>对maps更大的资源限制的</description>
    </property>
    <property>
        <name>mapreduce.map.java.opts</name>
        <value>-Xmx2014M</value>
        <description>maps中对jvm child设置更大的堆大小</description>
    </property>
    <property>
        <name>mapreduce.reduce.memory.mb</name>
        <value>3072</value>
        <description>设置 reduces对于较大的资源限制</description>
    </property>
    <property>
        <name>mapreduce.reduce.java.opts</name>
        <value>-Xmx2560M</value>
        <description>reduces对 jvm child设置更大的堆大小</description>
    </property>
    <property>
        <name>mapreduce.task.io.sort</name>
        <value>512</value>
        <description>更高的内存限制，而对数据进行排序的效率</description>
    </property>
    <property>
        <name>mapreduce.task.io.sort.factor</name>
        <value>100</value>
        <description>在文件排序中更多的流合并为一次</description>
    </property>
    <property>
        <name>mapreduce.reduce.shuffle.parallelcopies</name>
        <value>50</value>
        <description>通过reduces从很多的map中读取较多的平行副本</description>
    </property>
</configuration>
```

### 配置mapreduce的JobHistory服务器

参数	|	属性值	|	解释
------	|	------	|	------
maprecude.jobhistory.address	|	MapReduce JobHistory Server host:port	|	默认端口号 10020
mapreduce.jobhistory.webapp.address	|	MapReduce JobHistory Server Web UIhost:port	|	默认端口号 19888
mapreduce.jobhistory.intermediate-done-dir	|	/mr­history/tmp	|	在历史文件被写入由MapReduce作业
mapreduce.jobhistory.done-dir	|	/mr­history/done	|	目录中的历史文件是由MR JobHistory Server管理

```
<configuration>
    <property>
        <name> mapreduce.jobhistory.address</name>
        <value>192.168.1.100:10200</value>
        <description>IP地址192.168.1.100可替换为主机名</description>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>192.168.1.100:19888</value>
        <description>IP地址192.168.1.100可替换为主机名</description>
    </property>
    <property>
        <name>mapreduce.jobhistory.intermediate-done-dir</name>
        <value>/usr/local/hadoop/mr­history/tmp</value>
        <description>在历史文件被写入由MapReduce作业</description>
    </property>
    <property>
        <name>mapreduce.jobhistory.done-dir</name>
        <value>/usr/local/hadoop/mr­history/done</value>
        <description>目录中的历史文件是由MR JobHistoryServer管理</description>
    </property>
</configuration>
```
完整参数[看这里](https://hadoop.apache.org/docs/r3.1.2/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)
