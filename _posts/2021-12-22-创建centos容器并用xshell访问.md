---
layout: post
title: "创建centos容器并用xshell访问"
date: 2021-12-22 22:32:12 +0800
tags: docker
description: 
---

最近在看Linux相关的东西，但是手中又没有Linux系统，弄个虚拟机又太麻烦。docker的话正适合我这种懒人。一个命令，就可以启动centos容器，并可以用Xshell进行访问，用来学习非常方便，步骤也非常简单：

### 下载镜像
这里我用的是centos7的镜像
{% highlight dos %}
docker pull centos:7
{% endhighlight dos %}

### 建立容器
注意这里启动的时候要增加 **--privileged** 参数拿到容器的权限。映射到本机的端口无所谓，哪个方便用哪个
{% highlight dos %}
docker run --name centos-test -itd -p 5001:22 --privileged centos:7 init
{% endhighlight dos %}

为了每次开机不用启动容器，可以增加 --restart=always 参数，每次docker启动时，该容器也会一起启动

### 设置root密码
密码随意设置就好，为了使用xshell进行访问
{% highlight dos %}
passwd
{% endhighlight dos %}

### 安装ssh-server
{% highlight dos %}
yum install openssh-server -y
{% endhighlight dos %}
安装完成后，修改配置
> 38行去掉 **PermitRootLogin yes** 的注释
> 96行的 **UsePAM yes** 修改为 **UsePAM no**
然后开启ssh服务
{% highlight dos %}
systemctl start sshd
{% endhighlight dos %}
退出容器后就可以使用xshell进行访问了

![](/images/2021-12-22-1.PNG)