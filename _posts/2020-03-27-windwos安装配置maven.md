---
layout: post
title: "Windwos安装配置maven"
date: 2020-03-27 13:51:13 +0800
tags: 杂谈
description: 
---

最近开始java开发，开始还是把jar包用拷贝得方式，放到项目得目录下，来进行引用，这种办法真的很麻烦，于是开始使用maven，由于开发使用的都是Windows系统，就先记录一下在Windows下的maven安装和配置

对于jdk的安装就不介绍了，这个没什么

### 安装

1. 访问官网进行[下载]
![](/images/2020-03-27-1.jpg)

2. 解压到本地任意目录，进行环境变量配置，配置**M2_HOME**和**MAVEN_HOME**两个变量，并加入到path中
![](/images/2020-03-27-2.jpg)
![](/images/2020-03-27-3.jpg)
![](/images/2020-03-27-4.jpg)

3. 配置好后，在命令行中进行查看，是否成功
![](/images/2020-03-27-5.jpg)


### 配置

1. 修改本地放置仓库，默认情况下是**${user.home}/.m2/repository**，如果修改的话，在安装目录下的/conf/settings.xml进行修改
![](/images/2020-03-27-6.jpg)

2. 修改远程仓库地址，默认的远程仓库在国外，由于某些原因，下载速度很慢，推荐修改为阿里云镜像
![](/images/2020-03-27-7.jpg)