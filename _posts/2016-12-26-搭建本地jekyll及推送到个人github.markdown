---
layout: post
title:  "搭建本地jekyll及推送到个人github"
date:   2016-12-26 20:27:50 +0800
tags: jekyll
description: 自己搭建jekyll的过程
---
看了github pages之后，就特别想给自己搭建一个，因为github pages是根据jekyll的环境直接生成的静态页，所以自己想在本地搭建jekyll，但是Windows下搭建遇到了很多问题，所以写了这篇文章把自己遇到的问题和搭建的步骤整理一下

建立一个个人的github pages首先肯定要有一个github账号，这个就不用赘述了~

jekyll是一个无需数据库，非常简单而且快捷的播客搭建工具，相对于WordPress，省去了搭建服务器、管理数据库等很多繁琐的步骤，只需要几个静态页面就能够搭建成功

我觉得主要是在Windows下本地搭建jekyll的运行环境真的是繁琐，我弄了好多天，中间经历了安装、卸载、又安装的过程，才得以搭建成功，下面把我搭建的过程记录一下（因为我是Windows系统，所以只记录Windows下的步骤）：

## [](#header-2)安装前提

jekyll是用Ruby语言（原谅我的无知，如果不是这次学jekyll我真的不知道这个语言）编写的，所以搭建本地的jekyll总体上可以分为3个步骤：

1、安装Ruby：首先要下载rubyinstaller，[下载地址在这里][rubyinstaller]，选择需要的版本

2、安装Ruby DevKit：在ruby的下载地址中同样存在对应版本的DevKit，下载的时候一定要看好对应ruby的版本

3、安装jekyll

## [](#header-2)安装步骤

把下载的rubyinstaller安装到指定的目录中，如E:/，安装过程中所需的选项最好全部勾选，尤其是自动配置环境变量这里；然后把下载的DevKit也安装到目录中，如E:/DevKit

安装好之后就在DOS命令中进入DevKit中

{% highlight ruby %}
cd DevKit
ruby dk.rb init
{% endhighlight %}

运行结束后会在DevKit的根目录下生成一个配置文件 config.yml，里面会包含之前安装ruby的路径（理论上是这样，但是我执行之后并没有ruby的路径，ruby的安装路径是我手动添加上去的）

![](/images/2016-12-26-1.png)

接下来运行ruby dk.rb install。运行成功之后。就可以运行 gem install jekyll 来安装jekyll了(在创建过程中可能会出现缺少一些依赖文件的错误，按照提示安装上即可)


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

推送成功之后就可以去 https://username.github.io/ 中去查看了~！

## [](#header-3)结语

这次自己建立github pages 感觉真的是打开了新世界的大门，从对jekyll一点都不熟悉到安装、创建、运行，中间进行了很多查找和学习，尤其是在本地搭建jekyll的时候，遇到了很多的问题，幸好最终还是把这些问题解决了,同时对github的运用也变得更加熟练

最后，真的很谢谢大神们的文章：

[Jekyll 安装、使用方法与卸载][jekyll-from1]

[Using Jekyll with Pages——使用Jekyll框架][jekyll-from2]

[一步步在GitHub上创建博客主页-最新版][jekyll-from3]

[Jekyll本地搭建开发环境以及Github部署流程][jekyll-from4]

[jekyll-from1]: http://blog.csdn.net/u012675539/article/details/43734055
[jekyll-from2]: http://blog.csdn.net/coderhuhy/article/details/45667973
[jekyll-from3]: http://www.pchou.info/ssgithubPage/2014-07-04-build-github-blog-page-08.html
[jekyll-from4]: http://pizida.com/technology/2016/03/03/use-jekyll-create-blog-on-github/#jekyll-1
[rubyinstaller]: http://rubyinstaller.org/
[jekyll-docs]: http://jekyll.com.cn/docs/home/

