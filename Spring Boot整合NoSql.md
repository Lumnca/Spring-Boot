# :maple_leaf:Spring Boot整合NoSQL #

<b id="t"></b>

:arrow_down:[整合Spring Boot](#a1)

:arrow_down:[Redis集群整合Spring Boot](#a2)

:arrow_down:[Session共享](#a3)

<b id="a1"></b>

### :fallen_leaf:整合Spring Boot ###

:arrow_double_up:[返回目录](#t)

Redis的Java客户端有很多，Spring Boot借助于Spring Data Redis为Redis提供了开箱即用的自动化配置，开发者只需要添加配置依赖和Redis连接信息即可。

添加依赖：

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>


        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
    </dependencies>
```

默认情况下，sprign-boot-straer-data-redis 使用的Redis 工具是Lettuce，考虑到有的开发者习惯使用Jedis，因此可以从配置中排除Lettuce 并引入品Jedis

数据库配置。在application文件下添加配置字段：

```
spring.redis.database=0
spring.redis.host=47.106.254.86
spring.redis.port=6379
spring.redis.password=xxxxxxxxxx
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.min-idle=0
```

配置解释：
```
·第1-4行是基本连接信息配置，第5-8行是连接池信息配置。
·第1行配置表示使用的Redis库的编号，Redis中提供了16个database，编号为0-15.
·第2行配置表示Redis实例的地址。
·第3行配置表示Redis端口号，默认是6379。
·第4行配置表示Redis登录密码。
·第5行配置表示Redis连接池的最大连接数。
·第6行配置表示Redis连接池中的最大空闲连接数。
·第7行配置表示连接池的最大阻塞等待时间，默认为-1，表示没有限制。
·第8行配置表示连接池最小空闲连接数。
如果项目使用了Lettuce，则只需将第5~8行配置中的jedis修改为lettuce即可。
```

在Spring Boot中自动配置类提供了RedisAutoConfiguration进行Redis的配置，源码如下：

```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

由这一段源码可以看到，application.properties中配置的信息将被注入RedisProperties中，如果开发者自己没有提供RedisTemplate或者StringRedisTemplate 实例，则Spring Boot默认会提供这两个实例，RedisTemplate 和StringRedisTemplate 实例则提供了Redis的基本操作方法。

接下来是使用：

```java
@RestController
public class index {
    @Autowired
    RedisTemplate redisTemplate;
    @Autowired
    StringRedisTemplate stringRedisTemplate;
    @GetMapping("/index")
    public  String index(){
        ValueOperations<String,String> ops1 = stringRedisTemplate.opsForValue();
        ops1.set("name","lumnca");
        return ops1.get("name");
    }
}
```

StringRedisTemplate 是RedisTemplate 的子类，StringRedis Template中的key 和value 都是字符串，采用的序列化方案是StringRedisSerializer，而Redis Template则可以用来操作对象，Redis Template采用的序列化方案是JdkSerializationRedisSerializer。无论是StringRedis Template还是Redis Template，操作Redis的方法都是一致的。

SuingRecdisTemplate 和RedisTemplate 都是通过opsForValue、opsForZSt或者opsForset等方法首先获取一个操作对象，再使用该操作对象完成数据的读写。
`ops1.set("name","lumnca");`向Rdis中存储一条记录，最后将之读取出来。

点击运行看到返回值说明配置成功！查看数据库同时也多了一条name的记录。如果没有成功检查Redis配置文件是否允许远程连接。

上面只是对字符串进行了测试，还可以对其类型进行测试，使用opsForSet()，opsForList()，opsForHash()进行获取数据，

常用方法列表如下：

**Key类型操作**

|接口	|描述|
|:--:|:---|
|ValueOperations|操作Redis String（或者Value）类型数据|
|ListOperations|操作Redis List类型数据|
|SetOperations|操作Redis Set类型数据|
|ZSetOperations|操作Redis ZSet（或者Sorted Set）类型数据|
|HashOperations|操作Redis Hash类型数据|
|HyperLogLogOperations|操作Redis HyperLogLog类型数据，比如：pfadd，pfcount，...|
|GeoOperations|操作Redis Geospatial类型数据，比如：GEOADD,GEORADIUS,..)|

**Key绑定操作**

|接口	|描述|
|:--:|:---|
|BoundValueOperations|Redis字符串（或值）键绑定操作|
|BoundListOperations|Redis列表键绑定操作|
|BoundSetOperations |Redis Set键绑定操作|
|BoundZSetOperations|Redis ZSet（或Sorted Set）键绑定操作|
|BoundHashOperations|Redis Hash键绑定操作|
|BoundGeoOperations|Redis Geospatial 键绑定操作|

首先来对Hash使用：

```java
    public  String index(){
        HashOperations<String,Object,Object> ops1 = stringRedisTemplate.opsForHash();
        ops1.put("student","name","lumnca");
        ops1.put("student","age","21");
        ops1.put("student","sex","男");
        Map<String,String> map = new HashMap<String, String>();
        map.put("id","XK5115");
        ops1.putAll("student",map);
        return ops1.get("student","id").toString();
    }
```

如上HashOperations需要提供三个泛型值，即key值，哈希值，哈希属性值。使用put方法可以依次传入。也可以通过一个Map类型直接传入一个Hash类型数据。同样的其方法中还有删除，添加，修改，迭代等方法，这里不做解释，可以打开该配置方法的文档查看方法。同样的下面是简单使用List：

```java
    public  String index(){
        ListOperations<String,String> ops1 = stringRedisTemplate.opsForList();
        ops1.leftPush("lists","lumnca");
        ops1.leftPush("lists","kelly");
        ops1.leftPush("lists","marry");
        System.out.println(ops1.leftPop("lists"));
        return ops1.index("lists",1);
    }
```

与基本方法一致，从头和尾添加，从头和尾添加，通过index返回索引值。更多方法自行查看接口。Set类型与List类型一样，这里不做解释。

<b id="a2"></b>

### :fallen_leaf:Redis集群整合Spring Boot ###

:arrow_double_up:[返回目录](#t)

**（1）集群原理**

在Redis集群中，所有的Redis节点彼此互联，节点内部使用二进制协议优化传输速度和带宽。当一个节点挂掉后，集群中超过半数的节点检测失效时才认为该节点已失效。不同于Tomcat集群需要使用反向代理服务器，Redis集群中的任意节点都可以直接和Java客户端连接。Redis集群上的数据分配则是采用哈希槽（HASHSLOT），Redis集群中内置了16384个哈希槽，当有数据需要存储时，Redis会首先使用CRC16算法对key进行计算，将计算获得的结果对16384取余，这样每一个key都会对应一个取值在0~16383之间的哈希槽，Redis则根据这个余数将该条数据存储到对应的Redis节点上，开发者可根据每个Redis实例的性能来调整每个Redis实例上哈希槽的分布范围。

**（2）集群规划**

本案例在同一台服务器上用不同的端口表示不同的Redis服务器（伪分布式集群）。

`主节点：47.106.254.86：8001，47.106.254.86：8002，47.106.254.86：8003。`

`从节点：47.106.254.86：8004，47.106.254.86：8005，47.106.254.86：8006。`

**（3）集群配置**

`Redis 集群管理工具redis-trib.rb依赖Ruby环境，首先需要安装Ruby环境，由于CentOS 7yum库中默认的Ruby版本较低，因此建议采用如下步骤进行安装。
首先安装RVM，RVM是一个命令行工具，可以提供一个便捷的多版本Ruby环境的管理和切换，安装命令如下：`

```
sudo yum install ruby  
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
```

然后就是列出安装表,安装一个比较稳定的版本：

```
rvm list known
rvm install 2.6.3
```

最后安装Redis依赖：

```
gem install redis
```

文档丢失？

***


<b id="a3"></b>

### :fallen_leaf:Session共享 ###

:arrow_double_up:[返回目录](#t)

正常情况下，HtpSession是通过Servlet容器创建并进行管理的，创建成功之后都是保存在内存中。如果开发者需要对项目进行横向扩展搭建集群，那么可以利用一些硬件或者软件工具来做负载均衡，此时，来自同一用户的HTTP请求就有可能被分发到不同的实例上去，如何保证各个实例之间Session的同步就成为一个必须解决的问题。Spring Boot提供了自动化的Session共享配置，它结合Redis可以非常方便地解决这个问题。使用Redis解决Session共享问题的原理非常简单，就是把原本存储在不同服务器上的Session拿出来放在一个独立的服务器上.

![](https://www.runoob.com/wp-content/uploads/2018/08/1535725078-1224-20160201162405944-676557632.jpg)

当一个请求到达Nginx服务器后，首先进行请求分发，假设请求被real serverl处理了，real server1处理请求时，无论是存储Session还是读取Session，都去操作Session服务器而不是操作自身内存中的Session，其他real server在处理请求时也是如此，这样就可以实现Session共享了。

**Session共享配置**

添加依赖，最后面加的是项目打包的的组件
```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>


        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

在application配置文件中添加Redis的配置：

```xml
spring.http.encoding.force=true
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
server.tomcat.uri-encoding=UTF-8
server.port=8080
spring.redis.database=0
spring.redis.host=47.106.254.86
spring.redis.port=6379
spring.redis.password=xxxx
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.min-idle=0
```

创建一个控制器来完成端口显示：

```java
@RestController
public class index {
    @Value("${server.port}")
    String port;
    @GetMapping("/save")
    public  String saveName(String name, HttpSession session){
        session.setAttribute("name",name);
        return "响应你的端口是：" +port;
    }
    @GetMapping("/get")
    public String getName(HttpSession session){
        return "响应你的端口是：" +port+":"+ session.getAttribute("name").toString();
    }
}
```

这里注入了项目启动文件配置的server.port参数由于我们使用的是本地编译，所以这个端口号会不变，我们这里使用服务端创建，将项目打包为jar上传到你的CentOS上，在与包名相应的目录下执行java程序命令：

```
nohup java -jar test-1.0-SNAPSHOT.jar --server.port=1551 &
nohup java -jar test-1.0-SNAPSHOT.jar --server.port=1552 &
```

nohup 表示不挂断执行，&表示程序在后台运行。--server.port表示设置端口，一个为1551，一个为1552 。由于运行还需要java环境配置，请自行配置好服务运行环境。对于java程序在linux上运行，将在后面介绍。查看进程是否运行：

```shell
netstat -lnp|grep 1551
netstat -lnp|grep 1552  端口号
```

结束进程可以使用`kill -9 +pid`结束进程，如下：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a15.png)

其中第二个是正常现象。接下就是我们的服务器配置了。

**Nginx负载均衡**

我们使用Nginx进行负载均衡。首先先安装Nginx，安装步骤如下：

```
wget https://nginx.org/download/nginx-1.14.0.tar.gz 
tar-zxvf nginx-1.14.0.tar.gz
```

然后进入解压目录中执行编译安装，代码如下：
```
cd nginx-1.14.0
./configure
make
make instal1
```
安装成功后，找到Nginx安装目录，执行sbin目录下的nginx文件启动nginx，命令如下：

`/usr/1ocal/nginx/sbin/nginx Nginx`

启动成功后，默认端口是80，可以在物理机直接访问，如图6-19所示。如果端口占用，在配置文件中修改端口：

`vim  /usr/local/nginx/conf/nginx.conf`

修改内容如下：

```
 upstream sang.com{
        server 47.106.254.86:1551 weight=1;    #---服务器1
        server 47.106.254.86:1552 weight=1;    #---服务器2
       } 
    server {
        listen      100;            #--监听端口
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://sang.com;  #--请求转发到上面的服务群sang.com
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

主要修改如上代码，配置完成后或者修改后都需要重新启动服务：

```
/usr/local/nginx/sbin/nginx -s reload
```

接下来就可以测试运行了，在服务器上输入：`http://47.106.254.86:100/save?name=lumnca`得到如下显示：

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a16.png)

再获取：`http://47.106.254.86:100/get` 如下显示

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a17.png)

注意的是这里使用的端口是Nginx的端口，不是我们配置的1551和1552，由于是负载均衡，如果响应端口一样，再次刷新即可、
