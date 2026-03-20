# Redis

简介：一个基于**内存**的 key-value 结构数据库

- 基于内存存储，读写性能高
- 适合存储热点数据（热点商品、资讯、新闻）

中文网：https://www.redis.net.cn/

<img src="C:\Users\姚\AppData\Roaming\Typora\typora-user-images\image-20260316195915275.png" alt="image-20260316195915275" style="zoom: 50%;" />

`redis.windows.config`是配置文件

`redis-cli.exe`是Redis客户端

`redis-server.exe`是Redis服务端

`redis`服务端使用：

在`redis`目录下进入`cmd` 

输入 `redis-server.exe` `redis.windows.config`默认端口号6379

Ctrl+C退出

`redis`客户端使用：

在`redis`目录下进入`cmd`

输入`redis-cli.exe`

输入`exit`退出服务

`redis-cli.exe -h localhost -p 6379` `-h`指的是主机号，`-p`指的是端口号 `-a`指密码

默认是没有密码，进入`redis.windows.config`文件Ctrl+C输入pass查找，然后设置

<img src="C:\Users\姚\AppData\Roaming\Typora\typora-user-images\image-20260316200815336.png" alt="image-20260316200815336" style="zoom:50%;" />

图形化界面软件：`Another-redis`



## 在`JAVA`中操作`Redis`

### 	Redis的java客户端

- `Jedis`
- `Lettuce`
- `Spring Data Redis`

### `Spring Data Redis` 的使用方式

- 导入`Spring Data Redis 的maven`坐标

<img src="C:\Users\姚\AppData\Roaming\Typora\typora-user-images\image-20260316204329460.png" alt="image-20260316204329460" style="zoom:50%;" />

`spring-boot-starter-data-redis`

- 配置Redis数据源

<img src="C:\Users\姚\AppData\Roaming\Typora\typora-user-images\image-20260316205202325.png" alt="image-20260316205202325" style="zoom: 50%;" />

在spring的目录下

<img src="C:\Users\姚\AppData\Roaming\Typora\typora-user-images\image-20260316205138442.png" alt="image-20260316205138442" style="zoom:50%;" />

如果测试连接失败，尝试更改redis驱动版本，调低

- 编写配置类，创建`RedisTemplate`(Redis模板)对象

```java
@Configuration
public class RedisConfiguration {
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate redisTemplate = new RedisTemplate();
        //设置Redis连接工厂对象
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //设置redis key的序列化器	让redis的图形化软件的key值不乱码显示
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}
```

- 通过`RedisTemplate`对象操作`redis`

```
//五种数据对象的操作对象
        ValueOperations valueOperations = redisTemplate.opsForValue();//String类型
        HashOperations hashOperations = redisTemplate.opsForHash();//Hash类型
        ListOperations listOperations = redisTemplate.opsForList();//列表类型
        SetOperations setOperations = redisTemplate.opsForSet();//集合类型
        ZSetOperations zSetOperations = redisTemplate.opsForZSet();//有序集合
```

`String`类型

```java
public void testString(){
        //set get setex setnx     (key，value)
        redisTemplate.opsForValue().set("city","北京");
        String city = (String) redisTemplate.opsForValue().get("city");
        System.out.println( city);
        redisTemplate.opsForValue().set("code","1234",3, TimeUnit.MINUTES);//设置数据有效期
        redisTemplate.opsForValue().setIfAbsent("lock","1");//key不存在才插入数据
        redisTemplate.opsForValue().setIfAbsent("lock","2");//redis图形化界面可能出现乱码，但是没有问题
    }
```

Hash类型

```java
public void testHash(){
        //hset hget hdel hkeys hvals hgetall
        HashOperations hashOperations = redisTemplate.opsForHash();

        redisTemplate.opsForHash().put("100","name","Tom");
        redisTemplate.opsForHash().put("100","age","18");
        String name = (String) redisTemplate.opsForHash().get("100", "name");
        System.out.println( name);

        Set keys = hashOperations.keys("100");
        System.out.println( keys);

        List values = hashOperations.values("100");
        System.out.println( values);

        hashOperations.delete("100","age");
    }
```

列表数据

```java
public void testList(){
        //lpush lpop lrange lindex lset lrem
        ListOperations listOperations = redisTemplate.opsForList();

        listOperations.leftPushAll("mylist","a","b","c");
        listOperations.leftPush("mylist","d");//左侧加入元素

        List mylist = listOperations.range("mylist", 0, -1);//查询所有元素
        System.out.println( mylist);

        listOperations.rightPop("mylist");//右侧移除元素

        Long size = listOperations.size("mylist");
        System.out.println(size);
    }
```

集合数据类型

```java
public void testSet(){
        //sadd添加 smembers查看所有成员 scard查看元素个数 sinter交集 sunion并集 srem删除某元素
        SetOperations setOperations = redisTemplate.opsForSet();

        setOperations.add("Set1","a","b","c","e");
        setOperations.add("Set2","a","b","x","y");

        Set set1 = setOperations.members("Set1");
        System.out.println( set1);

        Long size = setOperations.size("Set2");
        System.out.println( size);

        Set intersect = setOperations.intersect("Set1", "Set2");
        System.out.println( intersect);

        Set union = setOperations.union("Set1", "Set2");
        System.out.println( union);

        setOperations.remove("Set1","a","b");
    }
```

有序集合数据

```
public void testZSet(){
        //zadd添加 zrange获取指定范围元素 zincrby设置某个元素分数 zrem删除某元素
        ZSetOperations zSetOperations = redisTemplate.opsForZSet();
        zSetOperations.add("zset1","a",1);
        zSetOperations.add("zset1","b",2);
        zSetOperations.add("zset1","c",3);

        Set zset1 = zSetOperations.range("zset1", 0, -1);
        System.out.println( zset1);

        zSetOperations.incrementScore("zset1","a",4);

        zSetOperations.remove("zset1","b");
    }
```

通用命令操作

```
public void testCommon(){
        //keys查询key exists查询key是否存在 type查key的类型 del删除key
        Set keys = redisTemplate.keys("*");
        System.out.println( keys);

        Boolean city = redisTemplate.hasKey("city");
        System.out.println( city);

        System.out.println( redisTemplate.type("city"));

        redisTemplate.delete("city");
    }
```

















