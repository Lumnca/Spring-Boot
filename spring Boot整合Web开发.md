# :maple_leaf:pring Boot整合Web #

<b id="t"></b>

:arrow_down:[返回JSON数据](#a1)

:arrow_down:[静态资源访问](#a2)

:arrow_down:[文件上传](#a3)

:arrow_down:[@ControllerAdvice](#a4)

:arrow_down:[CORS支持](#a5)

:arrow_down:[整合Servlet，Filter和Listener](#a6)

:arrow_down:[配置AOP](#a7)

:arrow_down:[自定义](#a8)

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

```java
spring.http.encoding.force-response=true
spring.mvc.static-path-pattern=/static/**
spring.resources.static-locations=classpath:/static/
```

过滤规则为`/static/**`，静态资源位置为/statics/ 重启项目访问即可。

* 2.JAVA编码定义，也可以通过Java编码来实现，只需要实现WebMvcConfigurer接口即可，然后就是实现该接口的addResourceHandles方法代码如下：

```java
@Configuration
public class myWebStaticConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry){
        registry.addResourceHandler("/static/**").
                addResourceLocations("classpath:/static/");
    }
}
```


<b id="a3"></b>

### :fallen_leaf:文件上传 ###

:arrow_double_up:[返回目录](#t)

在以前学的的东西可以知道，文件上传我们是使用包或者servlet3.0来完成文件上传，在Spring Boot中提供了文件上传自动化配置类MultipartAutoConfiguration中，默认也是采用StandardServletMultipartResolver。在这个类中展示到如果开发者没有提供MultipartResolve，那么默认采用的就是StandardServletMultipartResolver，因此在Spring Boot中上传文件甚至可以到零配置。

**:one:单文件上传**

项目只要有spring-boot-start-web依赖。就可以直接使用，在静态文件夹创建一个上传的html文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/upload" method="POST" enctype="multipart/form-data">
        <input type="file" name="uploadFile" value="选择文件">
        <input type="submit" value="上传">
    </form>
</body>
</html>
```

和以前的上传文件形式一样，方法必须为post，且必须含有enctype="multipart/form-data"。下面编写接口：


```java
@RestController
public class file {
    @PostMapping("/upload")
    public  String upload(MultipartFile uploadFile, HttpServletRequest request){
        if (uploadFile.isEmpty()) {
            return "上传失败，请选择文件";
        }

        String fileName = uploadFile.getOriginalFilename();
        String filePath = System.getProperty("user.dir") ;
        System.out.println(filePath+fileName);
        File dest = new File(filePath+"/"+fileName);
        try {
            uploadFile.transferTo(dest);
            return "success";
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "fail！";
    }
}
```

上面就是上传文件简单的代码，该文件会上传到项目的根目录下。因为` System.getProperty("user.dir") `获取的是项目的路径名称，transferTo为文件上传方法。这样就完成了一个简单的文件上传。当然可以在配置文件中为文件上传进行配置其配置如下：

```
spring.servlet.multipart.enabled=true
spring.servlet.multipart.file-size-threshold=0
spring.servlet.multipart.location=F:\\myboot
spring.servlet.multipart.max-file-size=1MB
spring.servlet.multipart.max-request-size=10MB
spring.servlet.multipart.resolve-lazily=false
```

* 其中第一行是是否开启文件上传，默认true。
* 第二行为文件写入磁盘阈值，默认为0（直接写入磁盘）
* 第三行为文件上传的临时保存位置
* 第四行表示单文件上传的最大大小。默认为1MB
* 第五行是多文件上传的总大小。默认为10MB
* 最后一样是文件是否延迟解析。

**:two:多文件上传**

多文件上传与单文件上传一致，需要修改的是表单：

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
    <input type="file" name="uploadFiles" value="选择文件" multiple>   <!--添加一个 multiple属性-->
    <input type="submit" value="上传">
</form>
```

对于接口处理就是文件数组，而不是以前的单文件参数：

```
@RestController
public class file {
    @PostMapping("/upload")
    public  String upload(MultipartFile[] uploadFiles, HttpServletRequest request){
        int a = 0;
        for (MultipartFile file:uploadFiles
             ) {
            if (!file.isEmpty()) {
                String fileName = file.getOriginalFilename();
                String filePath = System.getProperty("user.dir") ;
                System.out.println(filePath+fileName);
                File dest = new File(filePath+"/target/classes/static/"+fileName);
                try {
                    file.transferTo(dest);
                    a++;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return "上传文件数:"+uploadFiles.length+"  成功上传"+a+"个文件";
    }
}
```

核心就在于我们需要使用迭代来完成文件上传工作，这和单文件上传思路是一样的。

<b id="a4"></b>

### :fallen_leaf:@ControllerAdvice ###

:arrow_double_up:[返回目录](#t)

**:one:全局异常处理**

@ControllerAdvice最常见的使用场景就是全局异常处理。@ControllerAdvice结合@ExceptionHandler定义全局异常捕获机制：

```java
@ControllerAdvice
public class CustomExceptionHandler {
    @ExceptionHandler(Exception.class)
    public void upLoadException(HttpServletResponse response)throws IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.write("出现错误");
        out.flush();
        out.close();
    }
}
```

Exception.class是所有错误的类，它可以捕获所有出错的类型，并做出一致的错误反馈。当然在这个类里面添加任意参数和返回类型，可以返回的是一段话，也可以是一个错误界面，只需要输入对应的方法即可。

**:two:添加全局数据**

@ControllerAdvice也是一个全局数据处理组件，因此也可以在@ControllerAdvice中配置全局数据，使用@ModelAttribute注解进行配置，如下：

```java
@ControllerAdvice
public class GlobalConfig {
    @ModelAttribute(value = "info")
    public Map<String,String>  userInfor(){
       HashMap<String,String> map = new HashMap<>();
       map.put("Name","小明");
       map.put("sex","男");
       return  map;
    }
    @ModelAttribute(value = "welcome")
    public  String Hello(){
        return "Hello Spring Boot";
    }
    @ModelAttribute(value = "number")
    public int Number(){
        return 123456;
    }
}
```

其中@ModelAttribute注解的value参数就是全局数据的key。我们可以在控制器来访问他们：

```java
    @GetMapping("infor")
    public String infor(Model model){
        String infor = "";
        //获取全局数据
        Map<String,Object> map = model.asMap();
        
        //获取key值
        Set<String> keySet = map.keySet();

        //key迭代器
        Iterator<String> iterator = keySet.iterator();

        while (iterator.hasNext()){
            //寻找下一个key值
            String key = iterator.next();
            //获取value值
            Object value = map.get(key);

            infor = infor+key+">>>>>>"+value.toString()+"<br>";
        }
        return  "全局数据："+infor;
    }
```

像这样就可以做到全局Mode了数据共用。


**:three:请求参数预处理**

@ControllerAdvice也可以结合@InitBinder还能实现请求参数预处理，即将表单中的数据绑定到实体类上时进行一些额外处理。

例如有两个实体类：

author类：
```java
public class author {
    private String name;
    private  int age;

    public String getName() {
        return name;
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
}
```

book类：

```java
public class book {
    private String name;
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

    public void setPrice(Float price) {
        this.price = price;
    }
}
```

我们需要在控制器完成两个类的数据绑定：

```java
    public String infor(author authors,book books){
        return "作者名："+authors.getName()+" 年龄："+authors.getAge()+"<br>"
        +"书名："+books.getName()+" 价格："+books.getPrice();
    }
```

这样的形式两个类参数无法绑定而且两个类均匀有name属性会产生混淆。所以这个时候我们可以采用@ControllerAdvice结合@InitBInder可以顺利的解决问题，配置如下：

首先给控制器方法添加@ModelAttribute注解：

```java
    @GetMapping("infor")
    @ResponseBody
    public String infor(@ModelAttribute("a") author authors,@ModelAttribute("b") book books){
        return "作者名："+authors.getName()+" 年龄："+authors.getAge()+"<br>"
        +"书名："+books.getName()+" 价格："+books.getPrice();
    }
```

然后再配置前面所说的@ControllerAdvice：

```java
@ControllerAdvice
public class GlobalConfig {
    @InitBinder("b")
    public void  init(WebDataBinder binder){
        binder.setFieldDefaultPrefix("b.");
    }
    @InitBinder("a")
    public void init2(WebDataBinder binder){
        binder.setFieldDefaultPrefix("a.");
    }
}
```

在 GlobalConfig类中创建两个方法，第一个代表的是 @InitBinder("b")处理@ModelAttribute("b")所对应的参数。第二个代表的是@InitBinder("a")处理@ModelAttribute("a")所对应的参数。


setFieldDefaultPrefix意为设置前缀，好区分属性名，接下来就可以在html界面传值：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/infor" method="GET">
   作者名: <input type="text" name="a.name" ><br>
   作者年龄： <input type="text" name="a.age" ><br>
    书名：<input type="text" name="b.name" ><br>
    价格：<input type="text" name="b.price" ><br>
    <input type="submit" value="完成">
</form>
</body>
</html>
````

这样就可以实现属性一一对应传值。

**:four:自定义错误界面**

前面说过有过全局处理来实现错误处理，当然我们也可以使用html界面来实现不同的错误界面。spring boot自带有错误界面，我们可以去修改这个界面,对于修改错误界面，不需要我们去配置选项，系统自定义的错误文件在static下的error下的对应界面（没有可以自行创建）如下：

我们自己创建了这个文件：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a11.png)

这样我们就覆盖了原有的错误界面，接下来可以进行测试：

输入一个不存在的url路径即可看到404界面

对于500错误，我们可以简单的写个错误程序：


```java
    @GetMapping("/hello")
    public String hello(){
        int  i = 1/0;
        return "hello";
    }
```

访问该界面即可看到，注意，一定要取消前面的异常捕获，要不然会被转到异常捕获界面。

上面是详细的错误方式404,500，如果想对这一类的处理可以将html文件改成4xx.html,和5xx.html。如果你设置了404界面和4xx，那么优先级是404，然后就是4xx.

可以使用模板也来显示详细错误信息：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org/">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h3>出错了</h3>
    时间:<p th:text="${timestamp}"></p><br>
    错误码:<p th:text="${status}"></p><br>
    错误类型:<p th:text="${error}"></p><br>
    错误信息:<p th:text="${message}"></p><br>
    访问路径: <p th:text="${path}"></p><br>
</body>
</html>
```

前提是添加了依赖，和创建在模板文件下。修改默认界面的方法还有很多这里就不一一介绍了，可自行网上查阅资料。

<b id="a5"></b>

### :fallen_leaf:CORS支持 ###

:arrow_double_up:[返回目录](#t)

CORS是由W3C制定的一种跨域资源共享技术标准，其目的就是为了解决前端跨域请求。在JavaEE开发中，最常见的前端跨域请求解决方案就是JSONP，但是JSONP只支持GET请求，这是一个很大的缺陷，而CORS则支持多种HTTP请求方法，以CORS中的GET为例

`响应头中有一个Acess-Control-Allow-Origin字段，用来记录可以访问该资源的域。当浏览器收到这样的响应头信息之后，提取出Access-Control-Allow-Origin字段中的值，发现该值包含当前页面所在的域，就知道这个跨域是被允许的，因此就不再对前端的跨域请求进行限制。这就是GET请求的整个跨域流程，在这个过程中，前端请求的代码不需要修改，主要是后端进行处理。这个流程主要是针对GET、POST以及HEAD请求，并且没有自定义请求头，如果用户发起一个DELETE请求、PUT请求或者自定义了请求头，流程就会稍微复杂一些。`

在Spring Boot中配置CORS步骤如下：

只要添加了Web依赖就可以直接使用，创建控制器：

```java
@RequestMapping("/cors")
public class cors {
        @GetMapping("/get")
        @CrossOrigin(value = "http://localhost:86",maxAge = 1800,allowedHeaders = "*")
        public String get(){
            return  "Receive : CORS";
        }
}
```

上面代码中@CrossOrigin注解代表跨域配置，其中value是表示支持的域，这里表示来自`http://localhost:86`端口的请求运行跨域到本服务中，maxAge代表请求的有效时间，，这个属性默认是1800秒即30分钟。allowedHeaders代表运行的请求头，`*` 代表所有的请求头都被允许。这种方式是一种细粒的配置，可以控制到每个方法上，当然可以使用全局配置,在设置的全局配置文件中添加如下方法：

```java
    public void  addCorsMappings(CorsRegistry registry)
    {
        registry.addMapping("/cors/**")
                .allowedHeaders("*")
                .allowedMethods("*")
                .maxAge(1800)
                .allowedOrigins("http://localhost:86");
    }
```

最后测试要使用两个不同的域，不能在同一项目中运行，因为项目的域是一样的，所以需要启动两个域，在其中一个域上添加CORS请求：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p id="infor"></p>
<script>
    var xmlHttp = new XMLHttpRequest();        //创建XMLHttpRequest实例
    xmlHttp.open("GET","http://localhost:87/cors/get",false);      //打开请求
    xmlHttp.send(null);                        //发送
    document.getElementById("infor").innerText = xmlHttp.responseText
</script>
</body>
</html>
```

<b id="a6"></b>

### :fallen_leaf:整合Servlet，Filter和Listener ###

:arrow_double_up:[返回目录](#t)

一般情况下，使用Spring，Spring MVC这些框架后，基本就告别了Servlet，Filter和Listener了，但是有时在整合一些第三方框架时，可能还是会使用Servlet，Spring Boot中对于这些整合这些基本Web组件也是提供了很好的支持。，如下：

添加一个Servlet：

```java
@WebServlet("/my")
class MyServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        try {
            response.sendRedirect("/json");
        }
        catch (Exception e){
            e.printStackTrace();
        }

    }
    protected void doPost(HttpServletRequest request, HttpServletResponse response){
        doGet(request,response );
    }
}
```

为了能够使用这些，必须使用在开始类使用注解`@ServletComponentScan`

```java
@SpringBootApplication
@ServletComponentScan
public class MybootApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybootApplication.class, args);
    }
}
```

当然也可以使用Filter 过滤没有session信息不能进入json方法，如下：

```java
@WebFilter("/json")       //监听/json端口
class MyFileter implements Filter{
    public void init(FilterConfig filterConfig){
        System.out.println("监听器启动");
    }
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse)servletResponse;
        HttpServletRequest  request = (HttpServletRequest)servletRequest;

        HttpSession session = request.getSession();

        if (session.getAttribute("user")==null) {          //session会话对象对空，即没有登录成功
            response.sendRedirect("/upload.html");            //返回界面重新登录
            System.out.println("没有得到认证信息");
        }
        else{
            System.out.println("认证成功");
            filterChain.doFilter(request,response);                    //成功则把请求传回过滤链，必须含有
        }
    }
    public void destroy(){

    }
}
```

Filter也必须使用`@ServletComponentScan`注解。

也可以使用监听器来监视事件，如下是对Session信息的监听：

```java
@WebListener
class MyListenter implements HttpSessionAttributeListener,ServletRequestListener{
    public void requestInitialized(ServletRequestEvent sre)
    {

    }
    public  void  attributeAdded(HttpSessionBindingEvent se){
        System.out.println("Session信息："+se.getName()+" >>"+se.getValue());
    }
    public  void attributeRemoved(HttpSessionBindingEvent se){

    }
    public  void attributeReplaced(HttpSessionBindingEvent se){

    }
}
```

如上每添加一个Session信息，就会输出一条信息。同样的也需要使用`@ServletComponentScan`注解。像这样就可以使用Servlet里的服务配置。

<b id="a7"></b>

### :fallen_leaf:配置AOP ###

:arrow_double_up:[返回目录](#t)

AOP面向切面编程，首先需要了解这样一个场景：公司有一个人力资源管理系统目前已经上线，但是系统运行不稳定，有时运行得很慢，为了检测出到底是哪个环节出问题了，开发人员想要监控每一个方法的执行时间，再根据这些执行时间判断出问题所在。当问题解决后，再把这些监控移除掉。系统目前已经运行，如果手动修改系统中成千上万个方法，那么工作量未免太大，而且这些监控方法以后还要移除掉；如果能够在系统运行过程中动态添加代码，就能很好地解决这个需求。这种在系统运行时动态添加代码的方式称为面向切面编程（AOP）。Spring框架对AOP提供了很好的支持。在AOP中，有一些常见的概念需要读者了解。

`Joinpoint（连接点）：类里面可以被增强的方法即为连接点。例如，想修改哪个方法的功能，那么该方法就是一个连接点。`

`Pointcut（切入点）：对Joinpoint进行拦截的定义即为切入点。例如，拦截所有以insert开始的方法，这个定义即为切入点。`

`Advice（通知）：拦截到Joinpoint之后所要做的事情就是通知。例如，上文说到的打印日志监控。通知分为前置通知、后置通知、异常通知、最终通知和环绕通知。`

`Aspect（切面）：Pointcut和Advice的结合。`

`Target（目标对象）：要增强的类称为Target`

Spring Boot在Spring的基础上对AOP的配置提供了自动化配置解决方案`spring-boot-starter-aop`使开发者能够更加敏捷地使用AOP，如下是对AOP的简单运用，首先在Web项目中添加依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

接下来创建一个需要测试方法运行的所需时间的类：

```java
package com.example.myboot;

import org.springframework.stereotype.Service;
import java.util.Date;

@Service
public class UserService {
    public  String getUserById(){
        System.out.println("正在获取用户信息"+new Date().toString());
        return  "user id is : ...";
    }
    public  void  deleteUserById(){
        System.out.println("正在删除用户ID..."+new Date().toString());
    }
}
```

接下里创建切面：

```
package com.example.myboot;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
@Aspect
public class LogAspect {
    private long spendTime;

public class LogAspect {
    private long spendTime;

    @Pointcut("execution(* com.example.myboot.UserService.*(..))")
    public  void pc1(){
        System.out.println("------");
    }
    @Before(value = "pc1()")
    public void before(JoinPoint jp) {
        String name = jp.getSignature().getName();
        spendTime = new Date().getTime();
        System.out.println(name +"方法开始执行...");
    }
    @After(value = "pc1()")
    public  void after(JoinPoint jp) {
        String name = jp.getSignature().getName();
        long time = new Date().getTime()-spendTime;
        System.out.println(name +"方法执行结束，用时："+ time+"ms");
    }

    @AfterReturning(value = "pc1()",returning = "result")
    public void afterReturning(JoinPoint jp,Object result){
        String name = jp.getSignature().getName();
        System.out.println(name+"方法的返回值为:"+result);
    }
    @AfterThrowing(value = "pc1()",throwing = "e")
    public void afterThrowing(JoinPoint jp,Exception e){
        String name = jp.getSignature().getName();
        System.out.println(name+"方法出现了异常："+e);
    }
    @Around("pc1()")
    public Object around(ProceedingJoinPoint pjp)throws Throwable{
        return pjp.proceed();
    }
}
}
```

这里需要说明一下：

pc1()类方法，后面的注解value必须的是这个值，否者项目报错。

`@Pointcut("execution(* com.example.myboot.UserService.*(..))")` 是切入点定义。execution 中第一个`*` 代表任意方法返回值，第二个`*` 代表该包下的任意方法，`（..）` 代表的是方法的参数。整个语句联合起来就是说明在`com.example.myboot.UserService`包下的所有方法都是切入点。注意这个名称是我的项目包名，使用自己的包路径需要参考包路径。

`@Before`注解表示在目标方法执行前执行，可以通过JoinPoint类型来获取方法的名称与参数等信息。

`@After`注解表示在目标方法执行结束后执行。

`@AfterReturning`是运行含有返回值的方法之后所执行的方法，在方法中定义了参数Object result就表示可以是任意的返回值，即可以返回任意的具有返回值的方法，如果只想指定特定返回值的方法，只需要把result的参数类型定为Object即可。

`@afterThrowing`是捕获抛出错误的方法之后执行的方法，同样的Exception e指定所有的抛出类型。

`@Around`表示环绕通知，可以实现前面所有的方法通过调用proceed()方法来实现目标方法继续执行。开发者可以在这里面修改目标方法的执行参数，返回值等，并且可以在这里面处理异常。

配置完成后，使用控制器调用测试类中方法：

```java
    @Autowired
    UserService userService;
    @GetMapping("/getUserById")
    public  String getUserById(){
        return userService.getUserById();
    }
    @GetMapping("/deleteUserById")
    public  void deleteUserById(){
       userService.deleteUserById();
    }
```


执行控制器方法即可看到方法的执行信息。


<b id="a8"></b>

### :fallen_leaf:自定义 ###

:arrow_double_up:[返回目录](#t)

在Spring Boot中，有许多的自定义设置，比如默认开始界面，标题等。下面说下修改这些默认配置。

**自定义起始界面**

自定义起始界面是在Spring Boot项目启动前首先回去静态文件夹寻找index.html作为网站首页文件，若查找不到则会去查找动态的index作为界面

修改自定义起始界面：如果是静态Html界面的话，直接在static目录下添加文件即可，动态的话则需要在templates文件下创建一个index.html（使用模板创建）然后在控制器中添加一个/index的请求方法：

```java
    @GetMapping("/index")
    public String index(){
        return  "index";
```

最后在直接访问端口即可`http://localhost:86/`

**自定义favicon**

favicon.ico是浏览器选项卡左上角的图标，可以放在静态资源路径下或者类路径下，静态资源路径下的favicon.ico优先级高于类路径下的favicon.ico。
可以使用[在线转换网站](https:/jinaconvert.com/cn/convert-to-ico.php)将一张普通图片转为.ico图片，转换成功后，将文件重命名为favicon.ico，然后复制到resources/static目录下，如图:












