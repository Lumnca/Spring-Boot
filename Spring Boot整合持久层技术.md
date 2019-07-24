# :maple_leaf:pring Boot整合Web #

<b id="t"></b>

:arrow_down:[整合JdbcTemplate](#a1)

:arrow_down:[整合MyBatis](#a2)

<b id="a1"></b>

### :fallen_leaf:整合JdbcTemplate ###

:arrow_double_up:[返回目录](#t)

JdbeTemplate是Spring提供的一套JDBC模板框架，利用AOP技术来解决直接使用JDBC时大量重复代码的问题。Jdbc Template虽然没有MyBatis那么灵活，但是比直接使用JDBC要方便很多。Spring Boot中对Jdbc Template的使用提供了自动化配置类Jdbc TemplateAutoConfiguration。

在 TemplateAutoConfiguration类中，说明了当classpath下存在DataSource和JdbcTemplate并且DataSource只有一个实例时，自动化配置才会生效，若开发者没有提供JdbcOperations，则Spring Boot会自动向容器中注入一个JdbcTemplate 。由此可以看出，开发者想使用jdbcTemplate，只需要提供JdbcTemplate和DataSource的依赖即可。具体操作如下：

添加依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--JDBC依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!--mysql连接依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--阿里巴巴数据池依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
```

这里需要区分一点的就是使用的mysql版本要与连接mysql的版本接近，否者会出现错误，我这里使用的是mysql5.7.


数据库配置：

```xml
spring.datasource.url=jdbc:mysql://127.0.0.1/sgz?characterEncoding=utf8&useSSL=true
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

如果你的数据库是mysql8.0以下的使用驱动名应为：`com.mysql.jdbc.Driver`如果是8.0以上的应该是`com.mysql.cj.jdbc.Driver`

配置一个数据库访问类：

```java
@Repository
public class ListDao {
    @Autowired
    JdbcTemplate jdbc;
    public int add(){
        return jdbc.update("insert into wj values ('s','s',1,1,1,1)");
    }
}
```

控制器访问：

```java
@RestController   
public class index {

    @Autowired
    ListDao db;
    @GetMapping("/index")  
    public int index(){
        return  db.add();
    }
}
```

测试：在地址栏输入这个路径即可，若返回1表示设置添加数据库数据成功！

可以看到我们不用在像以前那样写很多的sql执行方法连接数据库等代码，极大的节省了代码编写量，这种整合提高了编码效率，当然上面只是简单演示，下面介绍一些JdbcTemplate的使用方法。

上面的例子可以看出，首先需要实例化一个 JdbcTemplate类，然后调用了update()方法，该方法是用于执行插入，修改，删除这一类没有返回结果集的方法，执行成功返回1，否者为0.所以我们可以使用这一类的方法来执行这类语句,如：

```java
    public int InsertSql(String sql){
        return jdbc.update(sql);
    }
```

对于有结果集的可以使用query，queryForObject方法来获取，不一样的是这种方法需要数据填充类来绑定数据，我使用如下实例：

数据库数据如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a13.png)

创建与数据库数据匹配的类（字段属性值要一致）：

```java
public class User {
    private String name;
    private String sex;
    private String tell;
    
    public  User(){
        
    }
    
    public  User(String n,String s,String t){
        name = n;
        sex = s;
        tell = t;
    }

    public String getSex() {
        return sex;
    }

    public void setTell(String tell) {
        this.tell = tell;
    }

    public String getTell() {
        return tell;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

使用JdbcTemplate整合：

```java
@Repository
public class ListDao {
    @Autowired
    JdbcTemplate jdbc;
    public int add(User user){
        return jdbc.update("insert into tab values(?,?,?)",user.getName(),user.getTell(),user.getSex());
    }
    public int update(User user){
        return  jdbc.update("update tab set tell = ? where name = ?",user.getTell(),user.getName());
    }
    public int delete(String name)
    {
        return jdbc.update("delete from  tab where name = ?",name);
    }
    public User getUser(String name){
        return jdbc.queryForObject("select * from tab where name = ?",
                new BeanPropertyRowMapper<>(User.class),name);
    }
    public List<User> getAll(){
        return jdbc.query("select * from tab", new BeanPropertyRowMapper<>(User.class));
    }
}
```

接下来就是控制器引用：

```java
public class index {

    @Autowired
    ListDao db;
    @GetMapping("/add")
    public int add(){
        return  db.add(new User("Hey","s","152"));
    }
    @GetMapping("/update")
    public  int update(){
        return  db.update(new User("lumnca","m","999"));
    }
    @GetMapping("/delete")
    public  int delete(){
        return  db.delete("kally");
    }
    @GetMapping("/getUser")
    public User getUser(){
        return db.getUser("lumnca");
    }
    @GetMapping("/getAll")
    public List<User> getAll(){
        return db.getAll();
    }
    @GetMapping("/index")
    public  User index(){
        return new User("aaa","s","488");
    }
}
```

像这样就可以实现全部的功能了，注意的是这个数据类，首先是必须含有无参的构造函数，其次是必须属性值一致。否者就使用自己实现RowMapper<>接口。将类的属性与数据库属性对应起来。

<b id="a1"></b>

### :fallen_leaf:整合MyBatis ###

:arrow_double_up:[返回目录](#t)






