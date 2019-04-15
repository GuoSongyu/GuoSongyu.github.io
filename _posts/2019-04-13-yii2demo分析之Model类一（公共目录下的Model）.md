---
layout: post
title: "yii2demo分析之Model类一（公共目录下的Model）"
date: 2019-04-13 16:44:12 +0800
tags: yii2
description: 
---

填坑的时间有点长了。。。我是罪人，直接正题吧

yii2的demo中，目录结构如下
![](/images/2019-04-13-1.jpg)

backend和frontend两个目录分别是站点后台和站点前台的目录，common目录为公共目录，公共的Model、配置、组件或者其他等文件的存放目录，但是在demo目录中common目录和前后台目录中都存在Model目录，本次介绍主要为common目录中的Model类的作用
![](/images/2019-04-13-2.jpg)

以common/models目录中的User.php为例，内容如下（依然是坑爹的笔记本屏幕）：
![](/images/2019-04-13-3.jpg)
![](/images/2019-04-13-4.jpg)

该类型文件可用gii自动生成，注释部分为对应的数据表字段（PS:User.php中实现了IdentityInterface接口，该接口作用在[2018-12-08-yii2demo分析之后台访问权限验证流程分析](https://guosongyu.github.io/2018/12/yii2demo%E5%88%86%E6%9E%90%E4%B9%8B%E5%90%8E%E5%8F%B0%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E9%AA%8C%E8%AF%81%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90)进行过介绍，大部分Model类不不需要实现此接口）

该文件继承ActiveRecord类，在yii中，该类用以访问和操作数据库中的数据，与数据库进行关联。
本例中，tableName和rules两个函数可以自动生成，前者返回关联的数据库名称，后者在保存字段时对字段进行验证，behaviors函数中可以根据需求实现一些行为逻辑，对数据进行加工，典型的MVC模式中的M层结构。

该类型的Model文件中函数建议大部分为直接操作数据的函数，如查询、保存等；或加工字段，如把要保存的字段格式处理为数据库中的格式。

该类中能够处理绝大多数的数据库操作，如关联查询、事务、自定义查询字段等，能够大大减少时间成本，对于该类中的详细用法，我就不再一次进行重复了。 [文档](https://www.yiichina.com/doc/guide/2.0/db-active-record)中的介绍远比我详细的多。。。