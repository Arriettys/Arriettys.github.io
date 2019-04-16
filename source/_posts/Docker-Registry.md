---
title: Docker-私有仓库
date: 2019-04-13 10:29:16
tags: Docker
categories: 虚拟化
---
Docke官方提供了Docker Hub网站来作为一个公开的集中仓库。然而，本地访问Docker Hub速度往往很慢，并且很多时候我们需要一个本地的私有仓库只供网内使用。
Docker仓库实际上提供两方面的功能，一个是镜像管理，一个是认证。前者主要由docker-registry项目来实现，通过http服务来上传下载；后者可以通过docker-index（闭源）项目或者利用现成认证方案（如nginx）实现http请求管理。
<!--more-->
## 安装docker-registry

	$ docker run -d -p 5000:5000 --restart=always --name registry -v /opt/registry:/var/lib/registry registry:2
将本机/opt/registry挂载到容器的/var/lib/registry作为存放images地址
## 上传镜像
查看系统已有的镜像：

	$ docker images
使用docker tag将centos镜像打个标记，建议格式：网卡IP地址:端口/image

	$ docker tag imageID 172.19.126.97:5000/centos
网卡ip地址可使用ifconfig eth0查看

使用docker push 上传标记的镜像
没有成功，这是因为从docker1.3.2版本开始，使用registry时，必须使用TLS保证其安全。
![](https://s2.ax1x.com/2019/04/13/ALFFmt.png)
在/etc/docker/目录下，创建daemon.json文件。在文件中写入：

	{ "insecure-registries":["172.19.126.97:5000"] }
ps.写自己的ip，别照抄啊！！

然后重启docker：

	$ systemctl restart docker
重新上传：
![](https://s2.ax1x.com/2019/04/13/ALFytO.png)
主机上就可以查看到上传的镜像

	ls /opt/registry/docker/registry/v2/repositories/

## 配置SSL证书及nginx反向代理docker registry
#### 搭建私有CA
初始化CA环境，在/etc/pki/CA/下建立证书索引数据库文件index.txt和序列号文件serial，并为证书序列号文件提供初始值。

	$ touch /etc/pki/CA/{index.txt,serial}
	$ echo 01 > /etc/pki/CA/serial
生成密钥并保存到/etc/pki/CA/private/cakey.pem
	
	$ (umask 077;openssl genrsa -out  /etc/pki/CA/private/cakey.pem 2048)
生成根证书

	$ openssl req -new -x509 -key  /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
接下来会让你填写以下信息：

	Country Name (2 letter code) [XX]:CN
	State or Province Name (full name) []:China
	Locality Name (eg, city) [Default City]:Beijing
	Organization Name (eg, company) [Default Company Ltd]:wts	//后面三个我也不知道怎么回事
	Organizational Unit Name (eg, section) []:sysops
	Common Name (eg, your name or your server's hostname) []:hub.wts.com
	Email Address []:admin@wts.com		//可以改成你自己的邮箱
使系统信任根证书

	$ cat /etc/pki/CA/cacert.pem >> /etc/pki/tls/certs/ca-bundle.crt

额...，突然涉及CA证书，有点晕，待搞清楚CA再更

转载自：
https://www.cnblogs.com/Eivll0m/p/7089675.html