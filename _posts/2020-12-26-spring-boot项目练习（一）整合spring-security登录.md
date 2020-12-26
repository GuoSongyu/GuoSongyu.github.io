---
layout: post
title: "spring boot项目练习（一）整合spring security登录"
date: 2020-12-26 17:26:35 +0800
tags: java
description: 
---

最近看了好多spring boot的资料，也一直在学习，准备动手，自己写一个接口测试的系统来使用，正好也有这个需求。

自己会尽量的把开发过程中遇到的问题，或者一些东西记录下来，尽量不太监。。哈哈

对于用户登录，本来想使用之前用过的shiro，但是spring boot已经默认配置好了spring security。索性就直接使用security来管理用户

## 引入包

> 只需要简单的引入两个包就可以使用了

{% highlight java %}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
{% endhighlight %}

## 配置登录

> 对于security默认的登录，不在介绍了，引入包之后，再次访问会进入默认的登录页面，默认的登录名是user，密码会打印在控制台中
> 默认形式，基本不会使用在项目中，我们这里直接介绍自定义登录和响应的用法

### 控制器+页面

> 控制器和页面几乎不需要多余的操作
> 控制器只需要渲染视图
> 页面只需要进行提交就可以了
> 其他的认证操作都交给security去处理就可以了

控制器

{% highlight java %}
@Controller(value = "loginController")
@RequestMapping(value = "login")
public class LoginController extends BaseController {

    @RequestMapping(value = "index")
    public ModelAndView index(){
        return result("login/index");
    }

}
{% endhighlight %}

视图,这里是贴上了form表单和提交js事件,资源文件的引入之类的没有写入,防止无用的东西占用篇幅

{% highlight html %}
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <title>SB Admin 2 - Login</title>
</head>
<body class="bg-gradient-primary">
<form class="user">
    <div class="form-group">
        <input type="text" class="form-control form-control-user" name="username" placeholder="用户名">
    </div>
    <div class="form-group">
        <input type="password" class="form-control form-control-user" name="password" placeholder="密码">
    </div>
    <div class="form-group">
        <div class="custom-control custom-checkbox small">
            <input type="checkbox" class="custom-control-input" id="customCheck">
            <label class="custom-control-label" for="customCheck">Remember Me</label>
        </div>
    </div>
    <button type="button" id="submit" class="btn btn-primary btn-user btn-block">
        Login
    </button>
</form>
</body>
<script>
    $(function () {
        $("#submit").click(function () {
            submitForm("/login/index",function (message) {
                showSuccess(message);
            },function (message) {
                showWarning(message);
            });
        });
    })
</script>
</html>
{% endhighlight %}

### security自定义用户认证配置

> 需要继承UserDetailsService接口类,并实现其中的函数
> 对于User对象,权限字符串不可直接给null值
> 在UserService中,用户名验证成功后,user对象会在下一步,进行密码验证

{% highlight java %}
@Component
public class UserService implements UserDetailsService {

    @Autowired
    private AdminMapper adminMapper;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //根据用户名查询信息
        QueryWrapper<AdminEntity> sql = new QueryWrapper<>();
        sql.eq("username",s);
        AdminEntity adminEntity = adminMapper.selectOne(sql);
        if(adminEntity == null){
            throw new UsernameNotFoundException("用户名或密码错误！");
        }
        //用户验证通过，构建User对象，进行密码验证
        //用户权限不能为空，先生成默认权限
        List authz = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");
        User user = new User(adminEntity.getUsername(), adminEntity.getPassword(), authz);

        return user;
    }
}
{% endhighlight %}

### 自定义MD5密码验证

> 自定义密码验证类,需要继承PasswordEncoder接口类
> 继承后,实现encode和matches函数,前者是加密,后者是匹配
> 代码中得MD5是自定义的工具类

{% highlight java %}
public class MD5Password implements PasswordEncoder {

    @Override
    public String encode(CharSequence charSequence) {
        return MD5.encode(charSequence.toString());
    }

    @Override
    public boolean matches(CharSequence charSequence, String s) {
        return MD5.valid(charSequence.toString(),s);
    }
}
{% endhighlight java %}

### 自定义相应

> 大部分业务逻辑中,对于成功和失败的响应都需要有固定的格式,security当然也提供了这种功能
> 成功响应继承AuthenticationSuccessHandler接口类
> 失败响应继承AuthenticationFailureHandler接口类

{% highlight java %}
//成功
public class SuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        R r = R.ok().message("登录成功！");
        //转为jsonobject对象
        JsonObject jsonObject = JsonUtil.toJsonObject(r);
        //设置返回字符串和返回类型
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.setContentType("application/json;charset=utf-8");
        //打印字符串
        PrintWriter out = null;
        out = httpServletResponse.getWriter();
        out.write(jsonObject.toString());
        out.close();
    }
}

//失败
public class FailureHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        R r = R.error().code(200).message(用户名或密码错误！");
        //转为jsonobject对象
        JsonObject jsonObject = JsonUtil.toJsonObject(r);
        //设置返回字符串和返回类型
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.setContentType("application/json;charset=utf-8");
        //打印字符串
        PrintWriter out = null;
        out = httpServletResponse.getWriter();
        out.write(jsonObject.toString());
        out.close();
    }
}
{% endhighlight %}

### 整合上述配置

{% highlight java %}
@Configuration
@EnableWebSecurity
public class CustomerConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
        		//登录页面
                .loginPage("/login/index")
                //登录请求url,同页面中ajax请求url
                .loginProcessingUrl("/login/index")
                //设置自定义成功返回
                .successHandler(new SuccessHandler())
                //设置自定义失败返回
                .failureHandler(new FailureHandler())
                .permitAll();

        http.authorizeRequests()
        		//不需要认证就能访问的url
                .antMatchers("/login/**","/static/**")
                .permitAll()
                //其余url都需要经过认证
                .anyRequest().authenticated();

        //禁用csrf保护
        http.csrf().disable();
    }

    /**
     * 配置自定义加密类
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new MD5Password();
    }
}
{% endhighlight %}

配置完成后的使用效果

![](/images/2020-12-26-1.jpg)

![](/images/2020-12-26-2.jpg)

两次请求的结果都是由security中的自定义配置中返回