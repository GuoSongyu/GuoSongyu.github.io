---
layout: post
title: "springmvc整合spring security数据库进行验证"
date: 2020-12-04 18:26:43 +0800
tags: java
description: 
---

之前介绍了security最快捷的一种登录方式，配置文件验证。将要登陆的用户、角色信息写在配置文件中，进行登录验证。但是在实际项目中，基本不会这么去使用，对于用户的认证信息是要从数据库中进行查找和验证。

这次要记录的就是通过自定义的用户查询逻辑，对用户进行认证

## XML配置

首先要将之前的security:authentication-manager配置进行修改

{% highlight xml %}
<security:authentication-manager id="demo">
    <security:authentication-provider user-service-ref="userService">
        <!-- 加密方式 -->
        <security:password-encoder ref="passwordEncoder"></security:password-encoder>
    </security:authentication-provider>
</security:authentication-manager>
<!-- 自定义的用户呀验证类 -->
<bean id="userService" class="com.bc.common.security.authc.CustomerUserDetailService"></bean>
<!-- 自定义密码验证类（security提供了md5的加密，这里是为了运用一下自定义密码验证类的配置） -->
<bean id="passwordEncoder" class="com.bc.common.security.password.MD5PasswordEncoder"></bean>
{% endhighlight %}

同时还需要对之前的http配置中增加登录的页面、请求url等。对于登录页面的编写，就不在说明了，就是正常的页面渲染而已

{% highlight xml %}
<security:http auto-config="true" authentication-manager-ref="demo">
    <security:headers>
        <!-- iframe保护 -->
        <security:frame-options policy="SAMEORIGIN"></security:frame-options>
    </security:headers>
    <!-- 配置需要登录的url -->
    <security:intercept-url pattern="/demo/**" access="hasAnyAuthority('ADMIN')" />
    <!-- 配置自定义登录 -->
    <!--
        login-page 登录页url
        login-processing-url 提交登录的访问url。
        default-target-url 登录后访问页面
    -->
    <security:form-login
            login-page="/backend/enter/login/index"
            login-processing-url="/backend/enter/login/ajaxurl"
            default-target-url="/demo/data/index"/>
    <!-- csrf配置 -->
    <security:csrf disabled="true"></security:csrf>
</security:http>
{% endhighlight %}

## 编写验证类

对于用户验证类，我们要实现security中的UserDetailsService接口中的loadUserByUsername函数。来获取用户的信息

{% highlight java %}
public class CustomerUserDetailService implements UserDetailsService {

    @Autowired
    private AdminMapper adminMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //自定义用户查询逻辑。通过用户名查询
        QueryWrapper<AdminEntity> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username",username)
                .eq("status",1);
        AdminEntity adminEntity = adminMapper.selectOne(queryWrapper);
        if(adminEntity == null){
            throw new UsernameNotFoundException("用户名或密码错误！");
        }
        //存在请求用户名下的用户信息，赋予默认的权限。创建User对象时权限不能为空
        List<GrantedAuthority> a = AuthorityUtils.commaSeparatedStringToAuthorityList("ADMIN");

        //这里将用户名，密码，权限传入User中。security会自动把请求密码加密和用户中的密码进行比对
        User user = new User(username,adminEntity.getPassword(),a);
        return user;
    }
}
{% endhighlight %}

密码验证类，实现PasswordEncoder接口中的encode（加密）、matches（比对）函数

{% highlight java %}
public class MD5PasswordEncoder implements PasswordEncoder {

    @Override
    public String encode(CharSequence charSequence) {
        return Md5.encryp((String)charSequence);
    }

    @Override
    public boolean matches(CharSequence charSequence, String s) {
        return Md5.verify((String)charSequence,s);
    }
}
{% endhighlight %}

上述工作做完之后，再次访问我们的/demo/data/index路径，发现自动跳转到我们自定义的登录页面中

![](/images/2020-12-04-1.jpg)

输入数据库中的用户名和密码，成功进入到页面中

![](/images/2020-12-04-2.jpg)

springmvc + spring security的登录整合就完成了，对比之前我们自己写的登录验证、跳转的逻辑，省去了大把的时间。降低了工作量和难度

![](/images/2020-12-04-3.jpg)