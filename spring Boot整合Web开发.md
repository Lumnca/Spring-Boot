# :maple_leaf:pring Boot整合Web #

<b id="t"></b>

:arrow_down:[返回JSON数据](#a1)

:arrow_down:[静态资源访问](#a2)

<b id="a1"></b>

### :fallen_leaf:返回JSON数据 ###

:arrow_double_up:[返回目录](#t)

### :one:默认实现 ###

JSON是现在目前最主流的前后端数据传输方式，Spring MVC中已经对JSON提供了很好的支持，在Spring Boot中更进一步，默认情况下，当开发者新创建一个Spring Boot项目后，添加Web依赖后，就会默认添加一个jackson-databind作为JSON处理器，此时不需要添加额外的JSON处理器就能返回一段JSON。

添加一个返回对象的方法：

```java
@Controller
@RestController
public class start {

    @GetMapping("/json")
    public book Books(){
        book books = new book();

        books.setName("三国演义");
        books.setAuthor("罗贯中");
        books.setPrice(30.00f);

        return books;
    }
}
```

由于返回的是对象，所以会自动解析为JSON字符格式，接下来运行即可。`http://localhost:86/json`，看到对象JOSN字符串`{"name":"三国演义","author":"罗贯中","price":30.0,"chapter":null}`说明运行成功。

### :two:自定义转换器 ###

前面的转换器是默认的jackson-databind，除此之外，还有GSon和fastjson，这里只用fastjson。有关fastjson可以参考我的另一篇笔记[fastjson](https://github.com/Lumnca/FastJSON)

我们这里介绍spring boot使用fastjson。这里就不需要引入包，直接添加依赖即可：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
    </dependencies>
```


含在web标签中意为清除jackson-databind依赖，下面代码是我们添加的fastjson依赖。接下来就是配置fastjson的HttpMessageConvertert：

```java
package com.example.myboot;

import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.nio.charset.Charset;

@Configuration
public class myFastJosnConfig {
    @Bean
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter(){
    
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();

        FastJsonConfig config = new FastJsonConfig();
        
        config.setDateFormat("yyyy-MM-dd");     
        
        config.setCharset(Charset.forName("UTF-8"));
        
        config.setSerializerFeatures(SerializerFeature.WriteClassName,
        SerializerFeature.WriteMapNullValue,
                SerializerFeature.PrettyFormat,
                SerializerFeature.WriteNullListAsEmpty,
                SerializerFeature.WriteNullStringAsEmpty);
        
        converter.setFastJsonConfig(config);
        return converter;
    }
}

```

由于提供了@Configuration注解，使得上面代码是为fastJson配置，其中包括日期格式，数据编码，是否在JSON中输出类名，是否输出value为null的数据，生成JSON格式化，空集和输出[]而非null。空字符串生成""，而非null等基本配置。配置完成后，还需要配置响应编码，否者返回的JSON中文会乱码，在application.properties添加配置：

```
spring.http.encoding.force-response=true
```

控制器还是和前面一样的配置，运行输出为以下内容：

`{ "@type":"com.example.myboot.book", "author":"罗贯中", "chapter":[], "name":"三国演义", "price":30.0F }`

可以看到比之前多了个类名，这是由于我们的添加了显示类名，可以把配置类的这个参数删除：

```
SerializerFeature.WriteClassName
```

<b id="a2"></b>

### :fallen_leaf:静态资源访问 ###

:arrow_double_up:[返回目录](#t)

Spring Boot含有默认的静态资源访问策略。也可以自己手动配置：

**:one:默认策略**

Spring Boot中对于Sping MVC的自动化配置都在WebMvcAutoConfiguration类中，因此对于默认的静态资源过滤策略可以从这个类中一窥究竟。
在WebMvcAutoConfiguration 类中有一个静态内部类WebMvcAutoConfigurationAdapter，实现了WebMvcConfigurer 接口。WebMvcConfigurer 接口中有一个方法 addResourceHandlers，是用来配置静态资源过滤的。方法在WebMvcAutoConfigurationAdapter类中得到了实现，部分核心代码如下：

```java
public void addResourceHandlers (ResourcelandlerRegistry registry){ String staticPathPattern=this. mvcProperties. getstaticPathPattern(); if(! registry. hasMappingForPattern(staticPathPattern)){
customizeResourceHandlerRegistration(
registry. addResourceHandler(staticPathPattern)
. addResourceLocations(getResourceLocations(
this. resourceProperties. getstaticLocations()))
. setCachePeriod(getSeconds(cachePeriod))
. setCacheControl(cacheControl));
}
```

Spring Boot在这里进行了默认的静态资源过滤配置，其中静态路径默认在上面的类中，定义如下：

```java
private  String staticPathPattern = "/**";
```

this.resourceProperties.getStaticLocations()获取到默认的今天资源位置定义在ResourceProperties：

```
classPath: /META-INF/resources
classPath: /resources
classPath: /static
classPath: /public
```

也就是说你可以在上面四个任意目录（没有该目录可以创建）中添加静态文件，但是系统访问会有优先级，访问顺序如上面列出从上到下。由于IDEA会自动创建一个static文件夹，所以一般放在这里面即可。访问时直接输入文件名即可：  `http://localhost:86/file.txt`

**:two:自定义策略**

如果你想修改默认的静态访问修改，可以通过自定义来解决这个问题。自定义配置有两种方式：

* 1.在配置文件中定义：

可以在application.properties中直接定义过滤规则和静态资源位置，代码如下：

```



```

过滤规则为`/static/**`，静态资源位置为/statics/ 重启项目访问即可。

* 2.JAVA编码定义，也可以通过Java编码来实现，只需要实现WebMvcConfigurer接口即可，然后就是实现该接口的addResourceHandles方法代码如下：

```java

```





























