---
layout: post
title:  "git pull失败提示refusing to merge unrelated histories"
date:   2017-9-22 11:49:05 +0800
tags: github
description: 推送远程仓库，git pull提示fatal:refusing to merge unrelated histories
---

写了一些简单的类库，写完之后想暂时推送的github仓库中，但是在git pull的时候，出现了如下提示，fatal: refusing to merge unrelated histories，要在后面增加一段命令之后才可以git pull成功：

{% highlight github %}
git pull origin master --allow-unrelated-histories
{% endhighlight %}

我用的是github 2.10.1版本，看了原因应该是在2.9之后git更改了一些默认行为，在默认情况下，git是不允许被一个不知情的维护者，随意合并历史，所以要在后面增加--allow-unrelated-histories命令


附上 stack overflow 大佬的解读：[https://stackoverflow.com/questions/37937984/git-refusing-to-merge-unrelated-histories][overflow]


[overflow]:https://stackoverflow.com/questions/37937984/git-refusing-to-merge-unrelated-histories