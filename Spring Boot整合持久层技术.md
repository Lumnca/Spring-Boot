# :maple_leaf:Spring Boot整合持久层技术 #

<b id="t"></b>

:arrow_down:[整合JdbcTemplate](#a1)

:arrow_down:[整合MyBatis](#a2)

:arrow_down:[整合JPA](#a4)

:arrow_down:[多数据源](#a3)

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

<b id="a4"></b>

### :fallen_leaf:整合JPA ###

:arrow_double_up:[返回目录](#t)

JPA（Java Persistence API）和Spring Data是两个范畴的概念。

作为一名JavaEE工程师，基本都有听说过Hibermate框架。Hibernate是一个ORM框架，而PA则是一种ORM规范，JPA和Hibermate的关系就像JDBC与JDBC驱动的关系，即JPA制定了ORM规范，而Hibernate是这些规范的实现（事实上，是先有Hibermate后有JPA，JPA规范的起草者也是Hibermate的作者），因此从功能上来说，JPA相当于Hibermate的一个子集。

Sping Data 是Spring的一个子项目，致力于简化数据库访问，通过规范的方法名称来分析开发者的意图，进而减少数据库访问层的代码量。Spring Data不仅支持关系型数据库，也支持非关系型数据库。Spring DataJPA可以有效简化关系型数据库访问代码。
Spring Boot 整合Spring Data JPA的步骤如下。

添加依赖

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
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
    </dependencies>
```

数据库配置：

```
spring.datasource.url=jdbc:mysql://47.106.254.86/ex?characterEncoding=utf8&useSSL=true
spring.datasource.username=lumnca
spring.datasource.password=chuan868
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.database=mysql
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
spring.jpa.show-sql=true
```

常里的配置信息主要分为两大类，

* 第1-5行是数据库基本信息配置，第6~9行是JPA相关配置中：。
* 第5行本示路否在控制合打印入执行过程生成的SQL.
* 第6行表示JPA对应的数据库是MySQL，
* 第7行表示年项目启动时根据实体关更新数据库中的表。
* 第8行则表示使用的数据库方言是MySOLS7Diakt.

接下来创建实体类：

```java
@Entity(name = "tab")
public class User implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "name")
    private String name;
    @Column(name = "tell")
    private String tell;
    @Column(name = "sex")
    private String sex;

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

    public void setNamex(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

代码解释：

`@Eintiy注解表示该类是一个实体类，在项目启动时会根据该类自动生成一张表，表的名称即@Entity注解中name的值，如果不配置name，默认表名为类名。`

`所有的实体类都要有主键，@Id注解表示该属性是一个主键，@GeneratedValue 注解表示主键自动生成，strategy则表示主键的生成策略。`

`默认情况下，生成的表中字段的名称就是实体类中属性的名称，通过@Column注解可以定制生成的字段的属性，name表示该属性对应的数据表中字段的名称，nulable表示该字段非空。`

`@Transient注解表示在生成数据库中的表时，该属性被忽略，即不生成对应的字段。`

创建UserDao接口：

```java
public interface UserDao extends JpaRepository<User,Integer> {

}
```

内置方法可以自行添加，参考网上说明也行，由于后面介绍RESTful，这里加单说下。

创建服务层：

```java
@Service
public class UserServer {
    @Autowired
    UserDao userDao;
    public void addUser(User user){
        userDao.save(user);
    }
    public Page<User> getUserByPage(Pageable pageable){
        return userDao.findAll(pageable);
    }
}
```

这里也是简单列举,最后控制器访问：

```java
@RestController
public class index {
    @Autowired
    UserServer us;
    @GetMapping("/add")
    public  int add(){
        us.addUser(new User("Really","man","15155126"));
        return 1;
    }
    @GetMapping("get")
    public  void get(){
        PageRequest pageRequest = PageRequest.of(1,2);
        Page<User> page = us.getUserByPage(pageRequest);
        System.out.println("总页数:"+page.getTotalPages());
        System.out.println("总记录数:"+page.getTotalElements());
        System.out.println("查询结果:"+page.getContent());
        System.out.println("当前页数:"+page.getNumber()+1);
    }
}
```

运行对应的路由即可。

<b id="a3"></b>

### :fallen_leaf:多数据源 ###

:arrow_double_up:[返回目录](#t)

所谓多数据源就是项目采用了不同的数据库源。一般来说说，采用MyCat等分布式数据库中间件是比较好的解决方案，这样就可以把数据库分离，分库分表，备份等操作交给中间件去做，Java代码只需注重业务即可。

JdbcTemplate多数据源配置，因为一个JdbcTemplate对应一个DateSource，开发者只需要手动提供多个DataSource，再手动配置JdbcTemplate即可。具体步骤如下：


首先配置两个同的数据库表：

```sql

---库1
create database stu;
use Stu;
 Create table Studnet(
	StuID int primary key,
    StuName varchar(50) not null,
    StuPwd varchar(50) not null
);

ALTER TABLE  Studnet MODIFY StuName VARCHAR(50) CHARACTER SET utf8 not null;

insert into Studnet values('20170101','刘大头','123456');


----库2

create database stu2;
use Stu2;
 
Create table Studnet(
	StuID int primary key,
    StuName varchar(50) not null,
    StuPwd varchar(50) not null
);


ALTER TABLE  Studnet MODIFY StuName VARCHAR(50) CHARACTER SET utf8 not null;

insert into Studnet values('20170102','周大头','123456');
```

添加依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.9</version>
        </dependency>
```


然后添加原有依赖后，配置数据连接池：

```java
#数据库源1
spring.datasource.one.url=jdbc:mysql://127.0.0.1/stu?characterEncoding=utf8&useSSL=true
spring.datasource.one.username=root
spring.datasource.one.password=xxx
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.one.driver-class-name=com.mysql.jdbc.Driver

#数据库源2
spring.datasource.two.url=jdbc:mysql://127.0.0.1/stu2?characterEncoding=utf8&useSSL=true
spring.datasource.two.username=root
spring.datasource.two.password=xxx
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.two.driver-class-name=com.mysql.jdbc.Driver
```

接下来就是配置数据源，新建一个DataSourceConfig类，根据数据池配置生成两个数据源：

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.one")
    DataSource dsOne(){
        return DruidDataSourceBuilder.create().build();
    }
    @Bean
    @ConfigurationProperties("spring.datasource.two")
    DataSource dsTwo(){
        return DruidDataSourceBuilder.create().build();
    }
}
```

其中DataSourceConfig提供了两个数据源：dsOne与dsTwo，默认方法名即实例名。 @ConfigurationProperties注解表明使用不同前缀文件来创建不同的DataSource实例。

下一步是配置JdbcTemplate:

新建一个JdbcTemplateConfig配置类：

```java
@Configuration
public class JdbcTemplateConfig {
    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne")DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo")DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```

由于引用了多个数据源，所需这个配置类需要手动配置，不能像前面那样直接使用，这样就生成了两个数据源，JdbcTemplateConfig 中提供了两个方法，每个方法都生成了一个JdbcTemplate实例，由于有两个实例，需要通过方法名来查找，@Qualifier注解表示查找不同名称的DataSource实例注入进来。

最后一步使用，这里就不配置服务层，直接使用：

```java
@RestController
public class index {
    @Resource(name = "jdbcTemplateOne")
    JdbcTemplate jdbcTemplateOne;
    @Autowired
    @Qualifier("jdbcTemplateTwo")
    JdbcTemplate jdbcTemplateTwo;
    @GetMapping("/getAll")
    public List<Stu> getAll(){
            List<Stu> stu1 = jdbcTemplateOne.query("Select * from  Studnet",
                    new BeanPropertyRowMapper<>(Stu.class));

            List<Stu> stu2 = jdbcTemplateTwo.query("Select * from  Studnet",
                    new BeanPropertyRowMapper<>(Stu.class));

            stu1.addAll(stu2);

            return  stu1;
    }
}
```

简单起见，这里没有添加Service层，而是直接将JdbcTemplate注入到了Controller中。在Controller中注入两个不同的JdbcTemplate有两种方式：一种是使用@Resource注解，并指明name属性，即按name进行装配，此时会根据实例名查找相应的实例注入；另一种是使用@Autowired注解结合@Qualifier 注解，效果等同于使用@Resource注解。


**使用MyBatis多数据源**

MyBatis多数据源配置比上面的复杂一点，这里也简单介绍一下，添加MyBatis依赖和数据池配置类不需要改变，第一步创建MyBatis配置：

```java
@Configuration
@MapperScan(value = "run.mapper2",sqlSessionFactoryRef = "sqlSessionFactoryBean2")
public class MyBatisConfigTwo {
    @Autowired
    @Qualifier("dsTwo")
    DataSource dsTwo;
    @Bean
    SqlSessionFactory sqlSessionFactoryBean2()throws Exception{
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dsTwo);
        return  factoryBean.getObject();
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate2()throws  Exception{
        return new SqlSessionTemplate(sqlSessionFactoryBean2());
    }
}
---------------------------------------数据源2配置--------------------------------------------
@Configuration
@MapperScan(value = "run.mapper1",sqlSessionFactoryRef = "sqlSessionFactoryBean1")
public class MyBatisConfigOne {
    @Autowired
    @Qualifier("dsOne")
    DataSource dsOne;
    @Bean
    SqlSessionFactory sqlSessionFactoryBean1()throws Exception{
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dsOne);
        return factoryBean.getObject();
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate()throws Exception{
        return new SqlSessionTemplate(sqlSessionFactoryBean1());
    }
}

```

* 在@MapperScan注解中指定Mapper接口所在的位置，同时指定SqlSessionFactory的实例名，则该位置下的Mapper 将使用SqlSessionFactory实例。

* 提供SqlSessionFactory的实例，直接创建出来，同时将DataSource的实例设置给SqlSessionFactory，这里创建的SqlSesionFactory 实例也就是@MapperScan

* 注解中sqlSessionFactoryRef参数指定的实例。提供一个SqlSessionTemplate 实例。这是一个线程安全类，主要用来管理MyBatis 中的SqlSession 操作。

当配置完成后创建Mapper和与之对应的xml文件：

```java
@Service
@Mapper
public interface Mapper1 {
    List<Stu> getAll();
}

xml文件
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="run.mapper1.Mapper1">
    <select id="getAll" resultType="run.Stu">
    SELECT * FROM Studnet
</select>
</mapper>
```

Mapper2配置一样，这里不列出，最后在控制器中引用：

```java
@RestController
public class index {
    @Autowired
    Mapper1 mapper1;
    @Autowired
    Mapper2 mapper2;
    @GetMapping("/get")
    public List<Stu> get(){
        List<Stu> s = mapper1.getAll();
        s.addAll(mapper2.getAll());
        return s;
    }
}
```
运行看到结果这说明配置成功
