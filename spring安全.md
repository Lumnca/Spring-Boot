# :golf:Spring Boot 安全管理  #

<b id="t"></b>

:arrow_double_down:[Spring Security的基本配置](#a1)

:arrow_double_down:[基于数据库的验证](#a2)

:arrow_double_down:[高级配置](#a3)

:arrow_double_down:[OAuth2](#a4)

:arrow_double_down:[Spring Boot整合Shiro](#a5)


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
                .antMatchers("/system/**")
                .hasRole("ADMIN")
                .antMatchers("/index/**")
                .access("hasAnyRole('ADMIN','USER')")
                .antMatchers("/set")
                .access("hasRole('USER') and  hasRole('ADMIN')")
                .and()
                .formLogin()
                .loginPage("/login_page")
                .loginProcessingUrl("/login")
                .usernameParameter("name")
                .passwordParameter("passwd")
                //成功处理
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response
                    , Authentication auth)throws IOException{
                        Object principal = auth.getPrincipal();
                        response.setContentType("application/json;charset=utf-8");
                        PrintWriter out = response.getWriter();
                        response.setStatus(200);
                        Map<String,Object> map = new HashMap<>();
                        map.put("status",200);
                        map.put("msg",principal);
                        ObjectMapper om = new ObjectMapper();
                        out.write(om.writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                    //登录失败处理
                }).failureHandler(new AuthenticationFailureHandler(){
            @Override
            public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
                response.setContentType("application/json;charset=utf-8");
                PrintWriter out = response.getWriter();
                response.setStatus(401);
                Map<String,Object> map = new HashMap<>();
                map.put("status",401);
                if(e instanceof LockedException){
                    map.put("msg","账户被锁定，登录失败");
                }
                else  if(e instanceof BadCredentialsException){
                    map.put("msg","用户名或密码错误，登录失败");
                }
                else  if(e instanceof DisabledException){
                    map.put("msg","账户被禁用，登录失败");
                }
                else  if(e instanceof AccountExpiredException){
                    map.put("msg","账户已过期，登录失败");
                }
                else  if(e instanceof CredentialsExpiredException){
                    map.put("msg","密码已过期，登录失败");
                }
                else {
                    map.put("msg","未知错误，登录失败，请联系管理人员");
                }
                ObjectMapper om = new ObjectMapper();
                out.write(om.writeValueAsString(map));
                out.flush();
                out.close();
            }
        }).permitAll().and().csrf().disable();  //必须含有
    }
}
```

像这样就可以实现JSON转递，你可以在login_page提供一个login接口的表单，如下：

```html
<form action="/login" method="post">
    <label>账号</label>
    <input type="text" placeholder="输入账号" name="name"><br>
    <label>密码</label>
    <input type="password" name="passwd"><br>
    <input type="submit" value="登录">
</form>
```


让后登录成功后就会返回JOSN字符串。如果失败也会有相应的错误提示。

**登录注销配置**

同样的添加配置：

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
                .antMatchers("/system/**")
                .hasRole("ADMIN")
                .antMatchers("/index/**")
                .access("hasAnyRole('ADMIN','USER')")
                .antMatchers("/set")
                .access("hasRole('USER') and  hasRole('ADMIN')")
                .and()
                .formLogin()
                .loginPage("/login_page")
                .loginProcessingUrl("/login")
                .usernameParameter("name")
                .passwordParameter("passwd")
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response
                    , Authentication auth)throws IOException{
                        Object principal = auth.getPrincipal();
                        response.setContentType("application/json;charset=utf-8");
                        PrintWriter out = response.getWriter();
                        response.setStatus(200);
                        Map<String,Object> map = new HashMap<>();
                        map.put("status",200);
                        map.put("msg",principal);
                        ObjectMapper om = new ObjectMapper();
                        out.write(om.writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                }).failureHandler(new AuthenticationFailureHandler(){
            @Override
            public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
                response.setContentType("application/json;charset=utf-8");
                PrintWriter out = response.getWriter();
                response.setStatus(401);
                Map<String,Object> map = new HashMap<>();
                map.put("status",401);
                if(e instanceof LockedException){
                    map.put("msg","账户被锁定，登录失败");
                }
                else  if(e instanceof BadCredentialsException){
                    map.put("msg","用户名或密码错误，登录失败");
                }
                else  if(e instanceof DisabledException){
                    map.put("msg","账户被禁用，登录失败");
                }
                else  if(e instanceof AccountExpiredException){
                    map.put("msg","账户已过期，登录失败");
                }
                else  if(e instanceof CredentialsExpiredException){
                    map.put("msg","密码已过期，登录失败");
                }
                else {
                    map.put("msg","未知错误，登录失败，请联系管理人员");
                }
                ObjectMapper om = new ObjectMapper();
                out.write(om.writeValueAsString(map));
                out.flush();
                out.close();
            }
        }).and()
                .logout()
                .logoutUrl("/logout")             //注销接口
                .clearAuthentication(true)        //清除认证信息
                .invalidateHttpSession(true)      //是否使Session失效
                .addLogoutHandler(new LogoutHandler() {        //配置登出处理器
                    @Override
                    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) {

                    }
                }).logoutSuccessHandler(new LogoutSuccessHandler() {
            @Override
            //处理器
            public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
            //重定向，也可以进行其他操作
                httpServletResponse.sendRedirect("login_page");
            }
        }).permitAll()
                .and().csrf().disable()
        ;
    }
}
```

在url栏中使用/logout接口即可退出。

**多个HttpSecurity**

如果业务比较复杂，开发者也可以配置多个HttpSecurity，实现对WebSecurityConfigurerAdapter的多次扩展，代码如下：


```java
@Configuration
public class MultinttpSecurityconfigt{
@Bean
PasswordEncoder passwordEncoder（）{
        return NoOpPasswordEncoder.getInstance（）；
}
@Autowired
protected void configure（AuthenticationManagerBuilder auth）throws Exception{
        auth.inMemoryAuthentication（）
        . withuser("admin").password("123").roles("ADMIN","USER")
        . and()
        . withUser("sang").password("123").roles("USER"); 
}
@Configuration
@order(1)
public static class Adminsecurityconfig extends websecurityconfigurerAdaptert{
@override
protected void configure(Httpsecurity http) throws Exception{
        http. antMatcher("/admin/**").authorizeRequests()
        . anyRequest().hasRole("ADMIN");
     }
}
@Configuration 
public static class OtherSecurityConfig extends WebSecurityconfigurerAdaptert {
@override 
protected void configure (HttpSecurity http) throws Exception {
        http. authorizeRequests()
        . anyRequest(). authenticated()
        . and()
        . formLogin()
        .1oginProcessingUr1("/1ogin")
        . permitA11()
        . and()
        . csrf()
        . disable();
       }
    }
}
```

配置多个HtpSecurity 时，MultilltpsSecurityConfig不需要继承WebsecurityConfigurerAdapter，在MultiHtpSecurity Config中创建静态内部类继承WebsecurityCconfigurerAdapter即可，静态内部类上添加@Configuration 注解和@Order 注解，@Order注解表示该配置的优先级，数字越小优先级越大，未加@Order注解的配置优先级最小。
第14-22行配置表示该类主要用来处理`“amin/**”`模式的URL，其他的URL将在第23-37行配置的HtpSecurity中进行处理。


**安全加密**

使用加密可以保证信息安全，如下可以对密码进行加密

```java
 @Override
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        //哈希加密，10次迭代密匙
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(10);
        //加密密匙
        String passwd = encoder.encode("123");
        System.out.println(passwd);

        auth.inMemoryAuthentication()
                .withUser("admin").password(passwd).roles("ADMIN","USER")
                .and()
                .withUser("sang").password(passwd).roles("USER");
    }
```

注意以上只是说明，可以提前将加密后的文本填写到其中，每次生成的密匙都会不一样，但每次的密匙都能够转换为123

**方法安全**

前面都是通过url来授权，我们也可以通过注解来灵活的配置方法安全，使用@EnableGlobalMethodSecurity注解开启基于方法的安全注解

```java
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class WebSecurityConfig {
}
```

prePostEnabled会解锁@PreAuthorize和@PostAuthorize注解，PreAuthoriz在方法执行前进行验证，PostAuthorize在方法执行后进行验证。

securedEnabled会解锁@Secured注解。

如下为一个服务类添加验证：

```java
@Service
public class MethodService {
    @Secured("ROLE_ADMIN")
    public  String admin(){
        return  "ADMIN";
    }
    @PreAuthorize("hasAnyRole('USER','ADMIN')")
    public String ors(){
        return "USER OR ADMIN";
    }
    @PostAuthorize("hasRole('USER') and  hasRole('ADMIN')")
    public String alls(){
        return "USER AND ADMIN";
    }
}
```

验证权限解释与上面介绍的一致，最后在控制器调用即可：

```java
    @Autowired
    MethodService ms;
    @GetMapping("/or")
    public String or(){
        return ms.ors();
    }
    @GetMapping("/all")
    public String all(){
        return ms.alls();
    }
    @GetMapping("/admin")
    public  String admin(){
        return  ms.admin();
    }
```

登录后相应对应权限即可。


<b id="a2"></b>

### :bowling:基于数据库的验证 ###

:arrow_double_up: [返回目录](#t)

前面介绍的验证都是内存中，在实际中用户的基本信息应该存在数据库中，所以使用数据库验证来获取数据验证比较常用，下面介绍如何使用数据库验证：

**设计数据表**

```sql
create database Authentication;
use Authentication;
create table user(
_id int(11) primary key,
_username varchar(32) not null,
_password varchar(255) not null,
_enabled tinyint(1),
_locked tinyint(1)
);

ALTER TABLE  user MODIFY _username VARCHAR(255) CHARACTER SET utf8 not null;

create table role(
id int(11) primary key,
name varchar(32) not null,
nameZh varchar(32)
);

ALTER TABLE role MODIFY name VARCHAR(32) CHARACTER SET utf8 not null;
ALTER TABLE role MODIFY nameZh VARCHAR(32) CHARACTER SET utf8 not null;

create table user_role(
id int(11) primary key,
uid int(11) not null,
rid int(11) not null
);
```

接下来往数据库中添加信息：

```sql
insert into user values(1,'root','123',1,0);
insert into user values(2,'admin','123',1,0);
insert into user values(3,'sang','123',1,0);

insert into role values(1,'ROLE_dba','数据库管理员');
insert into role values(2,'ROLE_admin','系统管理员');
insert into role values(3,'ROLE_user','用户');


insert into user_role values(1,1,1);
insert into user_role values(2,1,2);
insert into user_role values(3,2,2);
insert into user_role values(4,3,3);
```

注意得是角色名前面有一个ROLE的前缀，这是为了后面使用，建议加上。

添加依赖。这里使用MyBatis：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

    </dependencies>
```

配置数据库：

```java
spring.datasource.url=jdbc:mysql://47.106.254.86/Authentication?characterEncoding=utf8&useSSL=true
spring.datasource.username=lumnca
spring.datasource.password=xxx
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

创建与表一样的实体类：

```java
package run;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class User implements UserDetails {
    private Integer _id;
    private  String _username;
    private String _password;
    private Boolean _enabled;
    private Boolean _locked;
    private List<Role> _roles;
    public  User(){

    }
    public Boolean get_enabled() {
        return _enabled;
    }

    public Boolean get_locked() {
        return _locked;
    }

    public Integer get_id() {
        return _id;
    }

    public List<Role> get_roles() {
        return _roles;
    }

    public String get_password() {
        return _password;
    }

    public String get_username() {
        return _username;
    }

    public void set_enabled(Boolean _enabled) {
        this._enabled = _enabled;
    }

    public void set_id(Integer _id) {
        this._id = _id;
    }

    public void set_locked(Boolean _locked) {
        this._locked = _locked;
    }

    public void set_password(String _password) {
        this._password = _password;
    }

    public void set_roles(List<Role> _roles) {
        this._roles = _roles;
    }

    public void set_username(String _username) {
        this._username = _username;
    }
        
    //获取当前角色所具有的所有角色的信息
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        for (Role role:_roles
        ) {
            authorities.add(new SimpleGrantedAuthority(role.getName()));
        }
        return authorities;
    }
    //获取当前用户的密码
    @Override
    public String getPassword() {
        return _password;
    }
    //获取当前用户的用户名
    @Override
    public String getUsername() {
        return _username;
    }
    //用户信息是否过期
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    //用户是否被锁定
    @Override
    public boolean isAccountNonLocked() {
        return !_locked;
    }
    //密码是否过期
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    //用户是否可用
    @Override
    public boolean isEnabled() {
        return  _enable;
    }

}

//--------------role-----------------------------------
package run;

public class Role {
    private Integer id;
    private  String name;
    private  String namezh;
    public Role(){

    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }

    public String getNamezh() {
        return namezh;
    }

    public void setNamezh(String namezh) {
        this.namezh = namezh;
    }
}
```

这里注意的是属性的字段要与表中字段一样，不然就需要通过注解来一一对应。这里为了简单，所以直接和数据表一样的字段信息。对于User中继承了UserDetails类重写的方法含义如上注释所示。可以根据自己的实际需求来修改上面这7个重写的方法。默认情况下不需要我们自己进行密码对比，因为getPassword方法会自动获取对应的密码与用户输入的密码进行比对，不匹配会抛出BadCredentialsExpire异常，isAccountNonExpired返回值如果为false会抛出AccountNonExpiredExpire异常,所以可以设置这两个返回值来做不同的处理，由于数据库里面没有这两个字段信息，所以返回值都设置为true。可根据需要添加字段也行。

创建MyBatis的UserMapper：

```java
@Mapper
public interface UserMapper {
    @Select("select * from user where _username = #{username}")
    User  loadUserByUsername(String username);
    @Select(" SELECT * FROM role r, user_role ur where r.id = ur.rid and ur.uid = #{id}")
    List<Role> getUserRolesByUid(Integer id);
}
```

创建Dao层：

```java
@Service
public class UserDao implements UserMapper {
    @Autowired
    UserMapper userMapper;
    @Override
    public List<Role> getUserRolesByUid(Integer id) {
        return userMapper.getUserRolesByUid(id);
    }

    @Override
    public User loadUserByUsername(String username) {
        return userMapper.loadUserByUsername(username);
    }
}
```

创建使用层：

```java
@Service
public class UserServer implements UserDetailsService {
    @Autowired
    UserDao userDao;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        User user = userDao.loadUserByUsername(s);
        if(user==null){
            throw  new UsernameNotFoundException("账户不存在");
        }
        user.set_roles(userDao.getUserRolesByUid(user.get_id()));
        return user;
    }
}
```

这里实现了UserDetailsService 接口，并实现了接口中的loadUserByUsername方法，该方法的参数就是用户登录时输入的用户名，通过用户名去数据库中查询数据，不存在就抛出异常，存在对比密码信息再返回数据。

最后添加权限配置信息：

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    UserServer userServer;
    @Bean
    PasswordEncoder passwordEncoder(){
        PasswordEncoder instance = NoOpPasswordEncoder.getInstance();
        return instance;
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.userDetailsService(userServer);
    }
    @Override
    protected void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/admin/**")                            
                .hasRole("admin")                                     
                .antMatchers("/db/**")
                .hasRole("dba")
                .antMatchers("/user/**")
                .hasRole("user")
                .and()
                .formLogin()
                .loginProcessingUrl("/login")                     
                .permitAll()
                .and()
                .csrf()                 
                .disable();
    }
}
```
以上配置在Spring Boot2.7版本以后弃用，需要使用新的配置方式：

```java
@Configuration
public class MySecurityConfig {
    @Autowired
    UserService userServer;
    
    @Bean
    PasswordEncoder passwordEncoder(){
        PasswordEncoder instance = NoOpPasswordEncoder.getInstance();
        return instance;
    }
        //新的身份验证管理器Bean
    @Bean
    AuthenticationManager authenticationManager(HttpSecurity httpSecurity) throws Exception{
        AuthenticationManager authenticationManager = httpSecurity.getSharedObject(AuthenticationManagerBuilder.class)
                .userDetailsService(userServer)
                .and()
                .build();
        return authenticationManager;
    }
        //多Security过滤链
    @Bean
    public SecurityFilterChain securityFilterChain(AuthenticationManager authenticationManager,HttpSecurity httpSecurity) throws Exception{
        httpSecurity.authorizeRequests()
                .requestMatchers("/admin/**")
                .hasRole("admin")
                .requestMatchers("/aqi/**")
                .hasRole("aqi")
                .requestMatchers("/user/**")
                .hasRole("user")
                .and()
                .formLogin()
                .loginProcessingUrl("/login.html")
                .permitAll()
                .and()
                .csrf()
                .disable();
        return httpSecurity.build();
    }
        //放行的路由,可不写上面没有匹配的默认都放行
    @Bean
    WebSecurityCustomizer webSecurityCustomizer(){
        return  new WebSecurityCustomizer() {
            @Override
            public void customize(WebSecurity web) {
                web.ignoring().requestMatchers("/login.html","/static");
            }
        };
    }

}

```

再添加相应的控制器信息即可：

```java
@RestController
public class index {
    @GetMapping("/admin/index")
    public  String admin(){
        return "Hello Admin";
    }
    @GetMapping("/dba/index")
    public  String dba(){
        return "Hello Dba";
    }
    @GetMapping("/user/index")
    public  String user(){
        return "Hello User";
    }
}
```

点击运行测试正确，说明配置成功。

如果想数据库的密码是加密的，可以使用加密后的密码，再改动配置文件即可：

```sql
--123的一个加密密码
update user set _password = '$2a$10$MvbX0ry6FKWeoGpLPnuL7OSeywPFZo5jApIoT1IghcwPyQgvLb4a2' where _id = 3;
```

修改配置类：

```java
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder(10);
    }
```

<b id="a3"></b>

### :bowling:高级配置 ###

:arrow_double_up: [返回目录](#t)

**角色继承**

上面我们定义了3种角色，按道理权限最高的dba也应该具有user和admin的权限，所以需要角色继承，如下：

```java
    @Bean
    RoleHierarchy roleHierarchy(){
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        String hierarchy = "ROLE_dba > ROLE_admin  ROLE_admin > ROLE_user";
        roleHierarchy.setHierarchy(hierarchy);
        return  roleHierarchy;
    }
```

在配置类中添加如上配置，其中` "ROLE_dba > ROLE_admin  ROLE_admin > ROLE_user";`是权限继承规则。不要使用连等。

**动态配置权限**


动态配置权限需要在数据库中添加一张url模式表与一张模式对应权限表，然后实现自定义的FilterlnvocationSecurityMetadataSource，再修改配置即可,下面列出步骤：


添加数据库与对应的Mapper:

```java
public class Menu {
    private Integer id;
    private String pattern;
    private List<Role> roles;

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getPattern() {
        return pattern;
    }

    public void setPattern(String pattern) {
        this.pattern = pattern;
    }

}
```

在前面3张表的基础上添加2张表

表：Menu

|id|pattern|
|:--:|:---:|
|1|Role_admin|
|2|Role_user|

表MenuRole

|id|mid|rid|
|:--:|:---:|:--:|
|1|1|1|
|2|2|2|


建立Mapper：


```java
@Mapper
public interface MenuMapper {
    List<Menu> getAllMenus();
}

```


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="app.auth.MenuMapper">
    <resultMap id="BaseResultMap" type="app.auth.Menu">
        <id property="id" column="id"/>
        <result property="pattern" column="pattern"/>
        <collection property="roles" ofType="app.auth.Role">
            <id property="id" column="rid"/>
            <result property="name" column="rname"/>
            <result property="nameZh" column="rnameZh"/>
        </collection>
    </resultMap>
    <select id="getAllMenus" resultMap="BaseResultMap">
      SELECT m.*,r.id AS rid,r.name AS rname,r.nameZh AS rnameZh FROM menu m LEFT JOIN menu_role mr ON m.id = mr.mid LEFT JOIN role r ON mr.rid=r.id;
    </select>
</mapper>
```

加下来添加配置文件

构造url拦截器 CustomFilterInvocationSecurityMetadataSource


```java
@Component
public class CustomFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {
    AntPathMatcher antPathMatcher=new AntPathMatcher();
    @Autowired
    MenuMapper menuMapper;
    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        //获取url
        String requestUrl = ((FilterInvocation) o).getRequestUrl();
        List<Menu> allMenus = menuMapper.getAllMenus();
        //进行匹配
        for (Menu menu : allMenus) {
            if (antPathMatcher.match(menu.getPattern(), requestUrl)) {
                List<Role> roles = menu.getRoles();
                String[] roleArr = new String[roles.size()];
                for (int i = 0; i < roleArr.length; i++) {
                    roleArr[i] = roles.get(i).getName();
                }
                //对应需要的权限
                return SecurityConfig.createList(roleArr);
            }
        }
        //没有匹配到构造一个没有登录的角色
        return SecurityConfig.createList("ROLE_LOGIN");
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FilterInvocation.class.isAssignableFrom(aClass);
    }
}
```

添加处理：CustomAccessDecisionManager

```java
@Component
public class CustomAccessDecisionManager implements AccessDecisionManager {
    @Override
    public void decide(Authentication auth, Object o, Collection<ConfigAttribute> collection) throws AccessDeniedException, InsufficientAuthenticationException {
        Collection<? extends GrantedAuthority> auths = auth.getAuthorities();
        for (ConfigAttribute configAttribute : collection) {
            //没有登录但是Session存在（通过验证！）
            if ("ROLE_LOGIN".equals(configAttribute.getAttribute()) && auth instanceof UsernamePasswordAuthenticationToken) {
                return;
            }
             //没有登录但是访问的是无拦截路由（通过验证！）
            else if("ROLE_LOGIN".equals(configAttribute.getAttribute()) && !(auth instanceof UsernamePasswordAuthenticationToken)){
                return;
            }
            //验证匹配（通过验证！）
            for (GrantedAuthority authority : auths) {
                if (configAttribute.getAttribute().equals(authority.getAuthority())) {
                    return;
                }
            }
            throw new AccessDeniedException("权限不足");
        }
    }
    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}

```

完成之后修改之前的权限配置文件，添加如下代码：

```java
 @Bean
    RoleHierarchy roleHierarchy(){
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        String hir = "ROLE_dba > ROLE_admin > ROLE_user";
        roleHierarchy.setHierarchy(hir);
        return roleHierarchy;
    }
    @Bean CustomFilterInvocationSecurityMetadataSource cfisms() {
        return new CustomFilterInvocationSecurityMetadataSource();
    }
 @Override
    protected void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O object) {
                        object.setSecurityMetadataSource(cfisms());
                        object.setAccessDecisionManager(cadm());
                        return object;
                    }
                })
                .and()
               //....后面省略

}
```


这样就完成了动态基于数据库访问的shiro权限


<b id="a4"></b>

### :bowling:OAuth2 ###

:arrow_double_up: [返回目录](#t)

OAuth（开放授权）是一个开放标准，允许用户授权第三方移动应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方移动应用或分享他们数据的所有内容，OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0即完全废止了OAuth1.0。

要了解OAuth2，需要先了解OAuth3中几个基本的角色：

* Resource Owner：资源所有者。即用户。
* Client：客户端（第三方应用）。如云冲印。
* HTTP service：HTTP服务提供商，简称服务提供商。如上文提到的github或者Google。
* User Agent：用户代理。本文中就是指浏览器。
* Authorization server：授权（认证）服务器。即服务提供商专门用来处理认证的服务器。
* Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。
* Access Token：访问令牌。使用合法的访问令牌获取受保护的资源。

授权流程：

![](https://image-static.segmentfault.com/317/086/3170867390-5843d768b41e4_articlex)

如上如示，

* 步骤1：`客户端向用户请求授权`

* 步骤2：`同意授权后。返回一个授权许可凭证给客户端`

* 步骤3：`客户端拿着凭证向授权服务器申请令牌`

* 步骤4：`授权服务器确认信息无误后，发送令牌给客户端`

* 步骤5：`客户端拿着令牌去资源服务器访问资源`

* 步骤6：`资源服务器验证令牌无误后开发资源`


**授权模式**

OAuth 2.0 规定了四种获得令牌的流程。你可以选择最适合自己的那一种，向第三方应用颁发令牌。下面就是这四种授权方式。

```
授权码（authorization-code）
隐藏式（implicit）
密码式（password）
客户端凭证（client credentials）
```

`授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。`

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

`第二种方式：隐藏式`

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。

`第三种方式：密码式`

如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。

`第四种方式：凭证式`

最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。

这四种模式有各自的特点，分别适用于不同的开发场景，可以更具实际情况进行选择。

下面介绍在前后端分离中使用的密码模式

**添加依赖**

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>


        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth</groupId>
            <artifactId>spring-security-oauth2</artifactId>
            <version>2.3.3.RELEASE</version>
        </dependency>
    </dependencies>
```


**配置Redis**

```java
spring.redis.database=0
spring.redis.host=47.106.254.86
spring.redis.password=xxx
spring.redis.jedis.pool.min-idle=0
spring.redis.lettuce.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.max-active=8
```

**配置授权服务器**

授权服务器与资源服务器可以是同一台。也可以是不同服务器，这里是使用一样的服务器，通过配置分别开启授权服务器与资源服务器，首先是资源服务器

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    //支持密码模式
    @Autowired
    AuthenticationManager authenticationManager;
    //完成Redis缓存，将令牌信息保存到Redis
    @Autowired
    RedisConnectionFactory redisConnectionFactory;
    //刷新令牌支持
    @Autowired
    UserDetailsService userDetailsService;
    //启用加密密码
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder(10);
    }
    @Override
    public void configure(ClientDetailsServiceConfigurer clients)throws Exception{
        clients.inMemory()
                .withClient("password")
                .authorizedGrantTypes("password","refresh_token")
                .accessTokenValiditySeconds(1800)
                .resourceIds("rid")
                .scopes("all")
                .secret("$2a$10$MvbX0ry6FKWeoGpLPnuL7OSeywPFZo5jApIoT1IghcwPyQgvLb4a2");
    }
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception{
        endpoints.tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);

    }
    @Override
    public void  configure(AuthorizationServerSecurityConfigurer security) throws Exception{
        security.allowFormAuthenticationForClients();
    }
}
```

上面的配置第一个配置方法是配置password授权模式，authorizedGrantTypes表示Oauth2中的授权模式为password和refresh_token两种，在标准的Oauth2中并不包含刷新，就需要添加accessTokenValiditySeconds模式，该方法配置过期时间为1800秒，resourceIds配置了资源id，scopes方法配置了加密后的密码，明文是123，后一个配置方法配置了缓存到Redis，最后一个配置方法是支持client_id和client_secret做登录验证。

**配置资源服务器**

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public  void configure(ResourceServerSecurityConfigurer resource)throws Exception{
        resource.resourceId("rid").stateless(true);
    }
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").hasRole("user")
                .anyRequest().authenticated();
    }
}
```

第一个配置方法配置资源id，这里的id和授权服务器中资源id一样，然后设置这些资源令牌认证。

**配置Security**

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    @Override
    public AuthenticationManager authenticationManager()throws Exception{
        return super.authenticationManager();
    }
    @Bean
    @Override
    protected UserDetailsService userDetailsService(){
        return super.userDetailsService();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
                .withUser("admin")
                .password("$2a$10$MvbX0ry6FKWeoGpLPnuL7OSeywPFZo5jApIoT1IghcwPyQgvLb4a2")
                .roles("admin")
                .and()
                .withUser("sang")
                .password("$2a$10$MvbX0ry6FKWeoGpLPnuL7OSeywPFZo5jApIoT1IghcwPyQgvLb4a2")
                .roles("user");
    }
    @Override
    protected void configure(HttpSecurity http)throws Exception{
        http.antMatcher("/oauth/**").authorizeRequests()
                .antMatchers("/oauth/**").permitAll()
                .and().csrf().disable();
    }
}
```

与前面不一样的是，这里多了两个Bean，两个Bean作为将注入授权服务器配置类使用，另外这里oauth模式的url请求被直接放行不需要验证。前面的资源服务器中也添加了配置，但是Security中的配置优先级更高，所以先执行这里的配置。

最后添加验证控制器：

```java
@RestController
public class index {
    @GetMapping("/admin/index")
    public  String admin(){
        return "Hello Admin";
    }
    @GetMapping("/index")
    public  String index(){
        return "Hello";
    }
    @GetMapping("/user/index")
    public  String user(){
        return "Hello User";
    }
}
```


接下来启动Redis，第一步是要获取到令牌，输入如下url（方法为POST）：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a29.png)

转换为具体的url为`http://localhost:8080/oauth/token?username=sang&password=123&grant_type=password&client_id=password&scope=all&client_secret=123`

返回的JSON中access_token就是令牌信息。refresh_token是用于刷新的令牌。expires_in为过期时间单位为秒。如果想刷新令牌，输入如下url（POST）

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a30.png)

url为`http://localhost:8080/oauth/token?grant_type=refresh_token&refresh_token=915495bc-2687-469e-b211-9251ec21b00e&client_id=password&client_secret=123`

在返回的信息中可以看到令牌变化了。

最后在url中加入令牌即可访问`http://localhost:8080/user/index?access_token=bbf61f63-2af8-4a79-b63b-85c2fde0eadf`如果权限不对，返回错误JSON

<b id="a5"></b>

### :bowling:Spring Boot整合Shiro ###

:arrow_double_up: [返回目录](#t)

Apache是一个开源的轻量级的java安全框架，它提供验证，授权，密码管理以及会话管理等功能，相比Security更直观，更简单。下面介绍shiro使用：

添加依赖

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-web-starter</artifactId>
            <version>1.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>
    </dependencies>
```

基本配置：

```java
shiro.enabled=true                  --开启Shiro配置
shiro.web.enabled=true              --开启Shiro Web配置
shiro.loginUrl=/login               --登录的默认地址/login
shiro.successUrl=/index             --登录成功后跳转的界面
shiro.unauthorizedUrl=/unauthorized             --未授权的跳转界面
shiro.sessionManager.sessionIdUrlRewritingEnabled=true         --是否允许通过URL参数实现会话
shiro.sessionManager.sessionIdCookieEnabled=true               --是否允许通过Cookies实现会话跟踪
```


配置Shiro：

```java
@Configuration
public class ShiroConfig {
    //定义用户信息
    @Bean
    public Realm realm(){
        TextConfigurationRealm realm = new TextConfigurationRealm();
        realm.setUserDefinitions("sang=123,user\n admin=123,admin");  //\n为下一行用户信息
        realm.setRoleDefinitions("admin=read,write\n user=read");  //读写权限配置
        return  realm;
    }
    //定义过滤规则anon可以匿名访问
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition(){
        DefaultShiroFilterChainDefinition chainDefinition =
                new DefaultShiroFilterChainDefinition();
        chainDefinition.addPathDefinition("/login","anon");
        chainDefinition.addPathDefinition("/doLogin","anon");           
        chainDefinition.addPathDefinition("/logout","logout");          //注销请求
        chainDefinition.addPathDefinition("/**","authc");   //需要认证
        return  chainDefinition;
    }
    //支持在模板页中使用shiro标签
    @Bean
    public ShiroDialect shiroDialect(){
        return  new ShiroDialect();
    }
}

```


异常处理：

```java
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(ArithmeticException.class)
    public ModelAndView error(AuthorizationException e){
        ModelAndView mv = new ModelAndView("unauthorized");
        mv.addObject("error",e.getMessage());
        return  mv;
    }
}
```

最后配置模板页进行测试，在resource文件夹下创建Templates文件夹，并创建如下测试文件：

**index.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
    <h3>Hello <shiro:principal/></h3>  <!--显示用户名-->
    <h3><a href="/logout">注销登录</a></h3>
    <h3><a shiro:hasRole="admin" href="/admin">管理员界面</a></h3>
    <h3><a shiro:hasAnyRoles="admin,user" href="/user">用户界面</a></h3>
</body>
</html>
```

**login.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>login</title>
</head>
<body>
    <form action="/doLogin" method="post">
        <label>用户名</label>
        <input type="text" name="username"><br>
        <label>密码</label>
        <input type="password" name="password">
        <div th:text="${error}"></div>
        <input type="submit" value="登录">
    </form>
</body>
</html>
```

**unauthorized.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Unauthorized</title>
</head>
<body>
    <h3>未获取授权，非法访问</h3>
    <h3 th:text="${error}"></h3>
</body>
</html>
```

**user.html**


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>User</title>
</head>
<body>
<h3>用户界面</h3>
</body>
</html>
```

**admin.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Admin</title>
</head>
<body>
<h3>管理员界面</h3>
</body>
</html>
```

运行测试成功说明配置正确。
