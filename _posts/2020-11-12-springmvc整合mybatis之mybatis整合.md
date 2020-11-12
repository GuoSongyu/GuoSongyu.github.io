---
layout: post
title: "springmvc整合mybatis之mybatis整合"
date: 2020-11-12 20:49:53 +0800
tags: java
description: 
---

上一篇文章中，已经说明了需要引入的jar包

{% highlight xml %}
<!-- mybatis通用Mapper类 -->
<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper</artifactId>
  <version>4.1.5</version>
</dependency>
<!-- mybatis类 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.5.3</version>
</dependency>
<!-- mybatis+spring整合包 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.5</version>
</dependency>
{% endhighlight %}

主要是下面两个，第一个通用mapper类只是为了更方便的去使用，里面已经集成好了一些常用的函数。我们不需要重复封装，直接调用就可以了

接下来需要对mapper的一些参数进行配置。我们在resources目录下，创建springmapper.xml文件，配置如下：

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 加载properties文件中的内容 -->
    <context:property-placeholder location="classpath:conf/jdbc.properties"/>
    <!-- 扫描的包路径 -->
    <context:component-scan base-package="com.bc.*" />
    <!-- 配置连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="initialPoolSize" value="${jdbc.initialSize}"></property>
        <property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
        <property name="minPoolSize" value="${jdbc.minPoolSize}"></property>
    </bean>
    <!-- mybatis整合 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <!-- 映射路径 -->
        <property name="mapperLocations" value="classpath:mapper/*"></property>
    </bean>
    <!-- 配置扫描器，这里使用了通用mapper中的类 -->
    <bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 扫描包路径,如果需要扫描多个包中间用半角逗号隔开 -->
        <property name="basePackage" value="com.bc.dao"/>
        <!-- 这边不能使用ref="sqlSessionFactory"原因是因为上面加载配置文件导致这边引用会报错 -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    </bean>
    <!-- 事务管理器 对mybatis操作数据库事务控制，spring使用jdbc的事务控制类 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
{% endhighlight %}

配置完成后，在web.xml引入刚才的配置文件

{% highlight xml %}
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>
	  	classpath:applicationContext.xml,
	  	<!-- 引入mybatis配置 -->
	  	classpath:springmapper.xml
	</param-value>
</context-param>
{% endhighlight %}

完成后，配合之前的mybatis-generator，进行测试，先随便创建一个表
![](/images/2020-11-12-1.png)

进行自动生成，生成如下几个文件，分别是Mapper查询接口（继承tk通用mapper）、实体类、xml映射文件
![](/images/2020-11-12-2.png)

创建控制器，并调用Mapper接口类，进行查询（省略了Serivce类）

{% highlight java %}
@RestController
@RequestMapping(value = "data")
public class DataController {

    @Autowired
    private OneMapper oneMapper;

    @GetMapping(value = "index")
    public One index(){
        One one = oneMapper.selectByPrimaryKey(1);

        return one;
    }

}
{% endhighlight %}

我们运用的selectByPrimaryKey函数就是通用类其中的一个，通用类中包含了很多常用的函数,大大加快的开发进度，节约了很多时间

![](/images/2020-11-12-3.png)

完成后，访问链接，读取数据库信息成功，mybatis配置成功！

![](/images/2020-11-12-4.png)

springMVC整合mybatis，至此就结束了，后续还有继续记录mybatis的一些用法（用到哪里记录哪里）

![](/images/2020-11-12-5.jpg)
