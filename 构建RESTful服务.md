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

