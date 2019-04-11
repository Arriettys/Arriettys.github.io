---
title: Docker-Linux内核功能
date: 2019-04-04 09:26:43
tags: Docker
categories: 虚拟化
---
Linux内核能够将root用户的权限分解为称为功能的不同单元。例如，CAP_CHOWN功能允许root用户对文件UID和GID进行任意更改。CAP_DAC_OVERRIDE功能允许root用户绕过文件读取，写入和执行操作的内核权限检查。几乎所有与Linux root用户相关的特殊功能都被分解为单独的功能。
<!--more-->
将root权限功能分解优点：

+ 从root用户帐户中删除单个功能，使其不那么强大/危险。
+ 在非常精细的级别为非root用户添加权限。

功能适用于文件和线程。文件功能允许用户执行具有更高权限的程序。功能适用于文件和线程。文件功能允许用户执行具有更高权限的程序。Linux内核允许你设置功能边界集，对文件/线程可以获得的功能施加限制。

在没有基于文件的功能的环境中，应用程序无法将其权限升级到超出边界集（超出其功能无法增长的集合）。Docker 在启动容器之前设置边界集。您可以使用Docker命令在边界集中添加或删除功能。

默认情况下，Docker 使用白名单方法删除除所需功能之外的所有功能。

## Docker管理功能
从Docker 1.12开始，您有3个高级选项可供使用：

+ 使用大量功能以root身份运行容器，并尝试手动管理容器中的功能。
+ 以有限的功能以root身份运行容器，并且永远不会在容器中更改它们。
+ 以无权限的用户身份运行容器。

选项2是Docker 1.12中最现实的。选项3将是理想的但不现实。应尽可能避免备选方案1。
ps.可以在Docker的未来版本中添加另一个选项，允许你以具有附加功能的非root用户身份运行容器。执行此操作的正确方法需要环境功能，该功能已添加到版本4.3中的Linux内核中。Docker是否有可能在较旧的内核中近似这种行为需要更多的研究。

在以下命令中，$CAP将用于指示一个或多个单独的功能。我们将在下一节中对它们进行测试。
1.从root容器帐户中删除功能。

	$ docker run --rm -it --cap-drop $CAP alpine sh	
	
2.向root容器帐户添加功能。

	$ docker run --rm -it --cap-add $CAP alpine sh
	
3.删除所有功能，然后将单个功能明确添加到root容器帐户。

	$ docker run --rm -it --cap-drop ALL --cap-add $CAP alpine sh

“CAP_”为Linux内核使用所有功能常量添加前缀。例如，CAP_CHOWN，CAP_NET_ADMIN，CAP_SETUID，CAP_SYSADMIN等.Docker功能常量不以“CAP_”为前缀，而是匹配内核的常量。

有关功能的更多信息（包括完整列表），请参阅[功能手册](http://man7.org/linux/man-pages/man7/capabilities.7.html)

## Docker测试
动各种新容器，使用上一步中学习的命令来调整与用于运行容器的帐户相关联的功能。

1.启动一个新容器并证明容器的root帐户可以更改文件的所有权。

```
$ docker run --rm -it alpine chown nobody /
```
该命令不提供表示操作成功的返回码。该命令有效，因为默认行为是使用root用户启动新容器。此root用户默认具有CAP_CHOWN功能。

2.启动另一个新容器并删除除CAP_CHOWN功能之外的容器root帐户的所有功能。请记住，在寻址功能常量时，Docker不使用“CAP_”前缀。

```
$ docker run --rm -it --cap-drop ALL --cap-add CHOWN alpine chown nobody /
```
此命令也不提供返回代码，表示运行成功。操作成功，虽然删除了容器root帐户的所有功能，但又添加了该chown功能。该chown功能是更改文件所有权所需的全部功能。

3.启动另一个新容器，从该root帐户中删除CHOWN功能。

```
$ docker run --rm -it --cap-drop CHOWN alpine chown nobody /
```
![](https://s2.ax1x.com/2019/04/04/AgIxOI.png)
这次该命令返回一个错误代码，表明它失败了。这是因为容器的root帐户没有该CHOWN功能，因此无法更改文件或目录的所有权。

4.创建另一个新容器，并尝试将该CHOWN功能添加到被调用的非root用户nobody。作为同一命令的一部分，尝试并更改文件或文件夹的所有权。

```
$ docker run --rm -it --cap-add chown -u nobody alpine chown nobody /
```
![](https://s2.ax1x.com/2019/04/04/AgolhF.png)
上述命令失败，因为Docker尚不支持向非root用户添加功能。

在此步骤中，你已为一系列新容器添加和删除了功能。可以看到，可以在非常精细的级别上从容器的root用户添加和删除功能。了解到Docker目前不支持向非root用户添加功能。

另外其余部分涉及处理Linux she'll功能，以后再添加，有兴趣的可以看原文

转载自：
https://training.play-with-docker.com/security-capabilities/