# :golf:Spring Boot 安全管理  #

<b id="t"></b>

:arrow_double_down:[Spring Security的基本配置](#a1)

:arrow_double_down:[Redis缓存](#a2)




<b id="a1"></b>

### :bowling:Spring Security的基本配置 ###

:arrow_double_up: [返回目录](#t)

Spring Boot针对Spring Security提供了自动化配置方案，因此可以使用Spring Security非常容易地整合进Spring Boot项目中，这也是在Spring Boot项目中使用Spring Security的优势。

**基本用法**

添加security依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

添加后即对项目中所有资源进行保护，访问需要输入密码，如下添加一个控制器：

```java
@RestController
public class index {
    @GetMapping("/index")
    public String index(){
        return  "Hello";
    }
}
```

访问控制器index需要权限认证会出现如下界面：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a27.png)

默认用户名为user密码在项目启动时会自动生成，在启动日志里可见密码：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a28.png)

**配置用户名和密码**

输入后即可访问。如果你不想使用默认的用户名与密码，可以在配置文件中添加配置信息：

```
spring.security.user.name=sang
spring.security.user.password=123
spring.security.user.roles=admin
```


接下就可以使用这个密码组登录了，而且这个还具有admin角色。

**基于内存的认证**

当然我们也可以自定义类继承自WebSecurityConfigureAdapter，进而实现对Spring Security更多的自定义配置，例如基于内存的认证，配置如下：

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
                .withUser("admin").password("123").roles("ADMIN","USER")
                .and()
                .withUser("sang").password("123").roles("USER");
    }
}
```

首先是该配置类继承WebSecurityConfigurerAdapter，并重写config方法，在该方法中添加两个用户，一个是sang，一个是admin，具备两个角色USER，ADMIN。使用任意一个都可以登录。

**HttpSecurity**

虽然现在可以实现认证功能，但是受保护的资源是默认，而且不能根据实际情况进行角色管理，如果要实现这些功能，就需要重写WebSecurityConfigurerAdapter 类中另一个方法，如下：

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
                .withUser("admin").password("123").roles("ADMIN","USER")
                .and()
                .withUser("sang").password("123").roles("USER");
    }
    @Override
    protected void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/system/**")                            //system/路径限制
                .hasRole("ADMIN")                                     //需要ADMIN角色权限
                .antMatchers("/index/**")
                .access("hasAnyRole('ADMIN','USER')")                 //含一种角色即可
                .antMatchers("/set")
                .access("hasRole('USER') and  hasRole('ADMIN')")      //必须同时含有两个角色
                .and()
                .formLogin()
                .loginProcessingUrl("/login")                         //开启接口/login表单验证
                .permitAll()
                .and()
                .csrf()                    //关闭csrf               
                .disable();     
    }
}
```

解释代码如上注释，接下来添加控制器路径：

```java
@RestController
public class index {
    @GetMapping("/index/hello")
    public String hello1(){
        return  "Hello User";
    }
    @GetMapping("/system/hello")
    public  String hello2(){
        return "Hello Admin";
    }
    @GetMapping("/hello")
    public String hello(){
        return "Hello Tourist";
    }
    @GetMapping("/set")
    public String hello3(){
        return "Hello Admin and user";
    }
}
```

这下就可以实现不同的角色实现不同的访问，其中没有被阻止的访问的资源可以直接访问，如果想全部都需要登录访问，可以添加如下代码：

```java
 @Override
    protected void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/system/**")
                .hasRole("ADMIN")
                .antMatchers("/index/**")
                .access("hasAnyRole('ADMIN','USER')")
                .antMatchers("/set")
                .access("hasRole('USER') and  hasRole('ADMIN')")
                .anyRequest()             //其他访问也需要登录
                .authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
```

**登录表单详细配置**

迄今为止，登录表单一直使用Spring Security提供的页面，登录成功后也是默认的页面跳转，但是前后端分离正在成为企业级应用开发的主流，在前后端分离方式中，前后端的数据交互通过JSON进行，这时登录界面不再是界面跳转，而是一段json提示。要实现这些功能，就继续上面的配置：

```java

```





