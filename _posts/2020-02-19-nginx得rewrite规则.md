---
layout: post
title: "nginx得rewrite规则"
date: 2020-02-19 14:22:17 +0800
tags: nginx
description: 
---

rewrite格式
> 格式：rewrite regex replacement [flag]
>
> rewrite配置可以在server、location以及if配置段内生效
>
> regex是用于匹配URI的正则表达式，其不会匹配到$host（域名）
>
> replacement是目标跳转的URI，可以以http://或者https://开头，也可以省略掉$host，直接写$request_uri部分（即请求的链接）
>
> flag，用来设置rewrite对URI的处理行为，其中有break、last、rediect、permanent
>
> rediect和permanent的区别在于，前者为临时重定向(302)，而后者是永久重定向(301)，对于用户通过浏览器访问，这两者的效果是一致的。但是，对于搜索引擎蜘蛛爬虫来说就有区别了，使用301更有利于SEO。所以，建议replacemnet是以http://或者https://开头的flag使用permanent。
>
> break和last如果直接在server中使用的话，并没有明显的区别，都是匹配到规则的话，直接进行访问，如果在location中的话，break不会跳出当前location，继续按照匹配的规则继续执行；如果是last的话，会跳出当前location，匹配其他的location

### break和last的说明

现在有如下rewrite规则
{% highlight nginx %}
server{
	....
	....
	location / {
		rewrite /1.html /2.html;
		rewrite /2.html /3.html;
	}
	
	location /2.html {
		rewrite 2.html a.html;
	}

	location /3.html {
		rewrite 3.html b.html;
	}

}
{% endhighlight nginx %}
直接访问的话，会按照这个顺序，进行跳转**1.html => 2.html ； 2.html => 3.html ; 3.html => b.html**

如果在第一个location中的第一行，接口加上break

{% highlight nginx %}
server{
	....
	....
	location / {
		rewrite /1.html /2.html break;
		rewrite /2.html /3.html;
	}
	
	location /2.html {
		rewrite 2.html a.html;
	}

	location /3.html {
		rewrite 3.html b.html;
	}

}
{% endhighlight nginx %}

访问的时候，匹配到break就直接停止继续访问了， 1.html => 2.html

如果把break改为last的话

{% highlight nginx %}
server{
	....
	....
	location / {
		rewrite /1.html /2.html last;
		rewrite /2.html /3.html;
	}
	
	location /2.html {
		rewrite 2.html a.html;
	}

	location /3.html {
		rewrite 3.html b.html;
	}

}
{% endhighlight nginx %}
当访问匹配到到last时，当前location下面的规则不在匹配，继续匹配其他符合条件的location规则，所以应该是 **1.html => 2.html ; 2.html => a.html**的访问结果