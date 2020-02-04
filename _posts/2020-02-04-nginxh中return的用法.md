---
layout: post
title: "nginxh中return的用法"
date: 2020-02-04 21:24:29 +0800
tags: 
description: 
---

> 可以在server、location、if中直接使用
> 返回状态码
> 返回连接
> 返回字符串
> 返回变量
> 返回HTML代码
> 返回json

{% highlight dos %}
server{
	..
	..
	#状态码
	return 403;
}
{% endhighlight dos %}

{% highlight dos %}
server{
	..
	..
	#连接
	return http://www.baidu.com;
}
{% endhighlight dos %}

{% highlight dos %}
server{
	..
	..
	#字符串
	return 200 "hello";
}
{% endhighlight dos %}

{% highlight dos %}
server{
	..
	..
	#变量
	return "$host$request_uri";
}
{% endhighlight dos %}

{% highlight dos %}
server{
	..
	..
	#html
	return "<html><script>window.location.href='$host$request_uri'</script></html>";
}
{% endhighlight dos %}

{% highlight dos %}
server{
	..
	..
	#json
	return '{"name":"hello","id":1}';
}
{% endhighlight dos %}