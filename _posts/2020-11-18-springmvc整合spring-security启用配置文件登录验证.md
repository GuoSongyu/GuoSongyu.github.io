---
layout: post
title: "springmvc整合spring security启用配置文件登录验证"
date: 2020-11-18 19:52:31 +0800
tags: 
description: 
---

作为spring全家桶之一的spring security，在配合springboot使用的时候，由于自动配置的原因，十分便捷。但是在springmvc上进行整合的时候，就需要进行很多的配置。该系列作为一个使用的记录帖，暂时不去深入讨论原理（PS：其实暂时我也不会）

![](/images/2020-11-18-1.jpg)

## jar包引入

{% highlight xml %}
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-core</artifactId>
  <version>5.4.1</version>
</dependency>
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-web</artifactId>
  <version>5.4.1</version>
</dependency>
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-config</artifactId>
  <version>5.4.1</version>
</dependency>
{% endhighlight %}

## web.xml配置

{% highlight xml %}
<filter>
	<filter-name>springSecurityFilterChain</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
	<filter-name>springSecurityFilterChain</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

## springsecurity.xml

对于用户验证，spring security提供两种模式，一种是非常简单的，将账号密码配置在XML文件中，使用默认的登陆页面；另一种是自定义用户验证逻辑和登录页面，先记录第一种简单的模式

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/security
        http://www.springframework.org/schema/security/spring-security.xsd">

    <!-- 设置放行 -->
    <security:http security="none" pattern="/static/**"></security:http>

    <!-- 设置拦截 auto-config="true"使用默认登录配置-->
    <security:http auto-config="true">
        <security:headers>
            <!-- iframe保护 -->
            <security:frame-options policy="SAMEORIGIN"></security:frame-options>
        </security:headers>
        <!-- 配置需要登录的url isAuthenticated代表带链接需要通过用户登录后才可以访问-->
        <security:intercept-url pattern="/demo/**" access="isAuthenticated()"></security:intercept-url>
        <!-- csrf配置 -->
        <security:csrf disabled="true"></security:csrf>
    </security:http>

    <!-- 配置认证管理 -->
    <security:authentication-manager>
        <security:authentication-provider>
            <security:user-service>
            	<!-- 直接写出用户名和密码 {noop}代表明文密码-->
                <security:user name="admin" password="{noop}admin" authorities="ROLE_ADMIN"></security:user>
            </security:user-service>
        </security:authentication-provider>
    </security:authentication-manager>
</beans>
{% endhighlight %}

## 验证

配置成功后，重启项目，/demo/**下链接都是一个简单的输出，正常可以直接访问，当我们用spring security进行管理之后，再访问的时候，就会跳到默认的登录页面，登录成功后，才能继续访问之前的页面

![](/images/2020-11-18-2.jpg)

输入配置文件中的账号密码

![](/images/2020-11-18-3.jpg)

登录成功，成功访问到我们原本要访问的页面

![](/images/2020-11-18-4.jpg)

对于第一种默认的登录方式，这样就配置成功了。感觉虽然简，但是实际中几乎是不会用到的。。。。

接下来还是回归我们常用的自定义模式吧

![](/images/2020-11-18-5.jpg)