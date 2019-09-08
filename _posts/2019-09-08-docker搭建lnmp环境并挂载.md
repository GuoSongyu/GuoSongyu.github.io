---
layout: post
title: "docker搭建lnmp环境并挂载"
date: 2019-09-08 21:57:00 +0800
tags: docker
description: 
---


上篇文章讲到已经将虚拟机中的目录挂载到本机中，那么，接下来要搭建的环境，只需要将软件安装到虚拟机中的挂载目录，在本机中就可以看到了。

作为一个PHPer，当然要首先配置php的运行环境。（以php7.0，MySQL5.7，nginx作为环境）：

### [](#header-3)一、mysql5.7安装
1、下载镜像
{% highlight dos %}
docker pull mysql:5.7
{% endhighlight dos %}

2、启动镜像
{% highlight dos %}
docker run --name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD mysql:5.7
{% endhighlight dos %}

3、启动成功后，将mysql容器中的配置文件复制到本机目录中（挂载时只挂载目录，不挂载文件）
{% highlight dos %}
docker cp 288e7f3b044d(容器id):/etc/mysql/conf.d /mydocker/lnmp/mysql/conf（本地挂载目录
{% endhighlight dos %}

4、删除容器，重启建立容器，并挂载配置目录和数据目录
{% highlight dos %}
docker run --name mysql5.7 \
-p 3306:3306 \
-v /mydocker/lnmp/mysql/data（虚拟机中挂载目录）:/var/lib/mysql \
-v /mydocker/lnmp/mysql/conf（虚拟机中挂载目录）:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
{% endhighlight dos %}

由于在将VirtualBox虚拟机中的目录以共享的形式挂载到本地，所以在启动的时候会报错

从错误中大概能看到好像是因为mysql默认数据是innodb格式，然后异步IO的配置在虚拟机中不被支持，所以在启动的时候，在最后面增加了一行my.cnf的默认配置将异步IO配置关掉,在命令最后添加
{% highlight dos %}
--innodb-use-native-aio=0
{% endhighlight dos %}
添加之后挂载成功，并且容器启动成功

### [](#header-3)二、nginx安装
1、下载nginx镜像
{% highlight dos %}
docker pull nginx
{% endhighlight dos %}

2、建立容器，并将配置文件复制到本地得虚拟机挂载目录中
{% highlight dos %}
docker cp e797b38dd76b(容器id):/etc/nginx/conf.d /mydocker/lnmp/nginx/conf/conf.d（本地挂载目录）
{% endhighlight dos %}

3、删除容器，重新建立容器，并挂载配置文件、项目空间和日志目录
{% highlight dos %}
docker run --name nginx -p 80:80 \
-v /mydocker/lnmp/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro \
-v /mydocker/lnmp/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /mydocker/lnmp/nginx/log:/var/log/nginx \
-v /mydocker/lnmp/nginx/html:/usr/share/nginx/html nginx:latest
{% endhighlight dos %}

4、建立成功，可以直接在本地目录中修改配置文件

### [](#header-3)三、php7.0安装
1、下载php镜像
{% highlight dos %}
docker pull php:7.0-fpm
{% endhighlight dos %}

2、建立容器，并挂载文件，将php的html目录挂载到和nginx的html相同目录下
{% highlight dos %}
docker run --name php7 \
-p 9000:9000 \
-v /mydocker/lnmp/php:/usr/local/etc/php \
-v /mydocker/lnmp/nginx/html:/var/www/html \
php:7.0-fpm
{% endhighlight dos %}

3、查询php容器得分配ip
{% highlight dos %}
docker inspect --format='{ {.NetworkSettings.IPAddress} }' php7.0
{% endhighlight dos %}

4、将查询到的php容器ip配置到nginx的配置文件中，重启nginx即可访问phpinfo信息

### [](#header-3)四、docker network配置

上面的步骤中，获取到php的ip之后，配置到nginx中，有一个缺点，就是php容器的ip是会变化的，所以如果一直这样的话，我们在每次使用之前就要去检查ip是否正确，再进行替换，这样非常不方便。

好在docker提供了docker network命令，可以将nginx的配置文件中的ip直接换成对应php容器的名称就可以，这样就不会被ip烦恼了

1、创建network
{% highlight dos %}
docker network create lnmp
{% endhighlight dos %}

2、将php和nginx分别加入到刚才的network中
{% highlight dos %}
docker network connect lnmp php7.0
docker network connect lnmp nginx
{% endhighlight dos %}

3、加入成功后可以查询network下的容器
{% highlight dos %}
docker network inspect lnmp
{% endhighlight dos %}

4、将nginx中的fastcgi_pass配置中的ip修改为php容器名称，重启nginx即可


以上就是lnmp环境搭建的步骤，成功之后，就可以运用docker来进行本地开发了