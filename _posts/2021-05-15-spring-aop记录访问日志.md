---
layout: post
title: "spring aop记录访问日志 "
date: 2021-05-15 20:45:59 +0800
tags: java
description: 
---

spring框架最大的特点之一就是AOP（Aspect Oriented Programming）。也就是面向切面编程。

可以在对原本的逻辑不产生修改的情况下，对原来的函数进行增强。大大降低代码的耦合度，提高程序的可重用性。可以说是面向对象思想下的一个衍生，但是在某些情况下，要优于面向对象。比如日志、鉴权等情况下，使用AOP，可以很轻松的进行实现。本文对系统访问日志采用AOP的方式进行开发。

## 引入依赖
不论是spring boot 还是 springMVC都需要引用下面的两个包
{% highlight java %}
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.9.6</version>
</dependency>
{% endhighlight %}

在spring boot中还需要引入
{% highlight java %}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
{% endhighlight %}

在springMVC中，需要引入
{% highlight java %}
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
{% endhighlight %}

## 创建切面类

一个切面类需要定义一个切点（一般情况下是某个函数，也可能是某个属性等）。再根据切点进行增强处理。对于系统访问日志来说，我做了如下两个后置增强
{% highlight java %}
@Aspect
@Component
public class LogAspect {

	private HttpServletRequest request;

	//创建切点。所有的控制器
	@Pointcut("execution(* com.bc.center.controller.*.*Controller.*(..))")
	public void cutPoint(){
	}

	/**
	 * 前置赋值
	 */
	@Before("cutPoint()")
	public void beforeController(){
	    request = getRequest();
	}

	/**
	 * 请求成功记录
	 */
	@AfterReturning("cutPoint()")
	public void requestSuccess(){
	    SystemLogDTO systemLogDTO = new SystemLogDTO();
	    systemLogDTO.setIsError(0);
	    saveLog(systemLogDTO);
	}

	/**
	 * 请求记录失败
	 */
	@AfterThrowing(value = "cutPoint()",throwing = "e")
	public void requestFail(Throwable e){
	    SystemLogDTO systemLogDTO = new SystemLogDTO();
	    systemLogDTO.setIsError(1);
	    //获取异常位置
	    Map errorDetail = new HashMap(2);
	    StackTraceElement[] stack = e.getStackTrace();
	    for (StackTraceElement stackTraceElement : stack){
	        String className = stackTraceElement.getClassName();
	        int lineNumber = stackTraceElement.getLineNumber();
	        if(className.contains("com.bc") && lineNumber > 0){
	            //存在当前项目目录异常，记录异常类和行数
	            errorDetail.put("className",className);
	            errorDetail.put("lineNumber",lineNumber);
	        }
	    }
	    if(errorDetail.size() > 0){
	        errorDetail.put("message",e.getMessage());
	        //记录成功，保存到对象中
	        systemLogDTO.setErrorDetail(JsonUtil.toJSONString(errorDetail));
	    }

	    saveLog(systemLogDTO);
	}
}
{% endhighlight %}

DTO类的属性如下:
{% highlight java %}
public class SystemLogDTO {

    /**
     * 请求url
     */
    private String uri;

    /**
     * 请求方式
     */
    private String requestType;

    /**
     * 请求参数
     */
    private String params;

    /**
     * 是否是抛出异常时的请求
     */
    private Integer isError;

    /**
     * 存在异常时，记录应用内的
     */
    private String errorDetail;

}
{% endhighlight %}

@Aspect：声明当前类是切面类。

@Pointcut：设置切点。通过execution关键字进行设置
> * com.bc.center.controller.*.*Controller.*(..))
> 第一个 * 参数是返回值 *表示任何返回值
> 第二个 com.bc.center.controller.*.*Controller.* 是匹配得切点路径
> 第三个 (..) 是请求参数。 ..表示任意参数

@Before：在切点之前执行
@After：在切点之后执行
@AfterReturn：切点正常返回时执行
@AfterThrowing：切点抛出异常时执行

这里我使用了**@AfterReturn**和**@AfterThrowing**，为了区分记录正常返回和异常返回的日志。

## 小结

以往采用的方案是以拦截器的方式进行记录。需要配置拦截器以及放行的url，相对要繁琐一些。使用切面的话，可以以所有的控制器为切点，只需要一个切面类，就可以实现日志的访问记录，最重要的是对原来的逻辑没有任何入侵。