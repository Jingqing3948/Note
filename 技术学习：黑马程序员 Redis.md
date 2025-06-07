---
title: 技术学习：黑马程序员 Redis
date: 2025-05-21
tags: 
- Java
- Redis
categories:
- 算道求索（课外IT技能学习）
- Lecture
description: 黑马程序员Redis入门课程学习笔记

---

## 前言

学习自：[【黑马程序员Redis入门到实战教程，深度透析redis底层原理+redis分布式锁+企业解决方案+黑马点评实战项目】 ](https://www.bilibili.com/video/BV1cr4y1671t/?p=2&share_source=copy_web&vd_source=dcdc734e318da0cd82bcccb180b12b40)

### SQL vs. NoSQL

- S：结构化，比如在 MySQL 我们需要定义一个表，第一个属性是 id 第二个是 name 第三个是 age…… 定义好表之后所有新插入的数据都必须遵循这个结构，各个表还可能有外键等关联，所以总的来说不能随意修改表的属性。而 NoSQL 结构要求没那么严格，比如下面这种 Redis 定义方式也可以，就是没啥章法：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505211052874.png" style="zoom:67%;" />

​	除了键值类型，还有文档类型，列类型，图类型等 NoSQL。

- R：关联的，SQL 数据库的各个属性之间有关联而 NoSQL 未必。

- SQL 语句查询：关系型数据库都支持，而非关系型不支持。
- ACID 事务：NoSQL 不支持。
- 存储位置：SQL 在硬盘，NoSQL 在内存。
- SQL 对安全一致性要求较高，NoSQL 对性能要求较高。

### Redis

Redis Remote Dictionary Server 是一种基于内存的**键值对**数据库，被创造的原因就是 MySQL 性能太差。里面没有表什么的结构，就是存储一对一对的键值，所以是 NoSQL 数据库。值可以是复杂的数据形式，比如键是 1001，值是 {name: "xxx", age:"19"}…… 这样的 json 格式。

Redis 是**单线程**的，但是效率仍然比 MySQL 高很多，因为内存存取速度远大于硬盘，C 语言编写，IO 多路复用等特点。

支持**数据持久化**，定期将数据从内存持久化到磁盘存储。

支持主从集群，分片集群。

支持多语言客户端。

## Redis 基础

安装部分就跳过不做过多赘述了。

Redis 默认启动方式是前台启动，会阻塞整个会话窗口。可以通过修改配置文件的方式改为后台启动。或者在 system 文件夹内新建一个系统服务 .service 文件，实现开机自启动。

配置完成后，终端客户端连接：

```cmd
$ systemctl start redis # 启动 redis 后台服务
$ redis-cli -h 127.0.0.1 [-p port] [-a password] # 客户端终端方式连接 redis-cli
> AUTH password # 输入用户名和密码进行登录，也可以在上面 -a 那一步登录 不过这样不安全
> ping # 如果成功登录了就会收到：PONG 否则会受到权限不够的提示

> SELECT 1 # 选择一个库，0-15
> get key # 获取某个键对应的值
> set key value # 添加一个键值对
```

图形化客户端连接：其实不是官方开发的是 Github 上有人发布的开源图形化客户端。

### Redis 通用命令

#### Keys

`Keys pattern` 查询符合某种模式的键，比如 `keys a*` 是查询 a 打头的所有键值。

这个查询并不高效，而且会阻塞其他请求，所以在生产环境中尽量不要使用，特别是不要在主节点上使用。

#### Del

`Del key1 key2 ...` 删除键值对。返回值是成功删除的键数量。

#### MSet

`msel k1 v1 k2 v2 ...` 批量添加键值对。

#### Exists

`exists key` 查询指定键是否存在。

#### Expire

给键值对设定一个有效期，到期自动删除。

`expire key seconds`

#### TTL

`ttl key` 查询这个键还有多久过期。-1表示永久有效，-2表示已过期。

### Redis 数据结构

key 一般是 string 类型，而 value 类型多种多样。

- String：字符串。
- hash
- list
- set
- sortedset

以上为基本数据类型。

- geo
- bitmap
- hyperlog

以上为特殊数据类型。

在 redis 里面输入 `help @想要查询的 command 名称` 就可以进行查询。

#### String

最简单的存储方式，字符串类型。具体可以分为三类：String，int，float。

是的 int 和 float 在 redis 里被视作属于 string。

这些底层存储方式都是字节数组，只不过数字会被直接转化为对应的二进制，这样能存储的范围更大。

| 字符串常用命令         | 解释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `Set key value`        | 添加键值对，如果存在则覆盖                                   |
| `Get key`              | 获取这个键的值                                               |
| `MSet k1 v1 k2 v2`     | 批量添加                                                     |
| `MGet k1 k2 k3`        | 批量获取                                                     |
| `INCR key`             | 让一个整形 Key 对应的值自增1                                 |
| `INCRBY key step`      | 指定步长的自增，如步长是2。可以指定为负数来取代 DECR 的功能。 |
| `INCRBYFLOAT key step` | 浮点数据的指定长度的自增                                     |
| `SETNX k v`            | 添加一个 String 类型的键值对，如果存在则不执行返回0. 其实 `set key value nx` 也能实现同样的效果 |
| `SETEX k v seconds`    | 添加一个 String 类型的键值对并指定有效期。`set key value ex` 也能实现同样的效果 |

##### Key 的层级格式

Redis 没有 MySQL 的 Table，如何区分不同的 key 意义呢，比如一个商品 id 是1，一个顾客 id 也是1.

Redis 允许键值进行拼接，如：`项目名:业务名:类型:id` 这样。

```cmd
127.0.0.1:6379> set project:user:1 '{"id":1, "name":"Jack", "age": 21}'
OK
127.0.0.1:6379> set project:user:2 '{"id":2, "name":"Rose", "age": 18}'
OK
127.0.0.1:6379> set project:product:1 '{"id":1, "name":"小米11", "price": 4999}'
127.0.0.1:6379> set project:product:2 '{"id":2, "name":"荣耀6", "price": 2999}'
OK

```

查看图形化客户端可以发现：Key 已经形成层级结构。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212230806.png)

