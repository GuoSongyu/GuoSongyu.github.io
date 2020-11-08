---
layout: post
title: "springmvc整合mybatis之mybatis generator自动生成"
date: 2020-11-08 08:41:26 +0800
tags: java
description: 
---

当我们使用hibernate作为数据持久层的时候，对于实体类的生成在Idea中可以使用__Persistence__菜单进行实体类的自动生成(Persistence->Generate Persistence Mapping->By Database Schema)

![](/images/2020-11-08-1.jpg)

当使用mybatis的时候，hibernate的Persistence菜单不能使用，但是mybatis提供了另一个非常好用的插件__mybatis-generator__，可以通过maven插件的方式，一键生成实体类、接口类、Xml映射等

## maven配置

jar包引入
{% highlight xml %}
<dependency>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-core</artifactId>
  <version>1.4.0</version>
</dependency>
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

配置maven插件
{% highlight xml %}
<plugin>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-maven-plugin</artifactId>
  <version>1.3.6</version>
  <configuration>
    <configurationFile>
      ${basedir}/src/main/resources/generator/generatorConfig.xml
    </configurationFile>
    <overwrite>true</overwrite>
    <verbose>true</verbose>
  </configuration>
  <dependencies>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql.version}</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>tk.mybatis</groupId>
      <artifactId>mapper</artifactId>
      <version>4.1.5</version>
    </dependency>
  </dependencies>
</plugin>
{% endhighlight %}

## 配置generator文件

在配置文件中，我们可以配置数据库链接、需要生成的类以及生成目录等信息，直接上配置吧，对应的配置说明都在文件注释中
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 引入数据库配置文件 -->
    <properties resource="conf/jdbc.properties"></properties>
    <context id="mySqlContext" defaultModelType="flat">
        <!-- 自动处理mysql中的关键字 -->
        <property name="autoDelimitKeywords" value="true"/>
        <!-- sql关键字中的分隔符 -->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <!-- 编码 -->
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 继承mybatis通用类 -->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
            <property name="forceAnnotation" value="true"/>
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>
        </plugin>
        <!-- 阻止自动生成时间戳和MBG注释，会影响版本控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
        </commentGenerator>
        <!-- 数据库信息 -->
        <jdbcConnection
                driverClass="${jdbc.driver}"
                connectionURL="${jdbc.url}"
                userId="${jdbc.username}"
                password="${jdbc.password}"
        >
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
        <!-- 生成实体类的位置 -->
        <javaModelGenerator targetPackage="com.bc.entity" targetProject="src/main/java">
        	<!-- 如果设置为true会在targetPackage的基础上，根据数据库的schema再生成一层package -->
            <property name="enableSubPackages" value="false"/>
            <!-- 在getter方法中，对String类型字段调用trim()方法 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- 如果用XML格式，该配置需要配置 -->
        <sqlMapGenerator targetPackage="mapper"  targetProject="src\main\resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>
        <!--
        mapper的生成形式
        XMLMAPPER：XML形式，不需要mapper注解
         -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.bc.dao" targetProject="src/main/java"></javaClientGenerator>
        <!-- 配置要生成的表 -->
        <table tableName="bc_one" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false">
        	<!-- 设置mysql主键 -->
            <generatedKey column="id" sqlStatement="MySql"></generatedKey>
            <!-- 匹配生成的类型名称，替换掉数据中的前缀 -->
            <domainObjectRenamingRule searchString="^Bc" replaceString=""></domainObjectRenamingRule>
        </table>
    </context>
</generatorConfiguration>
{% endhighlight %}

## 使用方式

在项目根目录中执行：
{% highlight cli %}
mvn mybatis-generator:generate
{% endhighlight %}

执行成功后，会显示如下界面

![](/images/2020-11-08-2.png)

如果执行失败的话，可以在上述命令执行时候在结尾增加__-X__参数。会显示详细的执行日志

完成，就会发生已经生成了实体、接口类和xml映射文件

![](/images/2020-11-08-3.jpg)

每个文件内容如下：

### OneMapper.java

{% highlight java %}
package com.bc.dao;

import com.bc.entity.One;
import tk.mybatis.mapper.common.Mapper;

public interface OneMapper extends Mapper<One> {

	//继承了通用类，可以实现一些通用的函数，这里可以编写自定义查询函数...

}
{% endhighlight %}


### One.java

{% highlight java %}
package com.bc.entity;

import javax.persistence.*;

@Table(name = "`bc_one`")
public class One {
    @Id
    @Column(name = "`id`")
    @GeneratedValue(strategy = GenerationType.IDENTITY, generator = "SELECT LAST_INSERT_ID()")
    private Integer id;

    @Column(name = "`name`")
    private String name;

    @Column(name = "`age`")
    private Integer age;

    /**
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * @return name
     */
    public String getName() {
        return name;
    }

    /**
     * @param name
     */
    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    /**
     * @return age
     */
    public Integer getAge() {
        return age;
    }

    /**
     * @param age
     */
    public void setAge(Integer age) {
        this.age = age;
    }
}
{% endhighlight %}

### OneMapper.xml

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bc.dao.OneMapper">
  <resultMap id="BaseResultMap" type="com.bc.entity.One">
    <!--
      WARNING - @mbg.generated
    -->
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="age" jdbcType="INTEGER" property="age" />
  </resultMap>
  <sql id="Base_Column_List">
    <!--
      WARNING - @mbg.generated
    -->
    id, `name`, age
  </sql>
</mapper>
{% endhighlight %}

至此mybatis-generator的配置和用法就记录完成了，感觉上确实方便了很多，对于Dao类生成来说，不需要重复的去建立文件了，直接通过命令行直接生成就好了
![](/images/2020-11-08-4.jpg)

下一篇文章介绍mybatis与springmvc的整合和使用