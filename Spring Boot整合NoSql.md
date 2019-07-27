# :maple_leaf:Spring Boot整合NoSQL #

<b id="t"></b>

:arrow_down:[整合Spring Boot](#a1)

:arrow_down:[Redis集群整合Spring Boot](#a1)

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


