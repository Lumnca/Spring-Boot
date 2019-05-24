# :golf:Spring 入门  #

<b id="t"></b>

:arrow_double_down:[不使用spring-boot-starter-parent](#a1)

:arrow_double_down:[@SpringBootApplication](#a2)

:arrow_double_down:[Web容器配置](#a3)

:arrow_double_down:[Properties配置](#a4)

:arrow_double_down:[YAML配置](#a5)


<b id="a1"></b>

### :bowling:不使用spring-boot-starter-parent ###

:arrow_double_up: [返回目录](#t)

从第一章的介绍中我们知道了pom.xml文件中添加依赖之前需要添加spring-boot-starter-parent，它主要提供了以下默认配置：

* Java版本默认使用1.8
* 编码格式为UTF-8
* 提供Dependency Management进行项目依赖的版本管理
* 默认的资源过滤与插件配置

spring-boot-starter-parent虽然方便，但是更多的时候使用自己的parent，这个时候如果还想进行项目管理，就需要使用Dependency Management来实现，添加如下代码到pom.xml文件下：

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

其次就是java版本：

```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
```

至于编码，如果采用的是前面说的模板式，可以不用手动添加。

<b id="a2"></b>

### :bowling:@SpringBootApplication ###

:arrow_double_up: [返回目录](#t)

@SpringBootApplication注解是加在项目启动类上的，实际上它是一个组合注解。这个注解是由三个注解组成。如下：

**:one:@SpringBootConfiguration**

@SpringBootConfiguration的功能就是一个配置类，开发者可以在这个类中配置Bean。从这个角度来讲，这个类所扮演的角色有点类似于Spring中的applicationContext.xml文件的角色。

**:two:@EnableAutoConfiguration**

@EnableAutoConfiguration表示开启自动化配置，Spring Boot中的自动化配置是非侵入式的，在任意时刻，开发者都可以使用自定义配置代替自动化配置中的某一个位置。

**:three:@ComponentScan**

@ComponentScan完成包扫描，也是Spring中的功能，由于@ComponentScan注解默认扫描的类都位于当前类所在包的下面，因此建议在实际项目开发中把项目启动类放在根包中。也就是启动注解类所在的类。

虽然项目的启动类也包含@Configuration注解，但是可以一个新的类专门用来配置Bean，这样便于配置管理，这个类只需要加上@Configuration注解即可。

<b id="a3"></b>

### :bowling:Web容器配置 ###

:arrow_double_up: [返回目录](#t)

由于我们使用最多的是tomcat，所以这里只说明tomcat的配置

**:one:常规配置**

在Spring Boot项目中，默认是使用tomcat作为配置容器。可以在main文件夹下的resources文件夹下的application.properties文件。添加如下配置：

```
server.port=8081
server.error.path= /error
server.servlet.session.timeout=30m
server.servlet.context-path=/index
server.tomcat.uri-encoding=UTF-8
server.tomcat.max-threads=500
server.tomcat.basedir=/home/sang/tmp
```

其中按照从上到下的顺序是：

* Web容器的端口号。
* 项目出错时跳转的页面
* session.timeout配置了失效时间，30m代表30分钟，如果不写单位默认单位是秒
* context-path表示项目名称，不配置时默认为/，如果配置了就要在访问路径上加上配置路径。
* uri-encoding表示配置Tomcat请求编码。
* max-threads表示Tomcat最大线程数。
* basedir是一个存放Tomcat运行日志和临时文件的目录，若不配置，则使用系统临时的目录。

配置完成直接运行即可，因为做了配置，所以访问原来网站的路径需要改为`http://localhost:8081/index/index`


**:two:常规配置**

由于Https具有良好的安全性，在开发中得到了越来越广泛的应用，对于个人开发者而言，一个HTTPS证书的价格还是有点贵，在jdk中提供了一个java数字证书管理工具keytool，在\jdk\bin目录下，通过这个工具可以生成一个数字证书，生成命令如下

```
keytool -genkey -alias tomcathttps -keyalg RSA -keysize 2048 -keystore sang.p12 -validity 365
```

其中 -alias 后面跟密匙别名，-keyalg后接加密算法，常用RSA，-keysize 2048 接密匙长度，-keystore后接生成密匙文件名，-validity 密匙有效天数

上面这条命令在cmd下使用，如果你的jdk环境装在C盘，使用管理员权限打开才能使用，按照提示信息填完，如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a7.png)

输入密匙的时候不会显示密码，输入两次一样即可，唯一要求长度要大于6。可以为纯数字。

填完信息后，文件后保存在你所运行cmd的目录下，比如我运行时目录在C:\Windows\System32，所以我需要去这个文件夹找到我命名的sang.p12

文件过多可以搜索，如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a8.png)

把生成的文件复制到项目的根目录下，再在application.properties添加配置：

```
server.ssl.key-store=sang.p12               //密匙文件
server.ssl.key-alias=tomcathttps            //密匙别名
server.ssl.key-store-password=123456        //密匙密码
```


点击运行，出现以下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a9.png)

按照上面操作即可。注意是https，如果以http访问，会访问失败。这是因为Spring Boot不支持同时在配置中启动http和https，这个时候可以配置重定向将http请求重定向为https请求，配置方式如下：

添加一个配置类：

```
package com.example.myboot;


import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class tomcatConfig {
    @Bean
    TomcatServletWebServerFactory tomcatServletWebServerFactory(){
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory(){
            @Override
            protected void postProcessContext(Context context){
                SecurityConstraint constraint = new SecurityConstraint();

                constraint.setUserConstraint("CONFIDENTIAL");

                SecurityCollection collection = new SecurityCollection();

                collection.addPattern("/*");

                constraint.addCollection(collection);

                context.addConstraint(constraint);
            }
        };
        factory.addAdditionalTomcatConnectors(createTomcatConnector());
        return  factory;
    }

    private Connector createTomcatConnector(){
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(8080);
        connector.setSecure(false);
        connector.setRedirectPort(8081);
        return connector;
    }
}
```

这里首先配置一个TomcatServletWebServerFactory类，然后添加一个Tomcat中的Connector（监听8080端口）并将请求转发到8081端口。在浏览器输入`http://localhost:8080/index`会自动转到`https://localhost:8081/index`端口。

这就是tomcat的主要配置。使用这些可以完成基本的需求使用，

<b id="a4"></b>

### :bowling:Properties配置 ###

:arrow_double_up: [返回目录](#t)

虽然Spring Boot中采用了大量的自动化配置，但是对于开发者而言，在实际中还是需要自己手动配置，承载这些配置的就是resources目录下的application.properties文件，前面的web容器配置也就是在这里面配置的，

**:one:类型安全配置属性**

Spring提供了@Value注解以及EnvironmentAware接口来将Spring Environment中的数据注入到属性上，Spring Boot对此进一步提出了类型安全配置属性，这样即使数据量非常多的情况下，也可以更加方便地将配置文件数据注入Bean中，如在application.properties配置文件添加如下配置：

```java
book.name=三国演义
book.author=罗贯中
book.price=40
```

然后就是配置数据注入Bean：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "book")
public class book {
    private String name;
    private String author;
    private Float price;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Float getPrice() {
        return price;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public void setPrice(Float price) {
        this.price = price;
    }
}
```

其中@ConfigurationProperties中的prefix属性决定了要加载配置文件的前缀，如上是book，表明说明加载book。这样他就会把配置文件中的book对应的属性进行绑定。最后只需要在控制器中添加注解引用即可：

```java
@RestController
public class start {

    @Autowired
    book oneBook;

    @GetMapping("/index")
    public String index(){

        return  "书名:"+ oneBook.getName()+"<br>作者："+oneBook.getAuthor()+"<br>价格："+oneBook.getPrice();
    }
}
```

当然一开始输出会中文乱码，这是由于没有对配置文件进行编码导致的，在IDEA的setting中如下修改：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a10.png)

修改后在配置文件处的中文会乱码，重新输入中文即可，在此运行，这样显示就不会乱码。

<b id="a5"></b>

### :bowling:YAML配置 ###

:arrow_double_up: [返回目录](#t)

**:one:常规配置**

YAML是JSON的超集，简洁而强大，是一种专门用来书写配置文件的语言，可以代替application.properties，在创建一个Spring Boot项目时，引入的spring-boot-starter-web依赖间接地引入了snakeyaml依赖，它会实现对YAML配置的解析，YAML的使用非常简单，利用缩进来表示层级关系，并且大小写敏感，在Spring Boot项目中使用YAML只需要在resources文件夹下创建一个application.yml文件即可，并在其中添加如下配置：

```
server:
  port: 86
  servlet:
    context-path: /app
  tomcat:
    uri-encoding: UTF-8
```

这个配置比较容易看懂，就是将86作为端口号，路径为/app，编码使用UTF-8编码。

**:two:复杂配置**

YAML不仅可以配置基础属性，还可以配置复杂属性，如下是完成上面的bean配置：

```
book:
  name: 三国志
  author : 罗贯中
  price : 30
```

这样就可以注入数据，YAML还支持列表的配置，如下：

```
book:
  name: 三国志
  author : 罗贯中
  price : 30
  chapter :
    - 桃园结义
    - 董卓乱政
```

使用 - 代表列表项，对应的需要在bean中添加属性：

```java
 private List<String> chapter;
 
     public List<String> getChapter() {
        return chapter;
    }

    public void setChapter(List<String> chapter) {
        this.chapter = chapter;
    }
```

然后就可以访问了：

```java
@RestController
public class start {

    @Autowired
    book oneBook;

    @GetMapping("/index")
    public String index(){

        return  "书名:"+ oneBook.getName()+"<br>作者："+oneBook.getAuthor()+"<br>价格："+oneBook.getPrice()
                +"章节:"+oneBook.getChapter().get(0)+"<br>"+oneBook.getChapter().get(1);
    }
}

```

当然也可以支持更复杂的配置，只需要满足缩进关系即可：


```
customer :
  name : 刘二狗
  id : KJ132321
  ownBook :
    - name : 三国演义
      author : 罗贯中
      price : 30
      chapter :
        - 1
        - 2
    - name : 西游记
      author : 吴承恩
      price : 40
      chapter :
        - 1
        - 2
```

添加顾客类：

```java
@Component
@ConfigurationProperties(prefix = "customer")
public class customer {
    private String name;
    private String id;
    private List<book> ownBook;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public void setOwnBook(List<book> ownBook) {
        this.ownBook = ownBook;
    }

    public List<book> getOwnBook() {
        return ownBook;
    }
}
```

最后再在控制器中调用即可。

```java
@RestController
public class start {

    @Autowired
    book book;
    @Autowired
    customer customer;

    @GetMapping("/index")
    public String index(){

        book book1 = customer.getOwnBook().get(0);
        return  "客户名："+customer.getName()+"<br>"+
                "购买书籍:"+book1.getName()+"<br>"+
                customer.getOwnBook().get(1).getName();
    }
}
```








