# :golf:Spring Boot 安全管理  #

<b id="t"></b>

:arrow_double_down:[Spring Security的基本配置](#a1)

:arrow_double_down:[基于数据库的验证](#a2)




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

