---
layout: post
title: "springMVC整合thymeleaf"
date: 2020-11-06 21:09:06 +0800
tags: java
description: 
---

对于maven项目来说，引入其他jar包是非常方便的。这也是maven的优势之一。
上篇文章说到，对于视图解析器，我们准备使用thymeleaf，特点什么的就不介绍，去[官网][thymeleaf]查看即可

## 引入maven配置

对于thymeleaf的使用，我们需要引入连个jar包，一个是thymeleaf本身，另一个就是thymeleaf对于spring框架的支持jar包
{% highlight xml %}
<dependency>
  <groupId>org.thymeleaf</groupId>
  <artifactId>thymeleaf</artifactId>
  <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.thymeleaf</groupId>
  <artifactId>thymeleaf-spring4</artifactId>
  <version>3.0.11.RELEASE</version>
</dependency>
{% endhighlight %}

## springmvc中配置视图解析器

{% highlight xml %}
<!-- 视图解析器 -->
<bean id="templateResolver" class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">
    <!--前缀（视图根路径）-->
    <property name="prefix" value="/WEB-INF/views/"></property>
    <!--后缀（类型）-->
    <property name="suffix" value=".html"></property>
    <!--模板类型-->
    <property name="templateMode" value="HTML"></property>
    <!--不使用缓存-->
    <property name="cacheable" value="false"></property>
    <!--编码类型-->
    <property name="characterEncoding" value="UTF-8"></property>
</bean>
<!--模板引擎配置-->
<bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
    <property name="templateResolver" ref="templateResolver"></property>
</bean>
<!--视图处理器-->
<bean class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
    <property name="templateEngine" ref="templateEngine"></property>
    <property name="characterEncoding" value="UTF-8"></property>
</bean>
{% endhighlight %}

## 结果测试

配置完成后，我们可以创建一个Controller进行测试。具体代码如下

控制器部分：

{% highlight java %}
@Controller
@RequestMapping(value = "show")
public class ShowController{
	@GetMapping(value = "index")
	public ModelAndView index(){
		//注意这里返回的是ModelAndView对象
	    ModelAndView mv = new ModelAndView();
	    mv.addObject("name","暮语");
	    //由于我们在配置文件中配置了根路径和后缀，此处自需要写出视图中间路径即可
	    mv.setViewName("demo/show/index");

	    return mv;
	}	
}
{% endhighlight %}

Html部分：

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Thymeleaf Demo</title>
</head>
<body>
	<!-- 这里读取的是，控制器中传过来的参数 -->
    <p>我得名字叫：[[${name}]]</p>
</body>
</html>
{% endhighlight %}

完成后，启动项目并在浏览器进行访问，返回如下结果，说明配置成功！

![](/images/2020-11-06-1.png)

Ok，视图引入完成后，下一篇该轮到数据库了

对于thymeleaf的用法，以后如果有机会的话，会写一篇文章单独介绍一下
![](/images/2020-11-06-2.jpg)


[thymeleaf]:https://www.thymeleaf.org/