# :maple_leaf:pring Boot整合Web #

<b id="t"></b>

:arrow_down:[整合JdbcTemplate](#a1)

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

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
```

数据库配置：

```xml

```
