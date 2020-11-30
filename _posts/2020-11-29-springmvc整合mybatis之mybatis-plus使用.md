---
layout: post
title: "springmvc整合mybatis之mybatis plus使用"
date: 2020-11-29 20:52:26 +0800
tags: java
description: 
---

之前记录了mybatis-generator和通用mapper的使用，使我们在开发过程中减少了很多重复的创建工作。在使用过程中只需要一个命令行就搞定了。

但是在使用过程中，由于命令行，实际也是读取的xml配置。所以如果要想每次只生成指定表的话，就需要去修改xml文件。或者直接整个数据库生成，但是这样难免会覆盖之前修改过的文件，很不方便。加上通用mapper虽然集成了一些，但是对于一些条件查询来说，还是不太方便

![](/images/2020-11-29-1.jpg)

mybatis-plus作为mybatis的增强工具来说，很好的解决了以上问题，可以通过灵活的配置和代码来实现，自动生成。同时也提供了BaseMapper中也提供了很多的函数。大大方便了开发者

## 包引入

mybatis-plus包自动关联了mybatis包，所以不需要重复引入，把原有的mybatis包在pom.xml中去除。只需要引入新的jar即可

{% highlight xml %}
<!-- mybatis-plus -->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus</artifactId>
  <version>3.3.2</version>
</dependency>
<!-- mybatis-plus代码生成器。自动生成实体、mapper等类 -->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-generator</artifactId>
  <version>3.3.2</version>
</dependency>
{% endhighlight %}

## 修改配置文件

原来的配置文件大部分不需要修改，只需要_sqlSessionFactory_对应的类换成mybatis-plus中类即可

{% highlight xml %}
<!-- 替换成mybatisplus中的类 -->
<bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <!-- 映射路径 -->
    <property name="mapperLocations" value="classpath:mapper/*"></property>
</bean>
{% endhighlight %}

## 自动生成

mybatis-plus中对于自动生成，提供了非常自由的配置，默认的话，是会生成controller、service（实现类和接口类）、entity、mapper这五个文件。刚好涵盖了除了视图之外的数据表映射，数据库查询，业务逻辑，控制器几个部分。而且生成的文件也可以随意配置。具体的使用方法在[官网][site]可以找到。这里记录了我自己使用的配置方法

### 全局配置

{% highlight java %}
GlobalConfig gc = new GlobalConfig();
//项目的根目录
String projectPath = "E:\\work\\app";
//生成文件的目录
gc.setOutputDir(projectPath + "/src/main/java");
//作者
gc.setAuthor("MuYu");
//覆盖
gc.setFileOverride(true);
gc.setBaseResultMap(true);
//设置xml文件后缀
gc.setXmlName("%sMapper");
//设置实体类后缀
gc.setEntityName("%sEntity");
//生成完成后，是否自动打开目录，实际开发过程中实在没有必要
gc.setOpen(false);
{% endhighlight %}

### 数据库配置

{% highlight java %}
DataSourceConfig dc = new DataSourceConfig();
//设置数据库驱动
dc.setDriverName("com.mysql.cj.jdbc.Driver");
//数据库用户名
dc.setUsername("root");
//数据库密码
dc.setPassword("root");
//数据库地址，在地址后加上时区、编码等可以防止一些错误
dc.setUrl("jdbc:mysql://127.0.0.1:3306/mysql?serverTimezone=UTC&useSSL=false&characterEncoding=UTF-8");
{% endhighlight %}

### 包路径设置

{% highlight java %}
PackageConfig pc = new PackageConfig();
//设置包路径
pc.setParent("com.demo");
//设置生成文件目录
//实体目录，结合上面的包路径，就是com.demo.entity
pc.setEntity("entity");
//mapper接口目录，对应的就是com.demo.common.mappers
pc.setMapper("common.mappers");
{% endhighlight %}

### 输出配置

{% highlight java %}
//设置XML文件的目录和文件名等信息
InjectionConfig cfg = new InjectionConfig() {
    @Override
    public void initMap() {

    }
};

// 自定义输出配置
final String projectPath = this.projectPath;
String templatePath = "/templates/mapper.xml.ftl";
List<FileOutConfig> focList = new ArrayList<>();
focList.add(new FileOutConfig(templatePath){

    @Override
    public String outputFile(TableInfo tableInfo) {
        // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
        return projectPath + "/src/main/resources/mybatis/" + tableInfo.getMapperName() + StringPool.DOT_XML;
    }
});
cfg.setFileOutConfigList(focList);
{% endhighlight %}

## 配置生成策略

{% highlight java %}
StrategyConfig strategy = new StrategyConfig();
//表前缀
strategy.setTablePrefix("bc_");
//设置驼峰命名规则
strategy.setNaming(NamingStrategy.underline_to_camel);
strategy.setColumnNaming(NamingStrategy.underline_to_camel);
//生成的具体表名，数组格式
strategy.setInclude(["bc_test_one","bc_test_two"]);
{% endhighlight %}

## 自定义模板

{% highlight java %}
TemplateConfig templateConfig = new TemplateConfig();
//设置不启用的生成文件。对应的参数在启用生成后，不会出现
templateConfig.disable(TemplateType.SERVICE, TemplateType.CONTROLLER);
//自定义生成模板路径，相对于resources
templateConfig.setEntity("template/entity.java");
templateConfig.setMapper("template/mapper.java");
templateConfig.setXml(null);
{% endhighlight %}

## 执行生成

上述配置完成后，由AutoGenerator类来执行生成

{% highlight java %}
AutoGenerator ag = new AutoGenerator();
//导入配置
//全局配置
ag.setGlobalConfig(createGlobalConfig());
//数据库配置
ag.setDataSource(createDataSourceConfig());
//包路径配置
ag.setPackageInfo(createPackageConfig());
//输出配置
ag.setCfg(createInjectionConfig());
//生成策略
ag.setStrategy(createStrategyConfig());
//模板配置
ag.setTemplate(createTemplateConfig());
//模板引擎，freemarker是默认提供的模板引擎之一
ag.setTemplateEngine(new FreemarkerTemplateEngine());
//生成
ag.execute();
{% endhighlight %}

### 说明

生成的mapper自动继承BaseMapper接口，BaseMapper中提供了很多便捷的查询接口。如：根据Map集合作为查询条件，使用mybatis提供的QueryWrapper查询条件构造类。可以实现大部分的查询。不需要再次编写sql语句

![](/images/2020-11-30-1.jpg)

QueryWrapper的简单示例，可以使用大部分的sql逻辑，如 or、and、have、in、not in等等

![](/images/2020-11-30-2.jpg)

查询成功

![](/images/2020-11-30-3.jpg)


[site]: https://baomidou.com/