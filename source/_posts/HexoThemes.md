---
title: Hexo主题优化
date: 2019-03-11 09:57:15
tags:
 - Hexo
 - Themes
 - NexT
categories: Web
---
## 1. 选择主题
Hexo默认的主题为landscape，其他主题可在一下网站浏览选择
https://github.com/search?q=hexo-theme
https://hexo.io/themes/
我使用的是NexT，一下的教程以NexT为例
<!--more-->
## 2. 安装NexT
在Hexo中有两份主要的配置文件，名称都是 _config.yml。一份在站点根目录下，负责Hexo本身的配置。一份在主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件
### 2.1 下载主题
建议你使用 Git克隆最新版本 的方式，之后的更新可以通过 git pull 来快速更新， 而不用再次下载压缩包替换。在终端窗口下，定位到Hexo站点下themes目录下

	cd your-hexo-site/themes
	git clone https://github.com/iissnan/hexo-theme-next ./next

这是themes目录下除了landscape，还会多一个next
### 2.2 启用主题
与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 站点配置文件， 找到 theme 字段，并将其值更改为 next。 

	theme: next
	
![](https://s2.ax1x.com/2019/03/11/A9oCpF.png)

### 2.3 选择Scheme
NexT提供了四种外观
Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
![](https://s2.ax1x.com/2019/03/11/A9oW3F.png)
Mist - Muse 的紧凑版本，整洁有序的单栏外观
![](https://s2.ax1x.com/2019/03/11/A9oHN6.png)
Pisces - 双栏 Scheme，小家碧玉似的清新
![](https://s2.ax1x.com/2019/03/11/A9Tk8S.png)
Gmini-双栏 Scheme，内容区较Pisces大
![](https://s2.ax1x.com/2019/03/11/A9TOI0.png)

Scheme的切换通过修改 主题配置文件
![](https://s2.ax1x.com/2019/03/11/A97glF.png)

用那种就把前面的#去掉

### 2.3 菜单栏设置
编辑 主题配置文件，设定菜单内容，对应的字段是 menu。

	menu:
 		 home: /
  		archives: /archives
 		 #about: /about
 		 #categories: /categories
  		tags: /tags
  		#commonweal: /404.html
  		
以上为NexT默认菜单项，同样的，用哪个就去掉前面的#
NexT 默认的菜单项有（标注！的项表示需要手动创建这个页面）

键值	|	设定值	|	显示文本
-	|	-	|	-
home	|	home: /	|	主页
archives	|	archives: /archives	|	归档页
categories	|	categories: /categories	|	分类页！
tags	|	tags: /tags	|	标签页 !
about	|	about: /about	|	关于页面 ！
commonweal	|	commonweal: /404.html	|	公益 404！

### 2.4 设置菜单项的显示文本
在上一步中设置的菜单的名称并不直接用于界面上的展示。Hexo 在生成的时候将使用 这个名称查找对应的语言翻译，并提取显示文本。这些翻译文本放置在 NexT 主题目录下的 languages/{language}.yml （{language} 为你所使用的语言）
以简体中文为例，若你需要添加一个菜单项，比如 something。那么就需要修改简体中文对应的翻译文件 languages/zh-Hans.yml，在 menu 字段下添加一项：

	menu:
  		home: 首页
  		archives: 归档
  		categories: 分类
  		tags: 标签
  		about: 关于
  		search: 搜索
  		commonweal: 公益404
  		something: 有料

### 2.5 设定菜单项的图标
对应的字段就在menu下，为menu_icons。将enable填入true即可

### 2.6 设置侧栏
默认情况下，侧栏仅在文章页面（拥有目录列表）时才显示，并放置于右侧位置。 可以通过修改 主题配置文件 中的 sidebar 字段来控制侧栏的行为。侧栏的设置包括两个部分，其一是侧栏的位置， 其二是侧栏显示的时机
#### 1.设置侧栏的位置，修改 sidebar.position 的值，支持的选项有：
left - 靠左放置
right - 靠右放置

	sidebar:
  		position: left

#### 2. 设置侧栏显示的时机，修改 sidebar.display 的值，支持的选项有：
post - 默认行为，在文章页面（拥有目录列表）时显示
always - 在所有页面中都显示
hide - 在所有页面中都隐藏（可以手动展开）
remove - 完全移除

	sidebar:
	  	display: post
### 2.7 设置头像
编辑 主题配置文件， 修改字段 avatar， 值设置成头像的链接地址。其中，头像的链接地址可以是：
互联网URI：http://example.com/avatar.png

站点内的地址：
将头像放置主题目录下的 source/uploads/ （新建 uploads 目录若不存在） 
配置为：avatar: /uploads/avatar.png
或者 放置在 source/images/ 目录下 
配置为：avatar: /images/avatar.png
个人建议图片全部放在网上，现在免费图床很多随便选个

### 2.8 添加背景图
在 themes/*/source/css/_custom/custom.styl 中添加如下代码：

	body
	{
    		background:url(/images/bg.jpg) || url(http://example.com/bg.hpg);
	   	background-size:cover;
    		background-repeat:no-repeat;
    		background-attachment:fixed;
    		background-position:center;
	}
	
### 2.9 修改字体
互联网上的字体直接使用URI，自己下载的则放在 themes/*/source/font、
在 themes/*/source/css/_custom/custom.styl 中添加如下代码：

	@font-face 
	{
    		font-family: Zitiming;
    		src: url('/fonts/Zitiming.ttf');
	}

在该文件中你可以使用css样式设定字体，但很容易与NexT或Hexo本身的css样式冲突，所以建议只在主题custom.styl申明font-family
然后我们在主题 _config.yml中找到font字段
![](https://s2.ax1x.com/2019/03/11/A9zsHO.png)

如使用互联网字体，则URI上方的enable填true，本地则填false
子字段如global、heading那部分要用该字体就在family填入font-family

### 2.10 添加看板娘
![](https://huaji8.top/img/live2d/shizuku.gif)
当人们浏览你的博客时，视线中出现一个跟随你鼠标转动的妹纸，能够更加吸引人们的注意力哦~
#### 2.10.1 安装
让我们先返回站点目录

	npm install --save hexo-helper-live2d
	
完成后，目录中会出现node_modules文件夹
在站点目录下建文件夹live2d_models
再在live2d_models下建文件夹下安装模型：
	
		npm install --save live2d-widget-model-你喜欢的模型名字
	
更多模型可以到项目[github](https://github.com/xiazeyu/live2d-widget-models)中选择
将以下代码添加到主题配置文件_config.yml，修改 你喜欢的模型名字 ：

	live2d:
  		enable: true
  		scriptFrom: local
  		pluginRootPath: live2dw/
  		pluginJsPath: lib/
  		pluginModelPath: assets/
  		tagMode: false
  		debug: false
  		model:
    			use: live2d_models/live2d-widget-model-shizuku
  		display:
    			position: right
    			width: 150
    			height: 300
  		mobile:
    			show: true

运行命令hexo clean && hexo g && hexo s查看

### 2.11 打赏
最后这个嘛，读书人的事怎么能见讨呢，叫打赏
就是将你的微信或支付宝收款二维码贴在每篇博客末尾
首先把你的收款二维码图片放在themes/*/source/images中
ps.最好微信的重命名为wechatpay.png，支付宝的为alipay.png，省的等会儿改
然后在 主题配置文件中找到 Reward字段
子字段 reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！（这个自己随意）
下面三个付款方式，用哪个就去掉前面的#，注意路径的名称要对上啊

## 3. 站点设置
### 3.1 设置语言
编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

	language: zh-Hans
	
### 3.2 设置作者
编辑 站点配置文件，设置 author 为你的昵称。
### 3.2 社交链接
编辑 主题配置文件，查找social字段
![](https://s2.ax1x.com/2019/03/14/AAkKaj.png)
用哪个就去掉#，修改成自己的主页网址
ps. || 后面的别删，到时候相应图标显示不出的
再往下可以看到social_icon字段，enable为true。
![](https://s2.ax1x.com/2019/03/14/AAkUZ4.png)
如，我要显示github的icon，要与social字段内标题和||后的简写对应。





 

