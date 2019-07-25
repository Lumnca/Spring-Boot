# :maple_leaf:Spring Boot整合持久层技术 #

<b id="t"></b>

:arrow_down:[整合JdbcTemplate](#a1)

:arrow_down:[整合MyBatis](#a2)

<b id="a1"></b>

### :fallen_leaf:整合JdbcTemplate ###

:arrow_double_up:[返回目录](#t)

整合数据库有很多方式，在这里我们只介绍JdbeTemplate和MyBatis。其余的可以自行上网查找资料。

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

MyBatis是一款优秀的持久层框架，原名叫作iBaits，2010年由ApacheSoftwareFoundation迁移到Google Code并改名为MyBatis，2013年又迁移到GitHub上。MyBatis 支持定制化SQL、存储过程以及高级映射。MyBatis几乎避免了所有的JDBC代码手动设置参数以及获取结果集。在传统的SSM框架整合中，使用MyBatis需要大量的XML配置，而在SpringBot中，MyBatis官方提供了一套自动化配置方案，可以做到MyBatis开箱即用。具体使用步骤如下。

添加依赖：

```xml
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
```

数据表与实体类和前面一样这里不做简述，下一步创建数据库访问层：

在某个包下创建UserMapper接口，其方法这里只加入三个

```java
package mapper;

import org.apache.ibatis.annotations.Mapper;
import run.User;

import java.util.List;

@Mapper
public interface UserMapper {
    int add(User user);
    User getUser(String name);
    List<User> getAll();
}
```

这里在接口处声明@Mapper说明这是一个Mapper，该注解表明这个接口是MyBatis中的Mapper，这种方式需要在每个接口上添加注解，也可以在运行类上添加注解：

```java
@SpringBootApplication   
@MapperScan("mapper")  //扫描mapper包下所有的接口作为Mapper
public class start {
    public static  void main(String[] args){
        SpringApplication.run(start.class,args);
    }
}

```

接下来就是创建与之对应的UserMapper.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
         PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.UserMapper">     ------  Mapper类所在的包
  
<insert id="add" parameterType="run.User">    -------接口方法add实现配置  参数实体类对应run包的User类
    INSERT INTO tab(name,tell,sex) VALUES(#{ name },#{ tell },#{ sex })   -----#name 对应实体类的相应的属性
</insert>

<select id="getUser" parameterType="String" resultType="run.User">   ------返回类型为run包下的User类
    SELECT * FROM tab WHERE name = #{ name}
 </select>

<select id="getAll" resultType="run.User">
    SELECT * FROM tab
</select>
</mapper>
```

上面就是针对该接口的方法实现xml配置文件，`#{}`可以代替接口中的参数，实体类中的属性可以直接通过`#{实体类属性名}`获取

接下来实现接口服务类：

```java
@Service
public class ListDao implements UserMapper{

    @Autowired
    UserMapper db;
    @Override
    public int add(User user) {
        return db.add(user);
    }

    @Override
    public User getUser(String name) {
        return db.getUser(name);
    }

    @Override
    public List<User> getAll() {
        return db.getAll();
    }
}
```

最后在控制器中访问：

```java
@RestController
public class index {

    @Autowired
    ListDao db;
    @GetMapping("/add")
    public int add(){
        return  db.add(new User("Hey","s","152"));
    }
    @GetMapping("/getUser")
    public User getUser(){
        return db.getUser("lumnca");
    }
    @GetMapping("/getAll")
    public List<User> getAll(){
        return db.getAll();
    }
}
```

最后配置xml文件位置，由于xml文件都是在静态文件夹resource中扫描的，所以配置pom.xml位置到该文件中完成扫描：

```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
```

像这样访问控制器路径，访问成功完成配置。我的文件夹如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a14.png)

当然像这样使用xml比较麻烦，可以直接使用注解完成整合配置。删除前面所有有关xml的配置，在接口中作如下修改：

```java
@Mapper
public interface UserMapper {
    @Insert("INSERT INTO tab(name,tell,sex) VALUES(#{ name },#{ tell },#{ sex })")
    int add(User user);

    @Select("SELECT * FROM tab WHERE name = #{ name}")
    User getUser(String name);

    @Select("SELECT * FROM tab")
    List<User> getAll();
}
```

将接口要对应的sql语句作为注解添加到方法前面，就完成了整合。由于我们的实体类属性与数据库列属性一致，所以我们这里并没有配置参数，如果不一致，比如我们的实体类name改为了namex那么就需要做参数映射处理：

```java
@Mapper
public interface UserMapper {
    @Insert("INSERT INTO tab(name,tell,sex) VALUES(#{ namex },#{ tell },#{ sex })")
    @Options(useGeneratedKeys = true,keyProperty = "namex",keyColumn = "name")   //keyProperty实体类属性名 ，keyColumn数据库属性名
    int add(User user);

    @Select("SELECT * FROM tab WHERE name = #{ namex}")
    @Results({
            @Result(column = "name",property = "namex")            //返回结果实体类，column数据库列段名，property实体类名
    })
    User getUser(String name);

    @Select("SELECT * FROM tab")
    @Results({
            @Result(column = "name",property = "namex")
    })
    List<User> getAll();
}
```
