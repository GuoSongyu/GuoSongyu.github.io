---
layout: post
title: "nginx中return的使用"
date: 2020-02-19 14:52:57 +0800
tags: nginx
description: 
---

### nginx中return关键字，可以在server、location、if中使用

### 直接返回链接
{% highlight nginx %}
server{
	..
	..
	return http://www.baidu.com;
}
{% endhighlight nginx %}

### 返回字符串
{% highlight nginx %}
server{
	..
	..
	return 200 "error";
}
{% endhighlight nginx %}

### 返回变量
{% highlight nginx %}
server{
	..
	..
	return 200 "$host$request_uri";
}
{% endhighlight nginx %}

### HTML代码
{% highlight nginx %}
server{
	..
	..
	return 200 "<html><script>window.location.href='$host$request_uri'</script></html>";
}
{% endhighlight nginx %}

### json
{% highlight nginx %}
server{
	..
	..
	return 200 '{"name":"muyu","id":1}';
}
{% endhighlight nginx %}