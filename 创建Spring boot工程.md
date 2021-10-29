# :golf:Spring 入门  #

<b id="t"></b>

:arrow_double_down:[开发第一个Spring Boot程序](#a1)

:arrow_double_down:[Spring Boot的简单便捷创建方式](#a2)


<b id="a1"></b>

### :bowling:开发第一个Spring Boot程序 ###

:arrow_double_up: [返回目录](#t)

Spring Boot 可以通过很多中方式来创建，最通用的就是通过Maven了，因为大多数的IDE都支持Maven。首先就是Maven的配置与安装，可以参考这篇文章[IDEA配置Maven](https://zhuanlan.zhihu.com/p/48831465)

配置完毕后，我们可以建立一个最简单的Spring-boot程序。如下操作：

在工程界面选中Maven，我们这里暂时不使用框架，什么都不选，直接next:

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a1.png)

后面命名直接命名，没有什么苛刻的要求，创建完毕后来到我们的主界面：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a2.png)

这里的pom.xml即为我们的主配置文件，接下来为了能够启用Spring Boot，添加配置：

**:one:添加依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>as</groupId>
    <artifactId>as</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

这是添加Spring Boot的依赖和配置，就不需要像以前那样去获取包，再添加包。接下来做的是就是编写启动类：

**:two:编写启动类**

在src的main文件夹下创建包并创建一个启动类app：

```java
package Start;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication   //自动化配置加注解扫描
public class App {
    public static  void main(String[] args){
        SpringApplication.run(App.class,args);
    }
}
```

要注意的是一定要在包下创建启动类。

**:three:web主体**

在与启动类同一个包下添加一个控制器类：

```java
package Start;


import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController   //控制器注解
public class index {
    @GetMapping("/index")  //路径注解
    public String index(){
        return  "Hello Spring Boot!";   //显示文字内容
    }
}

```

至此，所有配置就已经完成，接下来就是项目启动，直接运行main类，控制器若出现如下所示的图：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a3.png)

说明配置成功，在浏览器上面输入url ： `http://localhost:8080/index` 看到显示文字，说明配置完成。

对于配置好Maven的项目，也可以使用命令行来启动项目和打包项目。

```
mvn -v 版本信息

mvn spring-boot:run  项目启动

mvn package 项目打包
```

当然如果你需要一个jar包需要在配置文件中添加如下内容：

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

<b id="a2"></b>

### :bowling:Spring Boot的简单便捷创建方式 ###

:arrow_double_up: [返回目录](#t)

**:one:在线创建**

Spring Boot提供一种在线创建的方式，打开网页[Spring模板创建](https://start.spring.io/) `(https://start.spring.io/)`选中对应的版本与工具即可创建，下载生成的模板包，解压后使用IDEA打开即可使用。

**:two:使用IDEA创建**

在工程面板哪里选中如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a4.png)

可以看到也是和前面在线创建一样，默认的会从网站下，当然如果本地有你的boot模板，也可以选择下面那个套用。

点击一下步，输入项目信息：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a5.png)

点击下一步，选择依赖，选择后就不用再去配置pom.xml启动项目文件：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a6.png)

后面都点击下一步，就完成了创建，建议使用这种方式来构建项目。

完成后打开，你可以发现和我们一样之前创建的是一样的。



















