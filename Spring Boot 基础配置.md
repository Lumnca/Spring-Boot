# :golf:Spring 入门  #

<b id="t"></b>

:arrow_double_down:[不使用spring-boot-starter-parent](#a1)

:arrow_double_down:[@SpringBootApplication](#a2)


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

**@SpringBootConfiguration**

@SpringBootConfiguration的功能就是一个配置类，开发者可以在这个类中配置Bean。从这个角度来讲，这个类所扮演的角色有点类似于Spring中的applicationContext.xml文件的角色。

**@EnableAutoConfiguration**

@EnableAutoConfiguration表示开启自动化配置，Spring Boot中的自动化配置是非侵入式的，在任意时刻，开发者都可以使用自定义配置代替自动化配置中的某一个位置。

**@ComponentScan**

@ComponentScan完成包扫描，也是Spring中的功能，由于@ComponentScan注解默认扫描的类都位于当前类所在包的下面，因此建议在实际项目开发中把项目启动类放在根包中。也就是启动注解类所在的类。

虽然项目的启动类也包含@Configuration注解，但是可以一个新的类专门用来配置Bean，这样便于配置管理，这个类只需要加上@Configuration注解即可。















