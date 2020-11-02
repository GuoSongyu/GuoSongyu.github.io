---
layout: post
title: "idea搭建springMVC"
date: 2020-11-02 20:01:24 +0800
tags: java
description: 
---

记录一下maven搭建springMVC的过程

## 项目搭建

首先点击File-->New-->Project进入到项目创建窗口

![](/images/2020-11-02-1.jpg)

选择如下图的选项，点击Next按钮进入到下一步

![](/images/2020-11-02-2.jpg)

填写项目名称和本地保存目录，以及包路径、发布名称、版本号信息，设置完成后，点击__Finish__完成设置

![](/images/2020-11-02-3.png)

完成设置后，项目目录如下所示，只有webapp目录，java、resources、test目录需要我们自己进行创建

![](/images/2020-11-02-4.jpg)

在创建目录时候，idea会给出我们缺少目录的提示，直接创建就好，同时也会看到刚刚创建的java目录，显示出maven项目的蓝色

![](/images/2020-11-02-5.jpg)



## Jar包导入

项目建立完成后，导入springMVC需要的jar包。基本的jar包如下：

{% highlight xml %}
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <spring.version>5.2.8.RELEASE</spring.version>
    <!-- mysql驱动版本对应MySQL5.7 -->
    <mysql.version>5.1.47</mysql.version>
</properties>
<dependencies>
    <!-- spring相关 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!-- mysql驱动 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql.version}</version>
    </dependency>
    <!-- servlet -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
</dependencies>
{% endhighlight %}

__PS：我们可以在idea中设置maven自动导入xml中的jar包，顺便记得安装本地的maven，设置本地包仓库目录和setting.xml配置，把默认的包源换成国内源，比如下面的阿里的maven源__

{% highlight xml %}
<mirror>
	<id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
{% endhighlight %}


## 文件配置

由于resources目录是我们手动进行创建的，所以我们需要手动创建Spring的监听器配置文件（applicationContext.xml）和Spring的中央控制器配置文件（springmvc.xml）。同时还要在webapp下的web.xml文件中，进行两个配置文件的引入和配置。

### applicationContext.xml

> 其中__base-package__是我们的包路径，后面星号，表示路径下的包都遵循这个配置

{% highlight xml %}
<!-- spring容器扫描 -->
<context:component-scan base-package="com.bc.*">
    <!-- 不扫描spring mvc的controller -->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
{% endhighlight %}

### springmvc.xml

{% highlight xml %}
<mvc:annotation-driven/>
<!--spring mvc扫描controller不扫描service-->
<context:component-scan base-package="com.bc.apps.*">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
</context:component-scan>
{% endhighlight %}

配置完成后，前去__web.xml__进行配置

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
  <display-name>Md</display-name>
  <!-- 监听器配置文件 -->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <!-- 配置过滤器 解决中文乱码问题 -->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!-- SpringMVC的核心控制器 -->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
{% endhighlight %}

### 配置完成

上述配置完成后，去java目录下创建测试代码，进行测试项目是否搭建完成。

{% highlight java %}
@RequestMapping(value = "/")
@RestController
public class IndexController {

    @GetMapping(value = "index")
    public String index(){
        return "Hello,Spring~";
    }

}
{% endhighlight %}

对项目进行编译并启动，再地址栏中输入__http://domain/index__，网址中输出我们打印的文字，项目搭建成功！

![](/images/2020-11-02-6.jpg)



## 结尾

至此SpringMVC的初步搭建，已经完成，其实复杂程度到不是很高，只不过在配置和引入jar包的时候，稍微有一些多。

下一步，要进行视图解析器的配置，将使用[Thymeleaf][Thymeleaf]作为我们的视图解析器

![](/images/2020-11-02-7.jpg)

[Thymeleaf]:https://www.thymeleaf.org/