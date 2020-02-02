---
layout: post
title: "centos7安装nginx"
date: 2020-02-02 11:33:36 +0800
tags: nginx
description: 
---

最近在学习nginx，把一些东西记录下来，方便以后去查询，也顺便当作对自己得监督吧

在centos上安装nginx其实没有想象得复杂

## yum源安装

### 一、配置yum源

进入到nginx官网中，获取源配置内容，进入centos得yum源配置目录，新建文件/etc/yum.repos.d/nginx.repo,并将内容复制到文件中

![](/images/2020-02-02-1.jpg)

![](/images/2020-02-02-2.jpg)

运行 yum list | grep nginx 命令可以查看刚才配置好的源

![](/images/2020-02-02-3.jpg)

###二、安装nginx

运行 yum install nginx 进行安装，等待安装完成后，执行nginx -V对nginx信息进行查看，证明nginx安装成功，可以进行使用了

![](/images/2020-02-02-4.jpg)


如果在启动过程中出现如下问题，可以进入到nginx.conf配置文件中，将此项注释打开，将目录修改到log下，重启即可

{% highlight dos %}
遇到：nginx: [error] open() "/var/run/nginx.pid" failed (2: No such file or directory)
{% endhighlight dos %}

![](/images/2020-02-02-5.jpg)

## 源码安装

### 一、下载并解压

进入nginx官网，下载需要得版本（推荐稳定版）,下载后，将压缩包拷贝至/usr/local/src目录下，运行命令进行解压

![](/images/2020-02-02-6.jpg)

{% highlight dos %}
tar zxf nginx-1.16.1.tar.gz
{% endhighlight dos %}

### 二、安装

进入到nginx-1.16.1目录，对源码进行编译，可以在编译前，查看可编译模块

{% highlight dos %}
#查看可编译模块
./configure --help
{% endhighlight dos %}

进行编译，并执行make命令安装，这里进行简单得命令，需要其他模块可按照说明进行增加

{% highlight dos %}
./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf
make && make install
{% endhighlight dos %}

安装完成后，进行启动
{% highlight dos %}
#启动服务
/usr/local/nginx/sbin/nginx
#查看版本
/usr/local/nginx/sbin/nginx -V
{% endhighlight dos %}