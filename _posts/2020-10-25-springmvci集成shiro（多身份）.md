---
layout: post
title: "springMVCi集成shiro（多身份）"
date: 2020-10-25 21:08:58 +0800
tags: java
description: 
---

最近的生活，真是一言难尽。。。不多说了，直接进入正题吧。

以前做项目的时候，对于用户登录这边，基本用的就是拦截器，多身份得话，就在拦截器中进行判断。在浏览帖子的时候，看到了关于shiro的介绍，又正巧要做一个新的项目。于是在新的项目中，用户的权限管理就改用shiro去控制。

关于shiro，就不多介绍了，一个轻量的用户权限管理包；项目的要求也很简单。后台管理员和前台用户两个身份分别进行登录，访问各自的url。

首先，通过maven引入shiro的jar包
{% highlight xml %}
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
	<version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>net.mingsoft</groupId>
    <artifactId>shiro-freemarker-tags</artifactId>
    <version>1.0.0</version>
</dependency>
{% endhighlight %}

然后，创建spring-shiro.xml文件，对shiro进行配置，需要注意的几个配置点：

> 多身份用户，对应多个realm对象
>
> 不同身份下的身份验证规则（FormAuthenticationFilter类或者重写的继承类）
>
> shiro的主filter配置（ShiroFilterFactoryBean），包括filters（过滤规则）、filterChainDefinitions（设置url对应的过滤规则）

配置文件如下：
{% highlight xml %}
<!-- 配置filter -->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"></property>
    <property name="filters">
        <map>
            <entry key="adminFilter" value-ref="AdminFilter"></entry>
            <entry key="userFilter" value-ref="UserFilter"></entry>
            <entry key="adminPermissionFilter" value-ref="AdminPermissionFilter"></entry>
        </map>
    </property>
    <property name="filterChainDefinitions">
        <value>
            /static/**=anon
            /admin/base/login/*=anon
            /index=anon
            /admin/**=adminFilter,adminPermissionFilter
            /ucenter/auth/**=anon
            /ucenter/**=userFilter
        </value>
    </property>
</bean>

<!-- 配置securityManager -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <!-- 引入ModularRealmAuthenticator类 -->
    <property name="authenticator" ref="CustomerModularRealmAuthenticator"></property>
    <!-- 配置多个realm对象 -->
    <property name="realms">
        <list>
            <ref bean="AdminRealm"></ref>
            <ref bean="TeacherRealm"></ref>
            <ref bean="UserRealm"></ref>
        </list>
    </property>
</bean>
<!-- 自定义realm -->
<!-- 后台对象realm -->
<bean id="AdminRealm" class="com.bc.core.shiro.realm.AdminRealm"></bean>
<!-- 教师端realm -->
<bean id="TeacherRealm" class="com.bc.core.shiro.realm.TeacherRealm"></bean>
<bean id="UserRealm" class="com.bc.core.shiro.realm.UserRealm"></bean>

<!-- 配置filter对应的登陆页面 -->
<bean id="AdminFilter" class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
    <property name="loginUrl" value="/admin/base/login/index"></property>
</bean>
<bean id="UserFilter" class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
    <property name="loginUrl" value="/ucenter/auth/login"></property>
</bean>

<!-- 自定义后台权限验证filter -->
<bean id="AdminPermissionFilter" class="com.bc.core.shiro.authz.AdminPermissionFilter">
    <property name="loginUrl" value="/admin/base/login/index"></property>
    <property name="unauthorizedUrl" value="/admin/main/error/authz"></property>
</bean>

<!-- 自定义ModularRealmAuthenticator类，选则当前要使用的realm -->
<bean id="CustomerModularRealmAuthenticator" class="com.bc.core.shiro.authc.CustomerRealmAuthenticator">
    <property name="authenticationStrategy">
        <!-- 配置认证策略，只要有一个Realm认证成功即可，并且返回所有认证成功信息 -->
        <bean class="org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy"></bean>
    </property>
</bean>
<!-- 配置 Bean 后置处理器: 会自动的调用和 Spring 整合后各个组件的生命周期方法. -->
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
{% endhighlight %}

在用户登录的时候，需要把用户的登录名、密码传入到UsernamePasswordToken类中，由于我们是两种身份不同的用户，所以需要对UsernamePasswordToken类进行拓展，增加loginType参数
{% highlight java %}
//UsernamePasswordToken拓展类
public class CustomerToken extends UsernamePasswordToken {

    private TokenTypeEnum loginType;

    public CustomerToken(String username,String password,TokenTypeEnum loginType){
        super(username,password);
        this.loginType = loginType;
    }

    public CustomerToken(String username,String password,TokenTypeEnum loginType,boolean rememberme){
        super(username,password,rememberme);
        this.loginType = loginType;
    }

    public TokenTypeEnum getLoginType() {
        return loginType;
    }

    public void setLoginType(TokenTypeEnum loginType) {
        this.loginType = loginType;
    }

}

//loginType枚举类
public enum TokenTypeEnum {

    Admin("Admin"),User("User");

    private String type;

    TokenTypeEnum(String type){
        this.type = type;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

}
{% endhighlight %}

这样在登录时候原本调用的UserPasswordToken可以改为新的自定义类，并增加LoginType参数
{% highlight java %}
Subject subject = SecurityUtils.getSubject();
UsernamePasswordToken usernamePasswordToken = new CustomerToken(username,password,TokenTypeEnum.Admin);
subject.login(usernamePasswordToken);
{% endhighlight %}

不同身份对应的用户信息验证，在自定义的realm类中，进行实现
{% highlight java %}
//继承AuthorizingRealm类，实现自己的逻辑
public class UserRealm extends AuthorizingRealm {

    @Autowired
    private IUserDao userDao;

    @Autowired
    private IUserBaseDao userBaseDao;

    @Autowired
    private SessionDAO sessionDAO;

    //权限注入函数
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //用户认证函数
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //获取登录用户名
        String username = (String)authenticationToken.getPrincipal();
        UserEntity userEntity = userDao.getOne(MapUtil.getMap("mobile",username,"isdel",0));
        if(userEntity == null){
            throw new UnknownAccountException("用户名或密码错误！");
        }
        //验证密码是否正确
        char[] pass = (char[])authenticationToken.getCredentials();
        String password = String.valueOf(pass);
        if(!userEntity.getPassword().equals(MD5.encryption(password))){
            throw new IncorrectCredentialsException("用户名或密码错误！");
        }
        //验证成功，赋值对象属性
        User user = new User();
        BeanUtils.copyProperties(userEntity,user);
        UserBaseEntity userBaseEntity = userBaseDao.getOne(MapUtil.getMap("userid",userEntity.getId()));
        if(userBaseEntity!= null){
            user.setName(userBaseEntity.getName());
            user.setAvatar(userBaseEntity.getAvatar());
        }

        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(admin,pass,getName());

        return simpleAuthenticationInfo;
    }
}
{% endhighlight %}

至此，多身份的登录流程就写完了，但是，突然发现在同一个浏览器登录两个身份的时候，前一个SimpleAuthenticationInfo中的principal对象会报错，后一个把前一个替换掉了，再去访问前一个的话，就会出现对象类型错误。

研究了一阵子。。虽然说是解决了，但是不知道其他人是怎么弄的，网上找了好多，也没找到类型的问题（是我找的不对吗）。
![](/images/2020-10-26-1.jpg)

下一篇文章，去说一下我的解决办法。