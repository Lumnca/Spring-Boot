# :maple_leaf:Spring Boot整合NoSQL #

<b id="t"></b>

:arrow_down:[整合Redis](#a1)

:arrow_down:[Redis集群整合Spring Boot](#a2)

:arrow_down:[Session共享](#a3)

<b id="a1"></b>

### :fallen_leaf:整合Redis ###

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

默认情况下，sprign-boot-straer-data-redis 使用的Redis 工具是Lettuce，考虑到有的开发者习惯使用Jedis，因此可以从配置中排除Lettuce 并引入Jedis

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
     public String demo2(){
        Map<String,String> map = new HashMap<>();
        map.put("name","Lumnca");
        map.put("id","2017110329");
        stringRedisTemplate.opsForHash().putAll("me",map);
        return stringRedisTemplate.opsForHash().get("me","id").toString();
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

与基本方法一致，从头和尾添加，从头和尾添加，通过index返回索引值。更多方法自行查看接口。Set类型与List类型一样，这里列出常用的方法API。

**设置过期时间与获取过期时间**

```java
@RestController
public class RedisTest {
    @Autowired
    RedisTemplate redisTemplate;
    @Autowired
    StringRedisTemplate stringRedisTemplate;
    @GetMapping("/set")
    public String set(){
       //获取过期时间
       System.out.println("Live Time:"+stringRedisTemplate.getExpire("name",TimeUnit.MILLISECONDS)+"ms");
       try {
           //设置过期时间
           stringRedisTemplate.expire("name",60000,TimeUnit.MILLISECONDS);
       }
       catch (Exception e){
            System.out.println(e.getMessage());
       }
       return "YES";
    }
}
```

如上getExpire是获取key值的过期时间方法，第二参数为时间单位这里用的ms（毫秒）。expire为键值设定过期时间。第二参数为过期数据，第三参数为时间单位这里用的ms，对于返回值都是long或者int型，参数-1代表时间永久，-2代表过期或者不存在。也可以使用**stringRedisTemplate.hasKey(key)** 来判断键值是否存在。

**增删修改**

```java
 public String demo2(){
        String msg;
        try {
            if(stringRedisTemplate.hasKey("name") ){
                stringRedisTemplate.delete("name");
                msg = "存在,已删除";
            }
            else{
                stringRedisTemplate.opsForValue().set("name","Lumnca",60000,TimeUnit.MILLISECONDS);
                msg = "不存在,已添加";
            }
        }
        catch (Exception e){
            msg = "失败";
        }
        return msg;

    }
```

 stringRedisTemplate.opsForValue().set()可以添加或者修改键值参数。参数分别为key，value，时间，时间单位。小于为永久。
 
 **各个数据类型操作**
 
 ```java
 
 public class RedisService {
	
	private Logger log = LoggerFactory.getLogger(this.getClass());
	
    private final RedisTemplate<String, Object> redisTemplate;
    
    public RedisService(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    // ============================String=============================

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 递增
     *
     * @param key 键
     * @param by  要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key 键
     * @param by  要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================Map=================================

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            log.error("redis error: ", e);
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return 0;
        }
    }
    
    // ===============================sorted set=================================

    /**
     * 向有序集合添加一个成员的
     * 
     * ZADD key score1 member1 [score2 member2] 
     *
     */
    public boolean zadd(String key, Object member, double score, long time) {
        try {
            redisTemplate.opsForZSet().add(key, member, score);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error("redis error: ", e);
            return false;
        }
    }
    
    /**
     * 	ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 
		通过分数返回有序集合指定区间内的成员
     *
     */
    public Set<Object> zRangeByScore(String key, double minScore, double maxScore) {
    	try {
    		return redisTemplate.opsForZSet().rangeByScore(key, minScore, maxScore);
    	} catch (Exception e) {
    		log.error("redis error: ", e);
    		return null;
    	}
    }
    
    /**
     * 	ZSCORE key member 
		返回有序集中，成员的分数值
     *
     */
    public Double zscore(String key, Object member) {
    	try {
    		return redisTemplate.opsForZSet().score(key, member);
    	} catch (Exception e) {
    		log.error("redis error: ", e);
    		return null;
    	}
    }
    
    /**
     * 	ZRANK key member 返回有序集合中指定成员的索引
     *
     */
    public Long zrank(String key, Object member) {
    	try {
    		return redisTemplate.opsForZSet().rank(key, member);
    	} catch (Exception e) {
    		log.error("redis error: ", e);
    		return null;
    	}
    }
    
    /**
     * Zscan 迭代有序集合中的元素（包括元素成员和元素分值）
     *
     */
    public Cursor<ZSetOperations.TypedTuple<Object>> zscan(String key) {
    	try {
    		Cursor<ZSetOperations.TypedTuple<Object>> cursor = redisTemplate.opsForZSet().scan(key, ScanOptions.NONE);
    		return cursor;
    	} catch (Exception e) {
    		log.error("redis error: ", e);
    		return null;
    	}
    }

 ```

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

**集群配置**

首先在Redis中创建节点，由于Redis集群必须至少创建6个节点，接下来创建 redisCluster 文件夹，将前面下载的 Redis 压缩文件复制到 redisCluster 文件
夹中之后编译安装，操作命令如下：

```
mkdir redisCluster
cp-f./redis-4.0.10.tar.gz./ redisCluster/
cd redisCluster 
tar-zxvf redis-4.0.10.tar.gz 
cd redis-4.0.10
make MALLOC=libc
make install
```

安装成功后，将redis-4.0.10/src目录下的redis-trib.rb文件复制到redisCluster目录下，命令如下：

```
cp -f ./redis-4.0.10/src/redis-trib.rb  ./
```

然后在redisCluster目录下创建6个文件夹，分别命名为8001、8002、8003、8004、8005、8006，再将redis-4.0.10目录下的redis.conf文件分别往这6个目录中复制一份，然后对每个目录中的redis.conf文件进行修改，以8001目录下的redis.conf文件为例，主要修改如下配置

```
port 8001
#bind 127.0.0.1
cluster-enabled yes 
cluster-config-file nodes-8001.conf 
protected no 
daemonize yes 
requirepass 123456
masterauth 123456
```

这里的配置在6.1.2小节的单机版安装配置的基础上增加了几条，其中端口修改为8001，cluster-enabled 表示开启集群，cluster-config-file表示集群节点的配置文件，由于每个节点都开启了密码认证，因此又增加了masterauth配置，使得从机可以登录到主机上。按照这里的配置，对8002~8006目录中的redis.conf文件依次进行修改，注意修改时每个文件的port 和cluster-config-file不一样。全部修改完成后，进入redis-4.0.10目录下，分别启动6个Redis实例，相关命令如下：

```
redis-server../8001/redis.conf
redis-server../8002/redis.conf
redis-server../8003/redis.conf
redis-server../8004/redis.conf
redis-server../8005/redis.conf
redis-server../8006/redis.conf
```

当6个Redis实例都启动成功后，回到redisCluster目录下，首先对redis-trib.rb文件进行修改，由于配置了密码登录，而该命令在执行时默认没有密码，因此将登录不上各个Redis实例，此时用vi编辑器打开redis-trib.rb文件，搜索到如下一行：

`=Redis.new（：host =>@info[：host]，：port =>Qinfo[：port]，：timeout =>60）`

修改这一行，添加密码参数：

`=Redis.new（：host =>einfo[：host]，：port =>@info[：port]，：timeout=>60，：password=>"123456"）`

123456就是各个Redis实例的登录密码。

这些配置都完成后，接下来就可以创建Redis集群了。执行如下命令创建Redis集群：

```
./redis-trib.rb create--replicas 1 192.168.248.144：8001 192.168.248.144：8002192.168.248.144：8003192.168.248.144：8004192.168.248.144：8005 192.168.248.144：8006
```
192.168.248.144改为你的IP地址

其中，replicas表示每个主节点的slave数量。在集群的创建过程中会分配主机和从机，每个集群在创建过程中都将分配到一个唯一的id并分配到一段slot。
当集群创建成功后，进入redis-4.0.10目录中，登录任意Redis实例，命令如下：

```
redis-cli -p 8001- a 123456 -c
```

p表示要登录的集群的端口，-a表示要登录的集群的密码，-c则表示以集群的方式登录。登录成功后，通过cluster info命令可以查询集群状态信息（如图6-5所示），通过cluster nodes命令可以查询集群节点信息（如图6-6所示），在集群节点信息中，可以看到每一个节点的id，该节点是slave还是master，如果是slave，那么它的master的id是什么，如果是master，那么每一个master的slot范围是多少，这些信息都会显示出来。

查看节点是否运行：

```
ps -ef  | grep redis
```

![](https://github.com/Lumnca/Spring-Boot/blob/master/img/a45.png)

如上显示表示成功！

当集群创建成功后，随着业务的增长，有可能需要添加主节点，添加主节点需要先构建主节点实例，将redisCluster目录下的8001目录再复制一份，名为8007，根据第3步的集群配置修改8007目录下的redis.conf文件，修改完成后，在redis-4.0.10目录下运行如下命令启动该节点：

`redis-server../8007/redis.conf`

启动成功后，进入redisCluster目录下，执行如下命令将该节点添加到集群中：

`./redis-trib.rb add-node 192.168.248.144：8007 192.168.248.144：8001`

可以看到，新实例已经被添加进集群中，但是由于slot已经被之前的实例分配完了，新添加的实例没有slot，也就意味着新添加的实例没有存储数据的机会，此时需要从另外三个实例中拿出一部分slot分配给新实例，具体操作如下。首先，在redisCluster目录下执行如下命令对slot重新分配：

`./redis-trib.rb reshard 192.168.248.144：8001`

第二个参数表示连接集群中的任意一个实例。

上面添加的节点是主节点，从节点的添加相对要容易一些。添加从节点的步骤如下：首先将redisCluster目录下的8001目录复制一份，命名为8008，然后按照6.1.2小节中第3步的配置修改8008目录下的redis.conf，修改完成后，启动该实例，然后输入如下命令添加从节点：

`./redis-trib.rb add-node--slave--master-id 
e0f2751b46c9ed3ca130e9fc825540386feaafb2192.168.248.144：8008 192.168.248.144：8001`

添加从节点需要指定该从节点的masterid，-master-id后面的参数即表示该从节点master的id，192.168.248.144：8008表示从节点的地址，192.168.248.144：8001则表示集群中任意一个实例的地址。

当从节点添加成功后，登录集群中任意一个Redis实例，通过cluster nodes命令就可以看到从节点的信息。

如果删除的是一个从节点，直接运行如下命令即可删除：`./redis-trib.rb del-node 192.168.248.144：8001122b2098df746afc3a77beddaad85630bf75ab9a`中间的实例地址表示集群中的任意一个实例，最后的参数表示要删除节点的id。但若删除的节点占有slot，则会删除失败，此时按照第5步提到的办法，先将要删除节点的slot全部都分配出去，然后运行如上命令就可以成功删除一个占有slot的节点了。

***

**整合Spring Boot**

首先添加依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
```

配置集群信息

由于集群节点有多个，可以保存在一个集合中，因此这里的配置文件使用YAML格式的，删除resources目录下的application.properties文件，创建application.yml配置文件（YAML配置可以参考2.7节），文件内容如下：

```
spring:
  redis:
    cluster:
      ports:
        - 8001
        - 8002
        - 8003
        - 8004
        - 8005
        - 8006
      host: 47.106.254.86
      poolConfig:
        max-total: 100
        max-idle: 50
        max-wait-millis: -1
        min-idle: 20
```

由于本案例Redis实例的host都是一样的，因此这里配置了一个host，而port配置成了一个集合，这些port将被注入一个集合中。poolConfig则是基本的连接池信息配置。

配置Redis：

```java

@Configuration
@ConfigurationProperties("spring.redis.cluster")
public class RedisConfig {
    List<Integer> ports;
    String host;
    JedisPoolConfig poolConfig;

    public List<Integer> getPorts() {
        return ports;
    }

    public void setPorts(List<Integer> ports) {
        this.ports = ports;
    }

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public JedisPoolConfig getPoolConfig() {
        return poolConfig;
    }

    public void setPoolConfig(JedisPoolConfig poolConfig) {
        this.poolConfig = poolConfig;
    }
    @Bean
    RedisClusterConfiguration redisClusterConfiguration(){
        RedisClusterConfiguration configuration = new RedisClusterConfiguration();
        List<RedisNode>nodes=new ArrayList<>();
        for(Integer port: ports) {
            nodes.add(new RedisNode(host, port));
        }
        configuration.setPassword(RedisPassword.of("你的密码"));
        configuration.setClusterNodes(nodes);
        return configuration;
    }
    @Bean
    JedisConnectionFactory jedisConnectionFactory() {
        JedisConnectionFactory factory = new JedisConnectionFactory(redisClusterConfiguration(), poolConfig);
        return factory;
    }
}
```

通过@ConfigurationProperties注解声明配置文件前缀，配置文件中定义的ports数组、host以及连接池配置信息都将被注入port、host、poolConfig三个属性中。
配置RedisClusterConfiguration实例，设置Redis登录密码以及Redis节点信息。
根据RedisClusterConfiguration实例以及连接池配置信息创建Jedis 连接工厂JedisConnectionFactory。


接下来测试：

```java
@RestController
@ConfigurationProperties(prefix = "spring.redis.cluster")
public class RedisTest {


    @Autowired
    RedisTemplate redisTemplate;
    @Autowired
    StringRedisTemplate stringRedisTemplate;
    @GetMapping("/demo1")
    public String demo1(){
        stringRedisTemplate.opsForValue().set("my","Lumnca");
        return stringRedisTemplate.opsForValue().get("my");
    }
}
```

输出Lumnca表示配置成功！

集群配置以及使用会有很多问题，其主要是集群配置过程中的细节，配置文件等一定要仔细检查注意细节。

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
