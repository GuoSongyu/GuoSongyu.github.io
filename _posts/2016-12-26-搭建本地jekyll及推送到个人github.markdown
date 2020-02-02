---
layout: post
title:  "搭建本地jekyll及推送到个人github"
date:   2020-01-25 19:13:50 +0800
tags: jekyll
description: 自己搭建jekyll的过程
---
看了github pages之后，就特别想给自己搭建一个，因为github pages是根据jekyll的环境直接生成的静态页，所以自己想在本地搭建jekyll，但是Windows下搭建遇到了很多问题，所以写了这篇文章把自己遇到的问题和搭建的步骤整理一下

建立一个个人的github pages首先肯定要有一个github账号，这个就不用赘述了~

jekyll是一个无需数据库，非常简单而且快捷的播客搭建工具，相对于WordPress，省去了搭建服务器、管理数据库等很多繁琐的步骤，只需要几个静态页面就能够搭建成功

我觉得主要是在Windows下本地搭建jekyll的运行环境真的是繁琐，我弄了好多天，中间经历了安装、卸载、又安装的过程，才得以搭建成功，下面把我搭建的过程记录一下（因为我是Windows系统，所以只记录Windows下的步骤，由于版本更新，之前得安装步骤已经发生变化，所有修改新的步骤如下）：

## [](#header-2)安装前提

jekyll是用Ruby语言（原谅我的无知，如果不是这次学jekyll我真的不知道这个语言）编写的，所以搭建本地的jekyll总体上可以分为3个步骤：

1、安装Ruby + DevKit：进入[下载地址][rubyinstaller]，选择如下版本进行安装，按照默认选项进行安装即可，安装完成后，结束界面会有一个勾选得 ~~ridk install~~ 不用处理，自动执行此命令即可

![](/images/2020-01-25-1.jpg)

2、安装完成后，进入命令行界面，执行 ~~ruby -v~~ 查看版本

![](/images/202-01-25-2.jpg)

3、安装jekyll

## [](#header-2)安装步骤

安装好ruby + DevKit之后，会在DevKit的根目录下生成一个配置文件 config.yml，里面会包含之前安装ruby的路径

![](/images/2020-01-25-3.jpg)

接下来运行gem install jekyll bundler 来安装jekyll了，安装成功后，对版本进行查看

![](/images/2020-01-25-4.jpg)

## [](#header-2)创建并运行

安装jekyll成功后，选择一个要创建站点的目录，中运行 jekyll new _filename_ ,创建文件成功

![](/images/2016-12-26-2.png)

进入到刚刚创建的文件中，运行站点，出现下图提示后，运行成功

![](/images/2016-12-26-3.png)

OK！创建成功，可以根据[文档][jekyll-docs]进行对站点的修改了

![](/images/2016-12-26-4.png)

## [](#header-2)推送到远程github库中

要推送到github中并生效首先要在你的github账号下面建立一个仓库，并以 _username.github.io_ 来命名

在本地创建好的站点中，执行git init 来初始化本地git库

运行 git remote add origin git@github.com:GuoSongyu/GuoSongyu.github.io.git 来添加远程仓库（就是把本地的仓库与远程的仓库关联起来，并且在此之前一定要记得在个人账号下面进行SSH keys设置）

添加完成之后git push origin master把本地内容推送到远程仓库

推送成功之后就可以去 https://username.github.io/ 中去查看了！

## [](#header-3)结语

使用了很长时间之后。突然更换得新得系统，需要重新搭建jekyll环境，结果按照之前得流程，出现了很多问题，于是重新对文档进行阅读和查看，对安装步骤进行了更新

附上文档[地址][jekyll-install]

[rubyinstaller]: http://rubyinstaller.org/
[jekyll-docs]: http://jekyll.com.cn/docs/home/
[jekyll-install]: https://jekyllrb.com/docs/installation/windows/