#### Hash

无序字典，类似 java HashMap.

之前我们学过，String 处理对象的方式是变成 json 字符串，但是这样缺点是修改比较难操作。而 Hash 数据结构是可以真正将所有字段都分开存储方便操作。

| Hash 常用命令            | 解释                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `HSET key field value`   | 添加或者修改 hash 类型 key 的 field 的值                     |
| `HGET key field`         | 获取一个 hash 类型 key 的 field 的值                         |
| `HMSET key f1 v1 ...`    | 批量添加多个 hash 类型 key 的 field 的值（Redis 4.0+ 推荐用 HSET） |
| `HMGET key f1 f2 ...`    | 批量获取多个 hash 类型 key 的 field 的值                     |
| `HGETALL key`            | 获取一个 hash 类型的 key 中的所有的 field 和 value           |
| `HKEYS key`              | 获取一个 hash 类型的 key 中的所有 field                      |
| `HVALS key`              | 获取一个 hash 类型的 key 中的所有 value                      |
| `HINCRBY key field step` | 让一个 hash 类型 key 的字段值自增，并指定步长                |
| `HSETNX key field value` | 添加一个 hash 类型的 key 的 field 值，前提是这个 field 不存在，否则不执行 |

添加完字段之后，图形化界面中可视化结构：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212241594.png)

#### List

大致可以看做 java 的双向链表结构。有序，插入删除速度快，查询速度一般。

常常用于存储有顺序的数据，比如排队，朋友圈点赞列表等。

| List 常用命令                 | 解释                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `LPUSH key element ...`       | 向列表左侧插入一个或多个元素                                 |
| `LPOP key`                    | 移除并返回列表左侧的第一个元素，没有则返回 nil               |
| `RPUSH key element ...`       | 向列表右侧插入一个或多个元素                                 |
| `RPOP key`                    | 移除并返回列表右侧的第一个元素                               |
| `LRANGE key start end`        | 返回一段脚标范围内的所有元素（脚标从左0开始向右逐渐增加）    |
| `BLPOP key [key ...] timeout` | 与 LPOP 类似，但在没有元素时会等待指定时间（阻塞），不是直接返回 nil |
| `BRPOP key [key ...] timeout` | 与 RPOP 类似，但在没有元素时会等待指定时间（阻塞），不是直接返回 nil |

用 List 模拟栈：只用 LPUSH 和 LPOP 或者 RPUSH RPOP 同向。

