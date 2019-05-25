# :lollipop:spring Boot整合视图层 #

在目前的企业级应用开发中，前后端分离是趋势，但是视图层技术还占有一席之地，Spring Boot对视图层技术提供了很好的支持，官方推荐使用的模板引擎是Thymeleaf，不过像FreeMaker也支持，JSP技术在这里并不推荐使用，下面是对这两种技术的介绍：

:arrow_heading_down:[整合Thymeleaf](#a1)

:arrow_heading_down:[整合FreeMarker](#a2)


<b id="a1"></b>

### :hourglass:整合Thymeleaf ###

Thymeleaf是新一代java模板引擎，与传统模板不一样的是，它还支持HTML原型，这样前后端使用都非常的方便，整合Thymeleaf步骤如下：

**:one:添加依赖**

在pom.xml文件中的dependencies标签中添加如下内容：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

**:two:配置Thymeleaf**

如果是使用IDEA创建的项目的话，模板默认生成的位置在resources文件夹下的templates包中，默认模板后缀名为html，如果想自定义配置，可以在application.properties文件中添加配置：

```

```

也可以在application.yml添加文件，两者完成的是一样的配置：

```
spring:
 thymeleaf:
   cache: false
   prefix: classpath:/templates/
   suffix: .html
   encoding: UTF-8
   content-type: text/html
   mode: HTML5
```

**:three:配置控制器**

我们在控制中配置来映射到我们的html界面

```java
@Controller
public class start {

    @GetMapping("/customer")
    public ModelAndView customer(){
        ModelAndView mvc = new ModelAndView();
        mvc.setViewName("customer");

        mvc.addObject("message","Hello Spring Boot!");
        return mvc;
    }
```

注意，这里只能用@Controller不能用@RestController,后者会直接输出json格式的/index,而不是跳转@RequestMapping("/customer")
@GetMapping("//customer")两种方式都是可以的，SpringBoot中允许有不同类型的请求方式。

对于类型ModelAndView是返回一个模型界面，addObject为添加键值对数据，setViewName("customer")代表映射customer.html界面，所以下一步我们需要添加这个界面


**:four:创建视图**

在resources目录下的templates目录下创建customer.html，具体如下：

```html
<!DOCTYPE html>
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
    <h3 th:text="${message}"></h3>
</body>
</html>
```

* 首先在第二行的html起始标签添加`xmlns:th="http://www.thymeleaf.org"`代表Thymeleaf的名称空间。

* `<h3 th:text="${message}"></h3>`代表使用模板的th，text类型显示文本内容。

一开始写上后会有错误提示，这个没有关系，直接运行，在url输入：`http://localhost:8080/customer` 即可。若出现字体信息，说明配置成功。

<b id="a2"></b>

### :hourglass:整合FreeMarker ###

FreeMarker也是一个非常古老的模板引擎，可以用在web和非web环境中，与Thymeleaf不同的是，FreeMarker需要解析才能在浏览器中显示。Spring Boot对FreeMarker也提供了很好的支持，其整合步骤如下：

**:one:添加依赖**

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
```

FreeMarker与Thymeleaf一样文件都是在templates文件夹下，当然也可以修改，修改如上面配置一样，只需要把Thymeleaf替换成FreeMarker即可。控制器配置也是一样这里都不做介绍。

**:two:视图**

这里不一样的是创建customer.ftl文件，而不是html文件

```html
<!DOCTYPE html>
<html lang="en" >
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
    <h3>${message}</h3>
</body>
</html>
```

同样的保存运行，输入url看到文字信息代表成功。


当然对于这两种的模板语法可以自己下去学习，参考网站为：

* [thymeleaf](https://www.thymeleaf.org/)

* [freemarker](https://freemarker.apache.org/)

当然对于目前流行的前后端分离技术，那么就不需要视图层技术，后端直接提供接口即可。
















