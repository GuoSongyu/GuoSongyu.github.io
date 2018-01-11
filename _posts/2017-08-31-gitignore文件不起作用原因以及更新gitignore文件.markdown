---
layout: post
title:  "gitignore文件不起作用原因以及更新gitignore文件"
date:   2017-8-31 15:09:50 +0800
tags: github
description: 记录移动端网站开发过程中遇到的问题
---

git中可以在.gitignore文件中配置不需要记录到版本库中的文件或文件夹(比如上传的图片、带有数据库信息的配置文件、或者生产的无用文件等),但是做了一阵子项目之后，发现又有一些其他的文件需要忽略，增加到.gitignore文件并不起作用

因为.gitignore文件只记录在git init命令之后第一次提交记录中的文件，后续如果想更新.gitignore文件需要先删除缓存区的文件，重新提交才可以

{% highlight github %}
$ git rm -r cached .
$ git add .
$ git commit -m "重新配置.gitignore文件"
{% endhighlight %}

这样就可以使修改后的.gitnore记录我们新添加的文件记录了