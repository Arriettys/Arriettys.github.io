---
title: Hexo博客多台电脑设备同步管理
date: 2019-03-25 21:02:33
tags: Hexo
categories: Web
---
最近一直在折腾Hexo博客, 玩的可谓是不亦乐乎啊，但也遇到了些问题我想在不同的终端进行github+Hexo的博客发布更新该如何进行呢，在Google中搜了一些教程，并自身进行了简化与实践！主体的思路是将博文内容相关文件放在Github项目中master中，将Hexo配置写博客用的相关文件放在Github项目的hexo分支上，这个是关键，多终端的同步只需要对分支hexo进行操作。下面是详细的步骤讲解：
<!--more-->
**首先，你得有个Hexo博客，没搭的看这里，**{% post_list HexoWeb%}
### 1. Hexo 本地文件同步
将Github+Hexo搭建自己的博客时安装的配置文件push到github的username.githun.io的hexo分支上

![](https://s2.ax1x.com/2019/03/25/AtqYUx.png)

提交时考虑以下注意事项

 + 将themes目录以内中的主题的.git目录删除（如果有），因为一个git仓库中不能包含另一个git仓库，否则提交主题文件夹会失败
 + 后期需要更新主题时在另一个地方git clone下来该主题的最新版本，然后将内容拷到当前主题目录即可
```
git init  //初始化本地仓库
git add . //当前文件全部上传
git commit -m "Blog Source Hexo"
git branch hexo  //新建hexo分支
git checkout hexo  //切换到hexo分支上
git remote add origin git@github.com:yourname/yourname.github.io.git  //将本地与Github项目对接
```
这样你的github项目中就会多出一个Hexo分支，这个就是用于多终端同步关键的部分。

### 2. 另一终端完成clone和push更新
此时在另一终端更新博客，只需要将Github的hexo分支clone下来，进行初次的相关配置
```
git clone -b hexo git@github.com:username/username.github.io.git  //将Github中hexo分支clone到本地
cd  username.github.io  //切换到刚刚clone的文件夹内
npm install    //注意，这里一定要切换到刚刚clone的文件夹内执行，安装必要的所需组件，不用再init  (由于仓库有一个.gitignore文件，里面默认是忽略掉 node_modules文件夹的，也就是说仓库的hexo分支并没有存储该目录，所以需要install下)
hexo new post "new blog name"   //新建一个.md文件，并编辑完成自己的博客内容
git add source  //因为只是新建了一篇新博客，所以这里只提交source，若其他文件改动，也要提交同步哦
git commit -m "XX"
git push origin hexo  //更新分支
hexo d -g   //push更新完分支之后将自己写的博客对接到自己搭的博客网站上，同时同步了Github中的master
```

### 3. 同步之后
至此同步设置就完成了，以后每次写博客记得先和线上的仓库同步hexo，做完自己的时候，再记得提交同步

```
git pull origin hexo  //先pull完成本地与远端的融合
...
...
git add * //你做出的改动 
git commit -m "XX"
git push origin hexo
hexo d -g
```
