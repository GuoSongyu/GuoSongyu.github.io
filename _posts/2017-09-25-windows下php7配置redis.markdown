---
layout: post
title:  "windows下php7配置redis"
date:   2017-9-25 22:41:37 +0800
tags: php
description: windows环境下，php7配置redis
---

最近在看一些nosql的东西，在某些情况下（虽然现在还不太知道"某些"指什么）需要配合nosql来是系统功能更加完善和效率，既然看了，就不能光看，还要动手才行，所以把自己配置redis的过程记录下来，我的环境是php7+apache2.4+mysql5.7

# 安装redis

1.首先要下载redis，这里附上github的[下载地址][redis]

2.下载完之后里面会有两个文件夹，32位和34位，选择自己系统对应的文件夹，放入到自己习惯的目录中

3.在目录中运行如下命令，redis.conf是配置文件，可以根据需要自行修改,运行成功之后，就证明redis已经启动，不要关闭，关闭的话就无法进行redis相关操作了
![](/images/2017-09-25-1.jpg)


# 安装php7的redis拓展

1.首先需要下载.dll格式的拓展文件[php_redis][php_redis]和[php_igbinary][php_igbinary](PS:下载的时候要注意php的版本和操作系统位数等信息)
![](/images/2017-09-25-2.jpg)


2.下载完毕后，将解压获得的php_redis.dll和php_igbinary.dll文件放入到php目录下的拓展目录ext中,并在php配置文件中加入如下代码后，重启apache
{% highlight php %}
extension=php_igbinary.dll
extension=php_redis.dll
{% endhighlight %}


3.重启完毕之后就会在phpinfo中查看到redis模块
![](/images/2017-09-25-3.jpg)



至此就在wamp环境下安装了redis，我们就可以使用它了，具体使用场景和方法，我也在慢慢学习中，有什么心得和体会的话还会继续更新记录在这里(:


[redis]:https://github.com/dmajkic/redis/downloads
[php_redis]:https://pecl.php.net/package/redis
[php_igbinary]:https://pecl.php.net/package/igbinary