用 List 模拟队列：LPUSH+RPOP 或 RPUSH+LPOP 不同向。

用 List 模拟阻塞队列：首先需要入口和出口在不同边；其次出队使用 BLPOP 或者 BRPOP。

*阻塞队列：当队列为空的时候，取元素的线程会等待一段时间直到队列非空。应用：线程池，待处理的任务先到等待队列，等待有空闲的线程时再开始被执行。这样不用一直创建新的线程，只利用现有的最大线程数，节约资源和响应速度。*

#### Set

类似 Java 中的 HashSet，无序，不重复，查找快，支持并集差集交集等运算。

| Set 常用命令           | 解释                          |
| ---------------------- | ----------------------------- |
| `SADD key member ...`  | 向 set 中添加一个或多个元素   |
| `SREM key member ...`  | 移除 set 中的指定元素         |
| `SCARD key`            | 返回 set 中元素的个数         |
| `SISMEMBER key member` | 判断一个元素是否存在于 set 中 |
| `SMEMBERS key`         | 获取 set 中的所有元素         |
| `SINTER key1 key2 ...` | 求 key1 与 key2 的交集        |
| `SDIFF key1 key2 ...`  | 求 key1 与 key2 的差集        |
| `SUNION key1 key2 ...` | 求 key1 和 key2 的并集        |

#### SortedSet

有点类似 java 的 treeset 但是底层逻辑差得很远。

*java 的 treeset：一种有序集合，底层逻辑是红黑树。红黑树类似 AVL 树，这两种树在排序的同时也会保证树的左右子节点深度，数量差别不大以免树结构的复杂度变得和线性结构差不多，AVL 是规定左右子节点深度差不能超过1，而红黑树要求松一些，规定从根到叶子的最长路径不可能达到最短路径的2倍长，所以旋转操作相对 AVL 少一些，AVL 更适用于查找操作远多于插入删除操作数量的情况。*

SortedSet 每个元素都带有一个 score 属性可以排序，通过跳表+hash表排序。

*跳表结构大概如下，来加速单链表的查找效率。可以再多建几级索引。图源：[跳表的原理与实现 [图解]_跳表实现-CSDN博客](https://blog.csdn.net/Appleeatingboy/article/details/119948340)*

![img](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212313909.png)

SortedSet 由于其排序效率高，常常被用于实现如排行榜这样的功能。

| Sorted Set 常用命令            | 解释                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `ZADD key score member`        | 添加一个或多个元素到 sorted set，如果已经存在则更新其 score 值 |
| `ZREM key member`              | 删除 sorted set 中的一个指定元素                             |
| `ZSCORE key member`            | 获取 sorted set 中指定元素的 score 值                        |
| `ZRANK key member`             | 获取 sorted set 中指定元素的排名（从 0 开始）                |
| `ZCARD key`                    | 获取 sorted set 中的元素个数                                 |
| `ZCOUNT key min max`           | 统计 score 值在指定范围内的元素个数                          |
| `ZINCRBY key increment member` | 让 sorted set 中指定元素自增，步长为指定的 increment 值      |
| `ZRANGE key start stop`        | 按 score 排序，获取指定排名范围内的元素 *如前十名*           |
| `ZRANGEBYSCORE key min max`    | 按 score 排序，获取指定 score 范围内的元素 *如60分~70分*     |
| `ZDIFF / ZINTER / ZUNION`      | 分别用于求差集、交集、并集                                   |

以上都是升序，降序就在 Z 后面加 REV。

### Redis 的 Java 客户端

主要有一下三种：

| 客户端   | 简介                                                        | 特点                                                         |
| -------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| Jedis    | 以 Redis 命令作为方法名称，学习成本低，简单实用。           | 实例非线程安全，多线程环境下需基于连接池使用。               |
| Lettuce  | 基于 Netty 实现，支持同步、异步和响应式编程方式，线程安全。 | 支持 Redis 的哨兵模式、集群模式和管道模式。                  |
| Redisson | 基于 Redis 实现的分布式、可伸缩 Java 数据结构集合。         | 提供如 `Map`、`Queue`、`Lock`、`Semaphore`、`AtomicLong` 等强大功能，适合分布式场景。 |

spring data redis 整合了前两种。

#### Jedis

使用方式很简单：引入依赖，建立连接，使用，释放资源。

##### 引入依赖

在 pom.xml 文件中粘贴：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505221740557.png)

