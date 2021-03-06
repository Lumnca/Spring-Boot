# :golf:构建RESTful服务 #

<b id="t"></b>

:arrow_double_down:[REST简介](#a1)

:arrow_double_down:[JPA实现REST](#a2)

<b id="a1"></b>

### :bowling:REST简介 ###

:arrow_double_up: [返回目录](#t)

REST（Representational State Transfer）是一种Web软件架构风格，它是一种风格，而不是标准，匹配或兼容这种架构风格的网络服务称为REST服务。REST服务简洁并且有层次，REST通常基于HTTP、URI和XML以及HTML这些现有的广泛流行的协议和标准。

在REST中，资源是由URI来指定的，对资源的增删改查操作可以通过HTTP协议提供的GET、POST、PUT、DFLETE等方法实现。使用REST可以更高效地利用缓存来提高响应速度，同时REST中的通信会话状态由客户端来维护，这可以让不同的服务器处理一系列请求中的不同请求，进而提高服务器的扩展性在前后端分离项目中，一个设计良好的Web软件架构必然要满足REST风格。

在SpTing MVC框架中，开发者可以通过@RestControler注解开发一个RESTfl服务，不过Spring Bot 对此提供了自动化配置方案，开发者只需要添加相关依赖就能快速构建一个RESTful服务。

<b id="a1"></b>

### :bowling:JPA实现REST ###

:arrow_double_up: [返回目录](#t)

在Spring Boot中使用JPA和Spring Data Rest可以快速开发一个RESTful应用，下面介绍Spring Boot中非常方便的RESTFul应用开发。

**:one:基本实现**

创建项目添加依赖：

```xml
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
```

项目配置：

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


数据库脚本：

```sql
create table tab1(
	id int auto_increment primary key,
    name varchar(50) not null,
    age int
);

ALTER TABLE tab1 MODIFY  name VARCHAR(251) CHARACTER SET utf8 not null;

insert into tab1 values(1,'张三',25);
```

实体类创建：

```
@Entity(name = "tab1")
public class User implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    @Column(name = "name")
    private String name;
    private int age;

    public  User(){

    }
    public  User(int _id,String _name,int _age){
        name = _name;
        id = _id;
        age =_age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }

    public void setNamex(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

```

实体配置类需要注意的是，字段的属性，主键由于是int自增型，因此可以使用GenerationType.IDENTITY自增序列。或者使用GenerationType.AUTO，如果为字符串则不能这样使用注解，直接省略这个注解就行。 @Column注解可以对应数据库的字段名。

创建接口：

```java
public interface UserReqository extends JpaRepository<User,String> {
}
```

这里主要第二个泛型参数，这个是你的主属性的类型值，由于上面的是类型，所以这里也一样。这样不需要你做过多的配置就可以完成了。接下来需要输入url访问就行。

默认是以你的实体类名+s访问全局数据：

`http://localhost:8080/users`

由于只能使用GET方法所以只能够访问。加上主属性可以看到单个实例：`http://localhost:8080/users/1`

对于数据过多的可以使用page与size参数来查询

其中page是分页，默认是20个数据一页可以自行修改：

`http://localhost:8080/users?page=4?size=10`意为每页10个数据，显示第4页的内容。


为了能够做出示例，这里我们使用POSTman工具来演示效果：


**添加数据**

想数据库中添加一个数据可以使用POST请求，然后在请求中添加实体类的JOSN字符串，如下所示：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a18.png)

如上只要返回了对于的数据JSON，则说明请求成功！




**查询数据**

查询数据直接在实体类名后加上主键属性即可，如`http://localhost:8080/users/2` 是查询主键id=2的记录，返回如下：

```json
{
    "name": "李四",
    "age": 25,
    "_links": {
        "self": {
            "href": "http://localhost:8080/users/2"
        },
        "user": {
            "href": "http://localhost:8080/users/2"
        }
    }
}
```


**修改数据**

修改数据需要将url定向到所要修改的数据，即使用查询的路径方式，下一步就是在主体中提交JSON信息如下修改id=2的信息：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a19.png)

**删除数据**

删除数据使用del方法即可，只需要定位id即可，如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a20.png)

这里就实现了基本的功能，当然也可以使用前端ajax请求来完成这些操作，这样就是开发更加简单与方便。上面说到不是int型主键时不能够使用这种方式，当为字符串使用如下方式访问：

```java
    @Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid",strategy = "uuid")
    @Column(name = "id",unique = true,nullable = false)
    private String id;
    @Column(name = "name")
    private String name;
    private int age;
```

**:two:自定义请求路径**

默认情况下请求路径都是实体类名+s，如果想修改这个设置的话，通过注解即可，在接口类声明处添加注解，如下注解：

```java
@RepositoryRestResource(path = "abc",collectionResourceRel = "us",itemResourceRel = "ues")
public interface UserReqository extends JpaRepository<User,Integer>, JpaSpecificationExecutor{
}
```

其中path是url路径修改，collectionResourceRel是返回JSON数据集合中的key值修改，itemResourceRel时JSON中单个实例的修改。效果如下图所示：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a21.png)

