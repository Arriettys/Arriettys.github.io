---
title: 使用Hexo+Github搭建免费个人网站
date: 2019-03-10 16:38:07
tags:
 - Hexo
 - Node.js
 - Github
categories: Web
---
## 1.写在前面的
个人之前是使用Tomcat+VPS尝试搭博客的，但由于过于死脑筋，前后端都是自己手敲的，结果就是前端和后台都会但都不够深入，框架也只是懂个皮毛用法，别人一问就不知道说明，真是够浪费时间的。所以这次就直接用Hexo模板建博客，这篇文章则是总结下自己的搭建过程，后续也会更新一些新玩意儿。
话不多说，这就让我们开始吧
使用github page服务搭建博客的好处有：
<!--more-->

		1. 全是静态文件，访问速度快；
		2. 不需要租服务器，不需要后台；
		3. 可以绑定自己的域名，不仔细看的话根本看不出来你的网站是基于github的，当然域名是需要花钱滴~
		4. 数据绝对安全，基于github的版本管理，想恢复到哪个历史版本都行；
		5. 博客内容可以轻松打包、转移、发布到其它平台；
## 2. 准备环境
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
然而安装 Hexo 前，你必须安装下列应用程序：
Node.js
Git
本人使用的是linux系统(退出Windows很多年了)，所以一下教程是基于Linux的，windows的朋友可以[看这里](https://www.simon96.online/2018/11/10/hexo-env/ )
### 2.1 Node.js
Node.js安装包及源码下载地址为：https://nodejs.org/en/download/

![](https://s2.ax1x.com/2019/03/10/A9iSIO.png)
Source code为源码是需要手动安装滴~，不建议安装源码，会有许多的问题，例如c++版本过低等，毕竟我们只是使用者，方便就好节省时间。
** binaries为编译版本，解压即用，可理解为免安装硬盘版（ps.注意ARM为嵌入式版本，PC选×64）网页下载或wget命令下载皆可。

	wget https://nodejs.org/dist/v10.15.3/node-v10.15.3-linux-x64.tar.xz 	//下载
	tar xf node-v10.15.3-linux-x64.tar.xz 	//解压
	mv node-v10.15.3-linux-x64/ /usr/local/ 	//移动
	//使用ln设置软连接
	ln -s /usr/local/node-v10.15.3-linux-x64/bin/npm	/usr/local/bin/
	ln -s /usr/local/node-v10.15.3-linux-x64/bin/node 	/usr/local/bin/
	//这样npm与node就可以全局使用了，例如查看版本
	node -v

### 2.2 Git
Git是目前世界上最先进的分布式版本控制系统（没有之一）
借助它我们就可以随时随地将我们的博客文章同步到Github上或版本回退
大部分的linux上都是自带了Git，这里只是简单讲下使用系统提供的包管理工具安装

安装前需确定是否安装了Git的依赖包：
curl，zlib，openssl，expat，libiconv
在有yum的系统（比如Fedora）：

	yum install git-core
在apt-get 的系统上（比如 Debian 体系）:
	
	apt-get install git
### 2.3 Hexo
安装Hexo，在命令行运行：
	
	npm install -g hexo-cli
npm就是之前node.js解压包中自带的，我们通过ln建立软连接所以直接使用
ps.这里建议在/usr/local/node-v10.15.3-linux-x64/bin/目录中运行命令，之后目录中hexo文件，之后我们有许多命令要用到它，这里也用ln建立软连接

	ln -s /usr/local/node-v10.15.3-linux-x64/bin/hexo	/usr/local/bin/
至此hexo的环境就完成了
#### 2.3.1 初始化Hexo
自己选好一个存放Hexo初始化文件的文件夹，如/home/HexoWeb
在文件夹中运行：

	hexo init
	npm install

完成后，站点目录：
	
		.
		├── _config.yml
		├── package.json
		├── scaffolds
		├── source
		|   ├── _drafts
		|   └── _posts
		└── themes
hexo相关命令都在站点目录下运行
OK，现在让我们启动服务器：

	hexo server 	||	hexo s
浏览器访问：http://localhost:4000/		不出意外的话就是hexo默认主题
![](https://s2.ax1x.com/2019/03/10/A9mfsg.png)
在已启动服务器的命令行下执行快捷键Ctrl + c，即可关闭服务器
ps.右下角的妹纸可不是默认主题的，后面我会交代怎样设置
## 3. 搭建github博客
现在你的电脑已经具备本地服务器的能力了，现在我们要借助github充当我们的服务器
### 3.1 创建github仓库
新建一个仓库，Repository name必须是username.github.io,其中username是你的用户名，顺带初始化README.md文件。仓库创建成功不会立即生效，需要过一段时间，大概10-30分钟，或者更久。之后浏览器访问https://username.github.io就会显示README.md文件的内容。以后你网站所有的代码都放这。
### 3.2 绑定域名
域名的购买我就不在这里提了，不过建议买国外的，国内的既要实名认证还有提交审核等个好几天。我是在dynadot购买的，.top一年5美刀~。
随后进入manage domains 点击DNS Settings管理DNS
![](https://s2.ax1x.com/2019/03/10/A9KGIs.png)
添加CNAME或A解析记录
![](https://s2.ax1x.com/2019/03/10/A9KUzV.png)
CNAME类型只需填写username.github.io
A类型则需要IP，IP可通过ping username.github.io得到
接着在github仓库中添加CNAME文件，其中填入域名。或者在仓库的settings中设置Custom Domain为域名，github会自动添加CNAME文件
![](https://s2.ax1x.com/2019/03/10/A9Qygx.png)
![](https://s2.ax1x.com/2019/03/10/A9QfVe.png)
ps.这里有个问题，设置Custom Domain完成点击save后，下面的选项会有警告，这是https带来的问题下面我会解决
### 3.3 通过HTTPS访问域名
在完成上述操作以后，只能通过HTTP协议传输(明文传输),于是在通过域名访问自己的username.github.io时，发现浏览器提示该网址不安全，没有合格的安全证书，不能通过https(密文传输)访问。
#### 3.3.1 HTTP与HTTPS
HTTP是明文传输协议，传输内容容易被嗅探和篡改。 
而HTTPS，即HTTP over SSL/TLS,是添加了一层SSL(Secure Sockets Layer，安全套接层)，或者是TLS(Transport Layer Security,传输层安全协议)，所以HTTPS就可以视为HTTP和SSL/TLS协议的组合。 
HTTPS能做到良好的保密性(防嗅探)，真实性(防篡改)，完整性(防域名劫持和域名欺骗)。
#### 3.3.2 申请SSL证书
SSL证书由你的NS(Name Server，域名服务商)颁发，像阿里云这样的通常是需要钱滴~，所以我们最好迁移到免费提供SSL的NS处，比如国内的DNSpod(国内都需要备案),还有国外的Netlify和Cloudflare，从速度和操作性考虑，本人选择了Cloudflare。
1. 到[Cloudflare官网](https://www.cloudflare.com/)注册
2. 根据指引点击Add Site，添加你的域名，会自动开始扫描DNS解析记录
3. 选择免费的那个计划
4. 扫描完成后，Cloudflare会选择给我们分配两个NS地址，将这两个地址替换原NS地址，等待生效；
至此就可以使用域名访问仓库了
## 4. 从本地到远程
### 4.1 配置SSH key
提交代码到github仓库中需要有github的权限，但直接使用账号密码则不安全，所以我们使用SSH Key来解决本地和github的连接问题	
ps.第一次使用git连接的，还需要配置用户账号账号

	git config --global user.name "liuxianan"// 你的github用户名，非昵称
	git config --global user.email  "xxx@qq.com"// 填写你的github注册邮箱

我们先检查本机是否已存在ssh密钥

	cd ~/.ssh
	
如果提示：No such file or directory 说明没有，则我们自己生成

	ssh-keygen -t rsa -C "你的密钥"
	
然后连续3次回车，最终会生成一个文件在用户目录下，打开用户目录，找到.ssh\id_rsa.pub文件，记事本打开并复制里面的内容，打开你的github主页，进入个人设置 -> SSH and GPG keys -> New SSH key：
![](https://s2.ax1x.com/2019/03/10/A9tLNT.png)
测试一下
	
	ssh -T git@github.com 
	
第一次连接会提示 Are you sure you want to continue connecting (yes/no)?，输入yes，成功会看到

	Hi Arriettys! You've successfully authenticated, but GitHub does not provide shell access.

### 4.1 站点配置
修改_config.yml（在站点目录下）。文件末尾修改为：

	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
  	type: git
  	repo: git@github.com:username/username.github.io.git
  	bransh: master
  	
  注意：上面仓库地址写ssh地址，不写http地址。
  推送命令：
  
  	hexo g
  	hexo d
  返回INFO Deploy done: git即成功推送
  等待1分钟左右，浏览器访问网址： https://username.github.io或域名
  

	