```xml
<dependencies>
    <!--jedis-->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.7.0</version>
    </dependency>
    <!--单元测试-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

第一部分依赖是必须引入的，第二部分是测试用的。

执行代码：

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import redis.clients.jedis.Jedis;

import java.util.Map;

public class JedisTest {
    private Jedis jedis;

    @BeforeEach
    void SetUp(){
        jedis = new Jedis("192.168.73.129",6379);
        jedis.auth("369789");
        jedis.select(0);
    }

    @Test
    void testString() {
        // 存入数据
        String result = jedis.set("name", "虎哥");
        System.out.println("result = " + result);
        // 获取数据
        String name = jedis.get("name");
        System.out.println("name = " + name);
    }

    @Test
    void testHash() {
        // 插入hash数据
        jedis.hset("user:1", "name", "Jack");
        jedis.hset("user:1", "age", "21");

        // 获取
        Map<String, String> map = jedis.hgetAll("user:1");
        System.out.println(map);
    }

    @AfterEach
    void tearDown() {
        if (jedis != null) {
            jedis.close();
        }
    }
}
```

##### 连接池

Jedis 线程不安全，而且频繁创建销毁 Jedis 线程性能损耗比较大。

*线程不安全的大概原因：每次 set 的时候都要调用 connect 方法。在 connect 方法里，一个实例的多个线程都共用一个 socket, inputstream, outputstream。所以如果线程1正在读写，线程2把 socket重新初始化了，那么1的连接就断了。或者两个线程同时读写，inputstream outputstream 共用可能就会导致输入或读取数据错误。[使用jedis面临的非线程安全问题-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1678172)*

![img](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505222302284.png)