**:three:自定义查询方法**

默认的查询方法支持分页查询，排序查询以及按照id查询，如果开发者想要某个属性查询，主要在接口中定义相关方法暴露即可，如下：

```java
@RepositoryRestResource(path = "abc",collectionResourceRel = "us",itemResourceRel = "ues")
public interface UserReqository extends JpaRepository<User,Integer>, JpaSpecificationExecutor {
    @RestResource(path = "names",rel = "name")
    List<User> findAllByNameContains(@Param("names")String names);
    @RestResource(path = "name",rel = "name")
    User findByNameEquals(@Param("name")String name);
}
```

其中本身在接口中添加方法就会使得多一条访问路径第一个方法就会映射为：`http://localhost:8080/abc/search/findAllByNameContains?name=张三`使用了 @RestResource注解可以改变方法名，其中path是改变路径名称，rel是对应后接参数，所以终方式也就成为了：`http://localhost:8080/abc/search/names?name=张三`也可以使用`http://localhost:8080/abc/search`来查看有哪几种对应方式。

**:four:隐藏方法**

默认情况下。凡是继承了Repository接口的类都会被暴露出来，即开发者可以执行执行基本增删改查的方法，就像前面一样，能够随意的使用这些方法。如果不想暴露相关操作，做以下配置即可：

```java
@RepositoryRestResource(path = "abc",collectionResourceRel = "us",itemResourceRel = "ues",exported = false)
public interface UserReqository extends JpaRepository<User,Integer>, JpaSpecificationExecutor {
    @RestResource(path = "names",rel = "name")
    List<User> findAllByNameContains(@Param("names")String names);
    @RestResource(path = "name",rel = "name")
    User findByNameEquals(@Param("name")String name);

}
```

添加了exported = false后，原有的接口方法全部失效，包括内部方法。如果只想某一个不被使用，那么可以在方法上配置，比如对删除进行限制：

```java
@RepositoryRestResource(path = "abc",collectionResourceRel = "us",itemResourceRel = "ues")
public interface UserReqository extends JpaRepository<User,Integer>, JpaSpecificationExecutor {
    @RestResource(path = "names",rel = "name")
    List<User> findAllByNameContains(@Param("names")String names);
    @RestResource(path = "name",rel = "name")
    User findByNameEquals(@Param("name")String name);

    @RestResource(exported = false)
    void deleteById(Integer id);
}


```

**:five:其他CORS**

配置CORS只需要在接口前面加上@CrossOrigin注解即可，这样接口中所有方法都支持跨域，如果只想某一个跨域，可以添加到某个方法上面去。

开发者还可以在application中添加一些配置：

```
#每页默认记录数，默认值为20
spring.data.rest.default-page-size=2
#分页查询页码参数名，默认值为page
spring.data.rest.page-param-name=page
#分页查询记录数参数名，默认值为size
spring.data.rest.1imit-param-name=size
#分页查询排序参数名，默认值为sort
spring.data.rest.sort-param-name=sort
#base-path 表示给所有请求路径都加上前缀
spring.data.rest.base-path=/api
#添加成功时是否返回添加内容
spring.data.rest.return-body-on-create=true
#更新成功时是否返回更新内容
spring.data.rest.return-body-on-update=true
``











