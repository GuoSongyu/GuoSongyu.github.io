---
layout: post
title: "DockerToolbox的安装与挂载"
date: 2019-08-28 21:02:55 +0800
tags: docker
description: 
---

“最近晚上有空嘛，给你俩分享点东西。”
“都知道代码用GitHub进行远程仓库储存，用了这个环境也可以储存在远程仓库，以后环境、代码全可以通过远程仓库来进行本地部署。”
通过简单的介绍，大佬（敬爱的宝哥）开始了对docker的简单说明，之后就有了自己的尝试和环境部署，于是就有了这篇文章

最开始看了官方的[安装文档（Windows）][install]，官方推荐安装Docker Desktop，由于我的Win10是家庭版的，无法安装，只好安装DockerToolbox来作为docker的使用工具，由于Win版的安装包是exe文件。就不做过多的介绍了

注：在DokcerToolbox安装完成后，启动时会自动下载boot2docker.iso作为虚拟主机文件，但是由于网络问题，可能会出现问题，其实在DokcerToolbox的安装目录中，就已经存在此文件，可以将文件复制到指定目录中（我本地的目录为：C:\Users\NAME\.docker\machine\cache），在重新启动即可

安装完成后，同时还会为我们安装Oracle VM VirtualBox虚拟机管理软件，稍后的虚拟机目录转移和目录挂载，都是靠这个来完成的

由于虚拟机默认是安装在C盘用户目录下的，同时虚拟硬盘也是在C盘下的，如果要将虚拟硬盘移动到其他目录，需要怎么做呢

![](/images/2019-08-28-1.jpg)
点击虚拟介质管理

![](/images/2019-08-28-2.jpg)
![](/images/2019-08-28-3.jpg)
点击复制，按钮，按照步骤操作，注意类型选选择最后一个虚拟机磁盘

![](/images/2019-08-28-4.jpg)
点击设置，并选择左侧的储存，可以将原来的删除，然后点击加号，选择使用现有磁盘按钮，选择刚才添加的虚拟磁盘

DockerToolbox的其实就是在Windows中安装一个虚拟机，然后通过这个虚拟机去和docker进行连接，所以在本机是看不到文件目录的，在开发中需要操作文件的话，只靠命令行的话肯定是不会方便的，所以我们可以在虚拟机中创建一个目录，然后将目录挂载到本地的一个目录，这样虚拟机中的文件就会实时出现在我们的本机中了，挂载步骤如下：

![](/images/2019-08-28-6.jpg)
![](/images/2019-08-28-5.jpg)
同样选择设置，然后左侧的共享文件夹按钮,点击加号，注意下面的选项不要选择只读分配，然后记住那个共享文件夹名称，稍后会用到

然后通过命令，进入虚拟机中创建一个目录，并将刚才的共享文件夹名称与目录挂载
{% highlight dos %}
docker-machine ssh default
{% endhighlight dos %}
{% highlight dos %}
sudo mount -b vboxsf mydocker /mydocker
{% endhighlight dos %}
vboxsf后面两个名称前面是共享文件夹名称，后面是虚拟主机的目录，这样就挂载成功了，在这个目录中的修改也会直接映射在本地的文件夹中


[install]:https://docs.docker.com/docker-for-windows/install/