> 图源：[java客户端：Jedis和Jedis连接池的基本使用和配置 - 知乎](https://zhuanlan.zhihu.com/p/84481313)

创建连接池代码：

```java
public class JedisConnectionFacotry {

     private static final JedisPool jedisPool;

     static {
         //配置连接池
         JedisPoolConfig poolConfig = new JedisPoolConfig();
         poolConfig.setMaxTotal(8);// 最大连接数
         poolConfig.setMaxIdle(8);// 最大空闲连接数
         poolConfig.setMinIdle(0);// 最小空闲连接数，防止空闲连接被释放
         poolConfig.setMaxWaitMillis(1000);
         //创建连接池对象
         jedisPool = new JedisPool(poolConfig,
                 "192.168.150.101",6379,1000,"123321");
     }

     public static Jedis getJedis(){
          return jedisPool.getResource();
     }
}
```

然后创建 Jedis 的时候改成使用连接池的获取代码：

```java
jedis = JedisConnectionFactory.getJedis();
```

`jedis.close()` 函数底层如果判断有连接池的时候，就不会 close 关闭 Jedis 资源了，而是归还资源，所以不需要改。

#### SpringDataRedis

SpringData 是 Spring 的数据操作模块，其中的 Redis 集成模块叫做 SpringDataRedis。整合了多个 Redis 客户端，并提供了统一的访问 API。

支持 JDK json 字符串 对象等的序列化和反序列化，因为现在我们直接操作 Jedis 传入的都是 byte[] 数组，如果想传入 json 或者对象需要自己手动调整序列化。

| API                           | 返回值类型        | 说明                      |
| ----------------------------- | ----------------- | ------------------------- |
| `redisTemplate.opsForValue()` | `ValueOperations` | 操作 `String` 类型数据    |
| `redisTemplate.opsForHash()`  | `HashOperations`  | 操作 `Hash` 类型数据      |
| `redisTemplate.opsForList()`  | `ListOperations`  | 操作 `List` 类型数据      |
| `redisTemplate.opsForSet()`   | `SetOperations`   | 操作 `Set` 类型数据       |
| `redisTemplate.opsForZSet()`  | `ZSetOperations`  | 操作 `SortedSet` 类型数据 |
| `redisTemplate`               | -                 | 通用的命令接口            |

使用起来也非常简单。首先在 IDEA 里面创建项目的时候就可以创建 spring initilize 的项目，然后导入相关依赖（redis，连接池等），配置连接，直接使用。

依赖：

```xml
<!-- pom.xml -->
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--common-pool-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!--Jackson依赖-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

配置：

```yaml
# application.yaml
spring:
  data:
    redis:
      host: 127.0.0.1
      port: 6379
      password: 123456
      database: 0
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: 100ms

```

SpringDataRedis 默认引入了 Lettuce 的依赖，如果想在上面的配置文件中使用 jedis 也可以只不过就是需要在 pom.xml 里面也引入 jedis 的依赖。

代码测试：

```java
@SpringBootTest
class RedisDemoApplicationTests {

    @Autowired
    RedisTemplate redisTemplate;

    @Test
    void testString() {
        redisTemplate.opsForValue().set("springtest","springtest");
        Object value = redisTemplate.opsForValue().get("springtest");
        System.out.println("value "+value);
    }

}
```

在 Redis 数据库中可见真正存入的数据形式其实是：\xAC\xED\x00\x05t\x00\x0Aspringtest，前面这一串可能就是 Spring Redis 的自动序列化用于记录对象格式的，反正 System.out.print 输出的是没有乱码的。

*老师的案例里面，老师是写入的 name xxx 键值对，然后再去终端执行 redis 命令 `get name` 发现获取到的值并没有更新，也是一样的道理，因为老师实际存入的键也是类似 `\乱码 \乱码 name` 这样的形式，和 `get name` 访问的键不一样.*

这种方式的主要两个问题：可读性差；内存占用大。Java 的 String 也会被视作是一种对象，而非单纯的 String。

##### RedisTemplate 自定义和序列化

想要“写入什么值就存入什么值”，就必须去改写 RedisTemplate 的序列化方法（其实底层调用的是 `JDKSerializationRedisSerializer` 的序列化方法）。

对于字符串，或者键，可以将底层序列化方法改成：`StringRedisSerializer`，这个是专门处理字符串的序列化方法。

对于对象，可以用 `GenericJackson2JsonRedisSerializer` 转 JSON 字符串的序列化方法。

具体修改：新建一个 `RedisConfig` 文件：

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){ // 这里可见我们直接让 key 默认就是 String 类型了
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具，用于对象处理
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
```

这个代码是所有 key 都用字符串序列化方法，而所有的 value 都用 json 对象处理方法。

使用的时候，唯一区别就是声明要使用的 RedisTemplate 类是 <String, Object> 的，也就是我们刚刚生成的。

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;
```

重新运行后发现数据库里面的键值都没有乱码了（值的话，可能 String Object 转换成 JSON 也是 String 的形式）。

尝试传入对象也非常顺利，假设新建了一个 User 对象，数据库中成功传入后的 json value 如下（第一句话就是帮助反序列化还原用的）：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505231149205.png)

并且 java 输出里面自动反序列化成对象的输出形式：

```shell
value User(name=jingqing, age=22)
```

但是可以看出，上面 class 那一行占用存储空间还是挺大的，比这个对象的其余部分内容都长。

不要这部分，就不能实现自动的反序列化了。要吧，又要占用内存空间。

##### StringRedisTemplate 手动序列化

另一种实现方式是 `StringRedisTemplate` ，就是统一使用 String 序列化器。对于对象的存储，需要手动序列化或者反序列化。

实现起来也不是特别难，就是通过一个 `ObjectMapper` 将 String 映射成一个 json String 后传入数据库，传出的时候再通过映射类映射回来。也不需要自定义 RedisTemplate 的序列化方案了。

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

@Test
void testString() {
    // 写入一条String数据
    stringRedisTemplate.opsForValue().set("verify:phone:13600527634", "124143");
    // 获取string数据
    Object name = stringRedisTemplate.opsForValue().get("name");
    System.out.println("name = " + name);
}

private static final ObjectMapper mapper = new ObjectMapper();

@Test
void testSaveUser() throws JsonProcessingException {
    // 创建对象
    User user = new User("虎哥", 21);
    // 手动序列化
    String json = mapper.writeValueAsString(user);
    // 写入数据
    stringRedisTemplate.opsForValue().set("user:200", json);

    // 获取数据
    String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
    // 手动反序列化
    User user1 = mapper.readValue(jsonUser, User.class);
    System.out.println("user1 = " + user1);
}
```

我们还可以进一步将序列化后写入和读出后反序列化封装成工具类，用工具类直接写入和读出，这样代码复杂度上也降低了。

