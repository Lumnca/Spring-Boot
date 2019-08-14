# :golf:Spring Boot整合WebSocket #

<b id="t"></b>

:arrow_double_down:[WebSocket](#a1)

:arrow_double_down:[Spring Boot整合WebSocket](#a2)

<b id="a1"></b>

### :bowling:WebSocket ###

:arrow_double_up: [返回目录](#t)

在HTTP协议中，所有的请求都是由客户端发起的，由服务端进行响应，服务端无法向客户端推送信息，但是在一些需要即时通讯的应用中，又不可避免地需要服务端向客户端推送信息，传送消息主要由以下几种方法：

**1.轮询**

轮询是最简单的一种解决方案，所谓轮询，就是客户端在固定的时间间隔下不停地向服务端发送请求，查看服务端是否有最新的数据，若服务端有最新的数据，则返回给客户端，若服务端没有，则返回一个空的JSON或者XML文档。轮询对开发人员而言实现方便，但是弊端也很明显；客户端每次都要新建HTTP请求，服务端要处理大量的无效请求，在高并发场景下会严重拖慢服务端的运行效率，同时服务端的资源被极大的浪费了，因此这种方式并不可取。

**2.长轮询**

长轮询是传统轮询的升级版，当聪明的工程师看到轮询所存在的问题后，就开始解决问题，于是有了长轮询。不同于传统轮询，在长轮询中，服务端不是每次都会立即响应客户端的请求，只有在服务端有最新数据的时候才会立即响应客户端的请求，否则服务端会持有这个请求而不返回，直到有新数据时才返回。这种方式可以在一定程度上节省网络资源和服务器资源，但是也存在一些问题，例如：

`如果浏览器在服务器响应之前有新数据要发送，就只能创建一个新的并发请求，或者先尝试断掉当前请求，再创建新的请求。`

`TCP和HTTP规范中都有连接超时一说，所以所谓的长轮询并不能一直持续，服务端和客户端的连接需要定期的连接和关闭再连接，这又增大了程序员的工作量，当然也有一些技术能够延长每次连接的时间，但毕竟是非主流解决方案。`

**3.Applet 和Flash**

Applet和Flash都已经是明日黄花，不过在这两个技术存在的岁月里，除了可以让我们的HTML页面更加绚丽之外，还可以解决消息推送问题。开发者可以使用Applet和Flash来模拟全双工通信，通过创建一个只有1个像素点大小的透明的Applet或者Flash，然后将之内嵌在网页中，再从Applet或者Flash的代码中创建一个Socket连接进行双向通信。这种连接方式消除了HTTP协议中的诸多限制，当服务器有消息发送到客户端的时候，开发者可以在Applet或者Flash中调用JavaScript函数将数据显示在页面上，当浏览器有数据要发送给服务器时也一样，通过Applet 或者Flash来传递。

这种方式真正地实现了全双工通信，不过也有问题，说明如下：

`浏览器必须能够运行Java或者Flash。`

`无论是Applet还是Flash都存在安全问题。`

`随着HTML5标准被各浏览器厂商广泛支持，Flash下架已经被提上日程（Adobe宣布2020年正式停止支持Flash）。其实，传统的解决方案不止这三种，但是无论哪种解决方案都有自身的缺陷，于是有了WebSocket。`

WebSocket是一种在单个TCP连接上进行全双工通信的协议，已被W3C定为标准。使用WebSocket可以使得客户端和服务器之间的数据交换变得更加简单，它允许服务端主动向客户端推送数据。在WebSocket协议中，浏览器和服务器只需要完成一次握手，两者之间就可以直接创建持久性的连接，并进行双向数据传输。
WebSocket 使用了HTTP/1.1的协议升级特性，一个WebSocket 请求首先使用非正常的HTTP请求以特定的模式访问一个URL，这个URL有两种模式，分别是ws和wss，对应HTTP协议中的HTTP和HTTPS，在请求头中有一个Connection:Upgrade字段，表示客户端想要对协议进行升级，另外还有一个Upgrade:websocket字段，表示客户端想要将请求协议升级为WebSocket协议。
这两个字段共同告诉服务器要将连接升级为WebSocket这样一种全双工协议，如果服务端同意协议升级，那么在握手完成之后，文本消息或者其他二进制消息就可以同时在两个方向上进行发送，而不需要关闭和重建连接。此时的客户端和服务端关系是对等的，它们可以互相向对方主动发送消息。和传统的解决方案相比，WebSocket主要有如下特点：

* WebSocket使用时需要先创建连接，这使得WebSocket成为一种有状态的协议，在之后的通信过程中可以省略部分状态信息（例如身份认证等）。

* WebSocket连接在端口80（ws）或者443（wss）上创建，与HTTP使用的端口相同，这样，基本上所有的防火墙都不会阻止WebSocket连接。

* WebSocket使用HTTP协议进行握手，因此它可以自然而然地集成到网络浏览器和HTTP服务器中，而不需要额外的成本。

* 心跳信息（ping 和pong）将被反复的发送，进而保持websockt连接一直处于活跃状态。

* 使用该协议，当消息启动或者到达的时候，服务端和客户端都可以知道Websocket连接关闭时将发送一个特殊的关闭消息。

* WebSocket支持跨域，可以避免Ajax的限制。

WebSocket既然有这么多的优势，使用场景当然也是非常广泛的，例如：

* 在线股票网站

* 即时聊天

* 多人在线游戏

* 应用集群通信

* 系统性能实时监控

<b id="a2"></b>

### :bowling:Spring Boot整合WebSocket ###

:arrow_double_up: [返回目录](#t)

了解之后下面介绍怎么使用

添加依赖：

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>webjars-locator-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>sockjs-client</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>stomp-websocket</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1</version>
        </dependency>
    </dependencies>
```


这里需要引入jQuery，接下来配置WebSocket，Spring框架提供了基于WebSocket的STOMP支持，STOMP是一个简单的协议，通常用于中间服务器在客户端之间的异步消息传递，配置如下：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config){
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry){
        registry.addEndpoint("/chat").withSockJS();
    }
}
```

`config.enableSimpleBroker("/topic")`表示设置消息代理的前缀，即如果消息的前缀是/topic，就会将消息转发给消息代理（borker），再由消息代理将消息广播给当前连接的客户端。

`config.setApplicationDestinationPrefixes("/app");`表示配置了一个或者多个前缀，通过这些前缀过滤出需要被注解方法处理的消息，例如，前缀为app的destination可以通过@MessageMapping注解的方法处理，而其他destination例如（/topic，/queue）将被直接交给broker处理。` registry.addEndpoint("/chat").withSockJS();`则表示定义一个前缀为“/chat”的endpoint，并开启sockjs支持，sockjs可以解决浏览器对WebSocket兼容性问题，客户端通过这里配置的URL来建立WebSocket连接。

定义控制器

```java@Controller
public class index {
    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Message greeting(Message message)throws Exception{
        return message;
    }
}
```

实体类Message(可自行添加属性)

```java
public class Message {
    private String name;
    private String content;
    private String iden;
    private Date date;
    public Message(){

    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    public String getContent() {
        return content;
    }

    public String getIden() {
        return iden;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public void setIden(String iden) {
        this.iden = iden;
    }
}
```

构建聊天界面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>聊天</title>
    <script src="/webjars/jquery/jquery.min.js"></script>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script src="app.js"></script>
</head>
<body>
<div>
    <label for="name">输入用户名</label>
    <input type="text" id="name" placeholder="用户名"><br>
</div>
<div>
    <button id="connect" type="button">连接</button>
    <button id="disconnect" type="button">断开连接</button>
</div>
<div id="chat" style="display: none"></div>
<div id="">
    <label for="name">请输入聊天类容</label>
    <input type="text" id="content" placeholder="聊天内容">
</div>
<button id="send" type="button">发送</button>
<div id="greetings">
    <div id="conversation" style="display: none">群聊进行中。。。</div>
</div>
</body>
</html>
```


创建app.js

```javaScript
var stompClient = null;
function  setConnected(connected) {
    $("#connect").prop("disabled",connected);
    $("#disconnect").prop("disabled",!connected);
    if(connected){
        $("#conversation").show();
        $("#chat").show();
    }
    else {
        $("#conversation").hide();
        $("#chat").hide();
    }
    $("#greetings").html("");
}

function connect() {
    if(!$("#name").val()){
        return ;
    }
    var  socket = new SockJS("/chat");
    stompClient = Stomp.over(socket);
    stompClient.connect({},function (frame) {
        setConnected(true);
        stompClient.subscribe("/topic/greetings",function (greeting) {
            showGreeting(JSON.parse(greeting.body));
        });
    });
}
function disconnect() {
    if(stompClient !==null){
        stompClient.disconnect();
    }
    setConnected(false)

}
function sendName() {
    stompClient.send("/app/hello",{},JSON.stringify({
        'name' : $("#name").val(),
        'content' : $("#content").val(),
        'date' : new Date(),
        'iden' : 'USer'
    }));
}

function  showGreeting(message) {
    $("#greetings").append("<div>"+message.name+":"+message.content+"<br>"+"时间:"+new Date()+"</div>");
}

$(function () {
    $("#connect").click(function () {
        connect();
    });
    $("#disconnect").click(function () {
        disconnect();
    })
    $("#send").click(function () {
        sendName();
    })
})
```

connect方法表示建立一个WebSocket连接，在建立WebSocket连接时，用户必须先输入用户名，然后才能建立连接。后面的就是创建连接，然后创建STOMP实例发起连接请求，在连接成功的回调方法中，首先调用 setConnected(true)方法进行页面设置，然后调用STOMP的subscribe方法订阅服务器发送回来的消息，并将服务器发送的消息展示出来。

接下来就可以测试了，每打开一个`http://localhost:8080/chat.html`链接一个用户就可以参与群聊。
