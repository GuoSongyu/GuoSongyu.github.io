---
layout: post
title:  "Windows下Apache配置https"
date:   2017-09-01 21:25:54 +0800
tags: 杂谈
description: 从申请证书到搭建成功，记录在Windows环境搭建https的过程
---

突然有一个新的项目，要做小程序，在配置远程地址的时候，发现在小程序的后台设置中是默认https开头的地址，但是我的服务器是http的，并没有开启ssl，也从来没有弄过，于是。。这一切就发生了


## [](#header-2)申请证书

要配置https，第一步就是要申请SSL证书，我看了好多帖子可以通过apache中的openssl.cnf文件来自制证书，但是现在好像不能用了，基本的浏览器都不识别这种自制证书，所以只好去申请，但是发现好多都是花钱买的，找了好久发现可以使用Let's Encrypt的证书，但是缺点就是有效期只有90天，需要每次到期的时候来重新申请一下，为了信仰。。这点缺点不算什么，下面来说证书的申请过程：

1. 首先进入sslforfree的网站[sslforfree][sslforfree],在红色区域填上要申请证书的域名（单域名证书）
![](/images/2017-09-01-1.jpg)


2. 点击**Create Free SSL Certificates**到第二部，从左到右第一个选项是通过FTP来验证域名，第二个是上传证书文件到你自己域名下的服务器，第三个是通过DNS方式，选择第二个方法进入下一步
![](/images/2017-09-01-2.jpg)


3. 点击之后会出现一个类似说明的部分，点击之后就会弹出验证域名阶段，此阶段通过之后就可以去生成证书了,这个验证的步骤就是：
- 首先在网站中下载两个验证文件
- 根据要求在根目录下建立一个文件夹，把下载的文件放到里面
- 然后按照地址去依次访问。访问成功之后就可以点击**Download SSL Certificates**来生成证书了
![](/images/2017-09-01-3.jpg)


4. 成功之后会让你输入邮箱以便过期的时候发送消息
![](/images/2017-09-01-4.jpg)


5. 输入邮箱之后就可以下载证书了
![](/images/2017-09-01-5.jpg)


6. 下载的是一个压缩文件，解压之后存在如下三个文件，到这里证书就申请完毕了，接下来就是部署到服务器了
![](/images/2017-09-01-6.jpg)


## [](#header-2)服务器部署

大部分的时间都坑在服务器部署中了，这里一定要记录的详细一些，先说明一下步骤，然后把遇到的问题放在后面

1. 首先吧3个文件上传到服务器，放在Apache24/conf目录下


2. 开启mod_ssl模块、启用httpd-ssl.conf配置文件（去掉前面的#）


3. httpd-ssl.conf中：
- 去掉SSLPassPhraseDialog  builtin前面的#
- 修改VirtualHost中 DocumentRoot、ServerName等（ServerName要是申请的域名）
- SSLEngine 改为On（SSL开启）
- SSLCertificateFile 对应为 certificate.crt 、SSLCertificateKeyFile对应为 private.key、SSLCACertificateFile对应为 ca_bundle.crt


4. 然后重启Apache就可以通过https来直接访问


## [](#header-2)遇到的问题

1. Apache启动之后发现访问的时候提示缺少证书或者证书未上传

> 这个问题真的卡了好久，最后发现是因为自己开启了SSLVerifyClient
> 这个选项的意思是除了服务端验证证书之外还要求客户端也要验证
> 但是我们申请只是服务端的证书，所以不要打开这个选项


2. 配置成功之后发现只有首页可以用https访问，跳转到其他链接的时候出现找不到链接的错误，但是用rewrite之前的域名是可以访问的（在此之前服务器已经开启rewrite）

> 这个问题可能经验不足吧，在httpd-ssl.conf文件中，除了配置其他
> 一定不要忘记了Directory来使指定项目的.htaccess文件生效

## [](#header-2)小结

这次配置https的过程一共用了差不多2天的时间申请证书差不多半天时间，中间还试了好多渠道，剩下的1天多的时间基本都在部署Apache，查了好多的文章，完成了之后觉得成就感满满

但是仔细想想，有很多问题是可以避免的，比如第二个问题，就是在虚拟主机配置的时候没有仔细的观察和记录，导致浪费了很多时间


最后，还是要加强学习啊~~~~~~~



[sslforfree]:https://www.sslforfree.com/