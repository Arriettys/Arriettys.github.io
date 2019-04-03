---
title: Docker-安全性
date: 2019-04-03 08:22:52
tags: Docker
categories: 虚拟化
---
**实验环境：Seccomp**
seccomp是Linux内核中的沙盒工具，其作用类似于系统调用的防火墙（系统调用）。它使用Berkeley Packet Filter（BPF）规则来过滤系统调用并控制它们的处理方式。这些过滤器可以显著限制容器访问Docker Host的Linux内核 - 特别是对于简单的容器/应用程序。
<!--more-->
**先决条件**
只有在使用 seccomp 构建 Docker 并且内核配置了 CONFIG_SECCOMP 的情况下，此功能才可用。要检查你的内核是否支持 seccomp：

```
$ cat /boot/config-`uname -r` | grep CONFIG_SECCOMP=

CONFIG_SECCOMP=y
```
**为容器传递配置文件**
默认的 seccomp 配置文件为使用 seccomp 运行容器提供了一个合理的设置，并禁用了大约 44 个超过 300+ 的系统调用。它具有适度的保护性，同时提供广泛的应用兼容性。默认的 Docker 配置文件可以在 [这里 ](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)找到

该配置文件是白名单，默认情况下阻止访问所有的系统调用，然后将特定的系统调用列入白名单。

seccomp 有助于以最小权限运行 Docker 容器。不建议更改默认的 seccomp 配置文件，但你可以通过将--security-opt标志传递给docker run命令来覆盖它

以下示例命令基于Alpine映像启动交互式容器并启动shell进程。它还应用了描述的seccomp配置文件<profile>.json
```
 $ docker run -it --rm --security-opt seccomp=<profile>.json alpine sh ...
```

Docker支持许多与安全相关的技术。其他安全相关技术可能会干扰对seccomp配置文件的测试。因此，测试seccomp配置文件效果的最佳方法是添加所有功能并禁用apparmor（一个高效和易于使用的Linux系统安全应用程序）。

以下docker run命令添加所有功能并禁用apparmor的参数 : --cap-add ALL --security-opt apparmor=unconfined.

## 克隆实验室GitHub repo
clone 实验室的GitHub仓库，以便你拥有将用于本实验其余部分的seccomp配置文件

```
$ git clone https://github.com/docker/labs
```
切换到labs/security/seccomp目录

```
$ cd labs/security/seccomp
```
再次目录中，docker run在整个实验中引用各种命令的seccomp配置文件

## 测试seccomp配置文件

此步骤中，将使用seccomp-profiles/deny.json包含的seccomp配置文件。此配置文件具有空的系统调用白名单，这意味着将阻止所有系统调用。作为演示的一部分，将添加所有功能并有效地禁用apparmor，以便你知道只有你的seccomp配置文件阻止了系统调用。

使用该docker run命令尝试启动一个新容器，其中添加了所有功能，apparmor unconfined，并应用了seccomp-profiles/deny.jsonseccomp配置文件。

```
$ docker run --rm -it --cap-add ALL --security-opt apparmor=unconfined --security-opt seccomp=seccomp-profiles/deny.json alpine sh
```
![](https://s2.ax1x.com/2019/04/03/AcNvXq.png)
在这种情况下，Docker实际上没有足够的系统调用来启动容器！

## 有选择地删除系统调用
在此步骤中，你将看到如何对default.json配置文件应用更改是一种很好的方法，可以微调容器可用的系统调用。
对default.json概要文件的修改，其中chmod（）、fchmod（）和chmodat（）系统调用从其白名单中删除，得到default-no-chmod.json。

使用default-no-chmod.json配置文件启动新容器并尝试运行该chmod 777 / -v命令

```
$ docker run --rm -it --security-opt seccomp=./seccomp-profiles/default-no-chmod.json alpine sh
```
然后从容器内部：

```
$ chmod 777 / -v
```
![](https://s2.ax1x.com/2019/04/03/AcUcD0.png)
因为chmod 777 / -v命令使用了一些chmod()，fchmod()和chmodat()已经从的白名单中删除的系统调用所以失败。

退出容器，使用default.json配置文件启动另一个新容器并运行相同的容器chmod 777 / -v
![](https://s2.ax1x.com/2019/04/03/AcUvPe.png)
该命令成功

检查两个配置文件的存在chmod()，fchmod()以及chmodat()系统调用
```
$ cat ./seccomp-profiles/default.json | grep chmod
$ cat ./seccomp-profiles/default-no-chmod.json | grep chmod
```

## 编写seccomp配置文件
可以从头开始编写Docker seccomp配置文件，也可以编辑现有配置文件。
默认的Docker seccomp配置文件的布局如下所示：

```
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "architectures": [
        "SCMP_ARCH_X86_64",
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
    ],
    "syscalls": [
        {
            "name": "accept",
            "action": "SCMP_ACT_ALLOW",
            "args": []
        },
        {
            "name": "accept4",
            "action": "SCMP_ACT_ALLOW",
            "args": []
        },
        ...
    ]
}
```
其中name是系统调用的名称，action是发生系统调用时seccomp的操作，args是系统调用的参数限制条件。

seccomp profile包含3个部分:
1. 默认操作(default Action)
2. 系统调用所支持的Linux架构(architectures)
3. 系统调用具体规则(syscalls)

在seccomp profile规则中，可定义以下5种行为来对进程的系统调用行为做出响应，更高的行动否决了较低的行动

	SCMP_ACT_KILL		当进程调用某系统调用，内核会发出SIGSYS信号终止该进程，该进程不会接收到这个信号
	SCMP_ACT_TRAP		当进程调用某系统调用，该进程会接收到SIGSYS信号，并根据该信号改变自身的行为。
	SCMP_ACT_ERRNO	当进程调用某系统调用，系统调用失败，进程会接收到返回值，该返回值与Linux内核的errno对应。
	SCMP_ACT_TRACE		当进程调用某系统调用，进程会被跟踪。
	SCMP_ACT_ALLOW	进程系统调用被允许。

配置文件可以包含基于系统调用参数值的更精细的过滤器。
ps.这里的知识点涉及linux系统调用知识，建议跳过，有基础了再来看

```
{
    ...
    "syscalls": [
        {
            "name": "accept",
            "action": "SCMP_ACT_ALLOW",
            "args": [
                {
                    "index": 0,
                    "op": "SCMP_CMP_MASKED_EQ",
                    "value": 2080505856,
                    "valueTwo": 0
                }
            ]
        }
    ]
}
```

+ index 是系统调用参数的索引
+ op是对参数执行的操作。参数列表：
	+ SCMP_CMP_NE - not equal
	+ SCMP_CMP_LT - less than
	+ SCMP_CMP_LE - less than or equal to
	+ SCMP_CMP_EQ - equal to
	+ SCMP_CMP_GE - greater than
	+ SCMP_CMP_GT - greater or equal to
	+ SCMP_CMP_MASKED_EQ - masked equal: true if (value & arg == valueTwo)
+ value 是操作的参数
+ 仅用于SCMP_cmp_masked_eq

如何编写Docker seccomp配置文件的最权威的来源是用于反序列化JSON的结构。

+ https://github.com/docker/engine-api/blob/c15549e10​​366236b069e50ef26562fb24f5911d4/types/seccomp.go
+ https://github.com/opencontainers/runtime-spec/blob/6be516e2237a6dd377408e455ac8b41faf48bdf6/specs-go/config.go#L502

参考自：
https://blog.csdn.net/fy_long/article/details/88426053
https://training.play-with-docker.com/security-seccomp/
