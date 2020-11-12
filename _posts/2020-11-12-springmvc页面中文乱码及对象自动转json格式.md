---
layout: post
title: "springMVC页面中文乱码及对象自动转json格式"
date: 2020-11-12 21:34:39 +0800
tags: java
description: 
---

对于页面中文乱码的问题，需要在springmvc.xml增加如下配置

{% highlight xml %}
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>application/json;charset=UTF-8</value>
                    <value>text/html;charset=UTF-8</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
{% endhighlight %}

控制器直接返回对象时，用到的情况其实很多，比如接口返回值之类的。开始的时候，虽然我在控制器中增加了@RestController但是返回的时候，仍然会报错。还需要要引入第三方json工具类，并在配置文件中进行配置。

对于json工具类，我选择了谷歌的gson，springmvc原本就支持gson类，所以只需要增加一句配置就可以了

{% highlight xml %}
<dependency>
  	<groupId>com.google.code.gson</groupId>
  	<artifactId>gson</artifactId>
  	<version>2.8.5</version>
</dependency>
{% endhighlight %}

然后在springmvc.xml继续增加

{% highlight xml %}
<mvc:annotation-driven>
    <mvc:message-converters>
    	<!-- 防止中文乱码 -->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>application/json;charset=UTF-8</value>
                    <value>text/html;charset=UTF-8</value>
                </list>
            </property>
        </bean>
        <!-- 对象自动转json -->
        <bean class="org.springframework.http.converter.json.GsonHttpMessageConverter"/>
    </mvc:message-converters>
</mvc:annotation-driven>
{% endhighlight %}