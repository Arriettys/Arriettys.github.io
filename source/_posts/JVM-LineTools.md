---
title: JDK-命令行工具
date: 2019-06-02 20:46:18
tags: Java
categories: JVM
---
![](https://s2.ax1x.com/2019/06/03/VYlEkQ.png)
<!--more-->
最近在看JVM的相关书籍，在讲到JDK命令行工具时，笔者按照书上的命令运行，但得带了与书上不同的结果。倒不是说出错了，就是时过境迁，命令行还是那些，但结果参数不是名称换了，就是新加入了参数。在这里笔者再对这些JDK工具实验一下，并结合网上的参考，说明一下命令参数和查询结果的意义

参考自：
《深入理解Java虚拟机  JVM高级特性与最佳实践》
https://www.cnblogs.com/yjd_hycf_space/p/7755633.html

+ 系统：Ubuntu
+ Java版本：openjdk version "1.8.0_212"

笔者介绍的这些工具主要用于监视虚拟机和故障处理的工具，我们可以在java/bin中找到他们

![](https://s2.ax1x.com/2019/06/02/VG4eUS.png)

这些工具大多是jdk/lib/tools.jar类库的一层薄包装，这要功能是在tools类库中实现的。

## jps(JVM Process Status Tool) 
显示指定系统内所有HotSpot虚拟机进程（与操作系统进程一致）

### 命令格式：
** jps [option] [hostid] **
#### option:	
	
	-q：只输出LVMID，省略主类的名称
	-m：输出虚拟机进程启动时传递给主类main()函数的参数
	-l：输出主类的全名，如果进程执行的是Jar包，输出Jar路径
	-v：输出虚拟机进程启动时JVM参数
	
#### hostid：
命令对应的服务器ip，默认不加参数，代码查看本机

![](https://s2.ax1x.com/2019/06/02/VG59aT.png)  
ps. 笔者这里启动了一个XMind的程序，没想到居然是Java写的~~

## jstat(JVM Statistics Monitoring Tool) 
用于监视虚拟机各种运行状态信息的命令行工具，它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据

### 命令格式：
** jstat [option vmid [interval [s|ms] [count]]] **
#### vmid
若是本地虚拟机VMID则与LVMID一致，若是远程虚拟机，VMID格式为：[protocol:][//]lvmid[@hostname[:port]/servername]
#### interval
查询时间间隔 [s|ms]
#### count 
查询次数
#### option

选项	|	作用
--	|	--
-class	|	监视类装载、卸载数量、总空间以及类装载所耗费的时间
-gc	|	监视Java堆状况，包括Eden区、两个survivor区、老年区、永久代等的容量、已用空间、GC时间合计等信息
-gccapacity	|	监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
-gcutil	|	监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
-gccause	|	与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因
-gcnew	|	监视新生代GC状况
-gcnewcapacity	|	监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间
-gcold	|	监视老年代GC状况
-gcoldcapacity	|	监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间
-gcpermcapacity	|	输出永久带使用到的最大、最小空间
-compiler	|	输出JIT编译器编译过的方法、耗时等信息
-printcompilation	|	输出已经被JIT编译的方法

ps.jstat选项众多，这里只列举常用的

下面我将以上图jps命令查询到的VMID来进行实验，读者若想自己动手试试，也可使用jps先查询到一个VMID
**  -class：类加载统计 ** 
![](https://s2.ax1x.com/2019/06/02/VGHUPK.png)

+ loaded：加载class的数量
+ Bytes：所占用空间大小
+ Unloaded：未加载数量
+ Bytes：未加载占用空间
+ Time：时间

** -compiler：编译统计 **
![](https://s2.ax1x.com/2019/06/02/VGbrSU.png)

+ Compiled：编译数量
+ Failed：失败数量
+ Invalid：不可用数量
+ Time：时间
+ FailedType：失败类型
+ FailedMethod：失败方法

** -gc: 垃圾回收统计 **
![](https://s2.ax1x.com/2019/06/02/VGbvff.png)

+ S0C：第一个幸存区的大小
+ S1C：第二个幸存区的大小
+ S0U：第一个幸存区的使用大小
+ S1U：第二个幸存区的使用大小
+ EC：伊甸园区的大小
+ EU：伊甸园区的使用大小
+ OC：老年代大小
+ OU：老年代使用大小
+ MC：方法区大小
+ MU：方法区使用大小
+ CCSC:压缩类空间大小
+ CCSU:压缩类空间使用大小
+ YGC：年轻代垃圾回收次数
+ YGCT：年轻代垃圾回收消耗时间
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

** -gccapacity：堆内存统计 **
![](https://s2.ax1x.com/2019/06/02/VGL8bj.png)

+ NGCMN：新生代最小容量
+ NGCMX：新生代最大容量
+ NGC：当前新生代容量
+ S0C：第一个幸存区大小
+ S1C：第二个幸存区的大小
+ EC：伊甸园区的大小
+ OGCMN：老年代最小容量
+ OGCMX：老年代最大容量
+ OGC：当前老年代大小
+ OC:当前老年代大小
+ MCMN:最小元数据容量
+ MCMX：最大元数据容量
+ MC：当前元数据空间大小
+ CCSMN：最小压缩类空间大小
+ CCSMX：最大压缩类空间大小
+ CCSC：当前压缩类空间大小
+ YGC：年轻代gc次数
+ FGC：老年代GC次数

**-gcnew：新生代垃圾回收统计 **
![](https://s2.ax1x.com/2019/06/02/VGOeL4.png)

+ S0C：第一个幸存区大小
+ S1C：第二个幸存区的大小
+ S0U：第一个幸存区的使用大小
+ S1U：第二个幸存区的使用大小
+ TT:对象在新生代存活的次数
+ MTT:对象在新生代存活的最大次数
+ DSS:期望的幸存区大小
+ EC：伊甸园区的大小
+ EU：伊甸园区的使用大小
+ YGC：年轻代垃圾回收次数
+ YGCT：年轻代垃圾回收消耗时间

** gcnewcapacity：新生代内存统计 **
![](https://s2.ax1x.com/2019/06/02/VGOaTA.png)

+ NGCMN：新生代最小容量
+ NGCMX：新生代最大容量
+ NGC：当前新生代容量
+ S0CMX：最大幸存1区大小
+ S0C：当前幸存1区大小
+ S1CMX：最大幸存2区大小
+ S1C：当前幸存2区大小
+ ECMX：最大伊甸园区大小
+ EC：当前伊甸园区大小
+ YGC：年轻代垃圾回收次数
+ FGC：老年代回收次数

** -gcold：老年代垃圾回收统计 **
![](https://s2.ax1x.com/2019/06/02/VGOfkn.png)

+ MC：方法区大小
+ MU：方法区使用大小
+ CCSC:压缩类空间大小
+ CCSU:压缩类空间使用大小
+ OC：老年代大小
+ OU：老年代使用大小
+ YGC：年轻代垃圾回收次数
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

** -gcoldcapacity：老年代内存统计 **
![](https://s2.ax1x.com/2019/06/02/VGOXkR.png)

+ OGCMN：老年代最小容量
+ OGCMX：老年代最大容量
+ OGC：当前老年代大小
+ OC：老年代大小
+ YGC：年轻代垃圾回收次数
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

** -gcmetacapacity：元数据空间统计 **
![](https://s2.ax1x.com/2019/06/02/VGXVht.png)

+ MCMN: 最小元数据容量
+ MCMX：最大元数据容量
+ MC：当前元数据空间大小
+ CCSMN：最小压缩类空间大小
+ CCSMX：最大压缩类空间大小
+ CCSC：当前压缩类空间大小
+ YGC：年轻代垃圾回收次数
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

** -gcutil：总结垃圾回收统计 **
![](https://s2.ax1x.com/2019/06/02/VGX8Nn.png)

+ S0：幸存1区当前使用比例
+ S1：幸存2区当前使用比例
+ E：伊甸园区使用比例
+ O：老年代使用比例
+ M：元数据区使用比例
+ CCS：压缩使用比例
+ YGC：年轻代垃圾回收次数
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

** -printcompilation：JVM编译方法统计 **
![](https://s2.ax1x.com/2019/06/02/VGX0HJ.png)

+ Compiled：最近编译方法的数量
+ Size：最近编译方法的字节码数量
+ Type：最近编译方法的编译类型。
+ Method：方法名标识。

## jinfo(Configuration Info for Java)
实时查看和 调整虚拟机各项参数。使用jps命令的-v参数可以查看虚拟机启动时显式指定的参数列表，未被显式指定的只能使用jinfo
### 命令格式
** jinfo [option] pid ** 
#### option 
查询的参数名称

## jmap(Memory Map for Java)
生成虚拟机的内存转储快照，获取dump文件。它还可以查询finaize执行队列、Java堆和永久代的详细信息，如空间使用率、当前用的是那种收集器等

### 命令格式
** jmap [option] vmid **

#### option

主要选项	|	作用
---	|	---
-dump	|	生成Java堆转储快照。格式为：-dump:[live, ]format=b, file=<filename>，其中live子参数说明只dump出存活的对象，format设置文件格式，b=binary
-finalizerinfo	|	显示在F-Queue中等待Finalizer线程执行finalize方法的对象
-heap	|	显示Java堆详细信息，如使用哪种回收器、参数配置、分代状况等
-histo	|	显示堆中对象统计信息，包括类、实例数量、合计容量
-permstat	|	以ClassLoader为统计口径显示永久代内存状态
-F	|	当虚拟机进程对-dump选项没有相应时，可使用这个选项强制生成dump快照

![](https://s2.ax1x.com/2019/06/03/VYAfgg.png)

## jhat(Java Heap Dump Browser)
用于分析heapdump文件，内置了微型的HTTP/HTML服务器，可以在浏览器查看分析结果，但一般不会使用，主要原因有二：

+ 一般不会在部署应用程序的服务器上直接分析dump文件，这是一个耗时且消耗硬件资源的过程
+ jhat的分析功能相对简陋

### 命令格式
** jhat filename **

## jstack(Stack Trace for Java) 
用于生成当前时刻线程快照（一般称为threaddump或者javacore文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等原因

### 命令格式
** jstack [option] vmid **
#### option

选项	|	作用
---	|	---
-F	|	当正常输出的请求不被相应时，强制输出线程堆栈
-l	|	除堆栈外，显示关于锁的附加信息
-m	|	如果调用到本地方法的话，可以显示C/C++的堆栈

![](https://s2.ax1x.com/2019/06/03/VYVLX4.png)

## HSDIS插件：JIT生成代码反汇编
Java虚拟机规范详细描述了虚拟机指令集中每条指令的执行过程、执行前后对操作数栈、局部变量表的影响。这些细节描述与早期虚拟机高度吻合，但随着技术的发展，高性能虚拟机真正的细节实现方式已经渐渐与虚拟机规范内容产生越来越大差距，虚拟机规范中的描述逐渐成了虚拟机实现的“概念模型”——即实现只能保证规范描述等效。基于这个原因，我们分析程序的执行语义问题(虚拟机做了什么)时，在字节码层面上分析完全可行，但分析程序的执行行为问题(虚拟机是怎么做的、性能如何)时，在字节码层面上分析就没有意义了，需要通过其他方式解决，H
SDIS插件登场了
它的作用是让HotSpot的-XX:+PrintAssembly指令调用它来把动态生成的本地代码来分析问题。读者可以根据自己操作系统和CPU类型从Project Kenai的网站上下载编译好的插件，直接放在JDK_HOME/jre/bin/client和JDK_HOME/jre/bin/server目录中即可。

