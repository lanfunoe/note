# Redis  Linux 混合学习

[学习资源](https://www.bilibili.com/video/BV1Rv41177Af)

[学习资源](https://www.runoob.com/redis/redis-tutorial.html)

本文是对上两者的综合（抄写

> 主线为学习Redis ，学习Linux为辅

## 安装

> 以下在linux下进行

 下载

 ```
wget https://download.redis.io/redis-stable.tar.gz
 ```

解压

需要提前准备c语言编译环境

[引用](https://www.runoob.com/linux/linux-comm-tar.html)

```
eg
压缩 a.c文件为test.tar.gz
# tar -czvf test.tar.gz a.c 
解压
# tar -xzvf test.tar.gz 
```

进入文件夹

编译

`make`

安装

`make install`

后台启动

1. 修改配置文件使其支持后台启动

   复制到配置文件到etc下    

   cp redis.conf /etc/redis.conf

   修改 daemonize 为yes

```
linux命令：/（内容）   #linux搜索命令
linux命令：wq         #保存退出
```

2. 启动

cd /usr/local/bin

redis-server /etc/redis.conf



3. 查看redis情况

 ```
linux命令：ps [options] [--help]           #Linux ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器。

 eg：查找指定进程格式：
 ps -ef | grep 进程关键字
 ```

4. 用客户端访问：redis-cli

2.2.5.5.多个端口可以：redis-cli -p[端口号]

5. 测试验证

   ping

6. Redis 关闭

   单实例关闭：redis-cli shut,也可以进入终端后再关闭

   多实例关闭，指定端口关闭：redis-cli -p[端口号]  shutdown

## 常识

Redis是单线程+多路IO复用技术


## String

### 基本常识

最大长度 512M

底层：动态字符串

字符串长度小于1M时，扩容加倍当前空间

大于1M时，扩容增加1M

String类型是二进制安全的。意味着Redis的string可以包含任何数据。

   ### key 操作

```
keys *              #查询所有key
set [key] [value]
exists [key]        #存在返回value，否则返回0
type[key]           #类型
del[key]            #删除
unlink [key]        #根据value选择非阻塞删除， 建议区分，异步与非阻塞
expire [key]        #设置key有效期 -1永不过期， -2已经过期
ttl [key]			#查看还有多久过期
select              #选择库 ,16
dbsize				#当前数据库key数量
get <key> <value>
append <key> <value>  #返回总长度
strlen <key>
setnx <key> <value>   #不存在才可以设置
incr <key>  
decr <key>
incrby <key> <value>
decrby <key> <value>
flushdb             #清空库
mget				#同时操作多个
mset
msetnx
getrange <key><start><end>  #substring() 
setrange <key><start><value>
setex <key> <life><value>
getset <key><value> #新值换旧值
```

## List

### 基本常识

单键多值

底层：双向链表

数据结构：quickList

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量比较多的时候才会改成quicklist。

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![img](file:///C:\Users\xtdx\AppData\Local\Temp\ksohtml\wps490.tmp.jpg) 

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

### 操作

```
lpush/rpush  <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
lpop/rpop  <key>从左边/右边吐出一个值。值在键在，值光键亡。

rpoplpush  <key1><key2>从<key1>列表右边吐出一个值，插到<key2>列表左边。

lrange <key><start><stop>
按照索引下标获得元素(从左到右)
lrange mylist 0 -1   0左边第一个，-1右边第一个，（0-1表示获取所有）
lindex <key><index>按照索引下标获得元素(从左到右)
llen <key>获得列表长度 

linsert <key>  before <value><newvalue>在<value>的后面插入<newvalue>插入值
lrem <key><n><value>从左边删除n个value(从左到右)
lset<key><index><value>将列表key下标为index的值替换成value
```

## Set

### 基本常识

可以自动排重，

包含contains方法

底层：value为null的hash表

增删查复杂度为O（1）

Set数据结构是dict字典，字典是用哈希表实现的。

### 操作

```
sadd <key><value1><value2> ..... 
将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素不会被重复添加
smembers <key>取出该集合的所有值。
sismember <key><value>判断集合<key>是否为含有该<value>值，有1，没有0
scard<key>返回该集合的元素个数。
srem <key><value1><value2> .... 删除集合中的某个元素。
spop <key>随机从该集合中吐出一个值。
srandmember <key><n>随机从该集合中取出n个值。不会从集合中删除 。
smove <source><destination>value把集合中一个值从一个集合移动到另一个集合
sinter <key1><key2>返回两个集合的交集元素。
sunion <key1><key2>返回两个集合的并集元素。
sdiff <key1><key2>返回两个集合的差集元素(key1中的，不包含key2中的)
```

## Hash

### 基本常识

hash 是一个键值对集合。

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 操作

```
hset <key><field><value>给<key>集合中的  <field>键赋值<value>
hget <key1><field>从<key1>集合<field>取出 value 
hmset <key1><field1><value1><field2><value2>... 批量设置hash的值
hexists<key1><field>查看哈希表 key 中，给定域 field 是否存在。 
hkeys <key>列出该hash集合的所有field
hvals <key>列出该hash集合的所有value
hincrby <key><field><increment>为哈希表 key 中的域 field 的值加上增量 
hsetnx <key><field><value>将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
```

## Redis有序集合Zset(sorted set) 

### 基本常识

与set不同之处是有序集合的每个成员都关联了一个score,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以重复。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

### 操作

```
zadd  <key><score1><value1><score2><value2>…
将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

zrange <key><start><stop>  [WITHSCORES]   
返回有序集 key 中，下标在<start><stop>之间的元素
带WITHSCORES，可以让分数一起和值返回到结果集。

zrangebyscore key minmax [withscores] [limit offset count]
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 

zrevrangebyscore key maxmin [withscores] [limit offset count]               
同上，改为从大到小排列。

zincrby <key><increment><value>      为元素的score加上增量

zrem  <key><value>删除该集合下，指定值的元素 

zcount <key><min><max>统计该集合，分数区间内的元素个数 

zrank <key><value>返回该值在集合中的排名，从0开始。
```

## 跳跃表（跳表）

视频理解较好，语言有障碍

## Redis配置文件

### 基本

配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit

大小写不敏感

![image-20220505212236932](resource\image-20220505212236932.png)

### 允许远程访问redis

注释bind=127.0.0.1，默认情况redis仅限本机访问，关闭protect模式，为确保安全，修改为自己的端口号

 ###  cp-backlog      

  在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。
注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果                                                                                         

### timeout

一个空闲的客户端维持多少秒会关闭，0表示关闭该功能。即永不关闭。

### tcp-keepalive

即连接超时设定

单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60

### daemonize

是否为后台

### pidfile

存放pid文件的位置，每个实例会产生一个不同的pid文件

### loglevel 

指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为notice

### logfile 

日志文件名称

### 密码设置

在命令中设置密码，只是临时的。重启redis服务器，密码就还原了。

永久设置，需要再配置文件中进行设置。

需要更改conf中requirepass

设置后通过auth “password”登录

### maxmemory 

​	建议必须设置，否则，将内存占满，造成服务器宕机
设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。
​	如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。
​	但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

### maxmemory-policy

volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
allkeys-lru：在所有集合key中，使用LRU算法移除key
volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
allkeys-random：在所有集合key中，移除随机的key
volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
noeviction：不进行移除。针对写操作，只是返回错误信息

### maxmemory-samples

设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。
一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。

## Redis的发布和订阅

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](resource\pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](resource\pubsub2.png)

![image-20220505222757004](resource\image-20220505222757004.png)

返回的1是订阅者数量

![image-20220505222826468](resource\image-20220505222826468.png)

> 注：发布的消息没有持久化

## Redis新数据类型

### Bitmaps

#### 简介

现代计算机用二进制（位） 作为信息的基础单位， 1个字节等于8位， 例如“abc”字符串是由3个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的ASCII码分别是97、 98、 99， 对应的二进制分别是01100001、 01100010和01100011，如下图

![img](file:///C:\Users\xtdx\AppData\Local\Temp\ksohtml\wps949E.tmp.jpg) 

合理地使用操作位能够有效地提高内存使用率和开发效率。

​	Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1） Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2） Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

![img](file:///C:\Users\xtdx\AppData\Local\Temp\ksohtml\wps949F.tmp.jpg) 

#### 命令

```
setbit<key><offset><value>  #设置Bitmaps中某个偏移量的值（0或1）
getbit<key><offset> 		#不存在返回零
bitcount<key>[start end] 	#统计字符串被设置为1的bit数。一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，start、end 是指bit组的字节的下标数，二者皆包含。
bitop  and(or/not/xor) <destkey> [key…]
```

*offset:偏移量从0开始

在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞

> 注意：redis的setbit设置或清除的是bit位置，而bitcount计算的是byte位置。

#### Bitmaps与set对比

> 假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表
>
> | set和Bitmaps存储一天活跃用户对比 |                    |                  |                        |
> | -------------------------------- | ------------------ | ---------------- | ---------------------- |
> | 数据类型                         | 每个用户id占用空间 | 需要存储的用户量 | 全部内存量             |
> | 集合类型                         | 64位               | 50000000         | 64位*50000000 = 400MB  |
> | Bitmaps                          | 1位                | 100000000        | 1位*100000000 = 12.5MB |
> |                                  |                    |                  |                        |
>
> 
>
> 
>
> 很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的
>
> | set和Bitmaps存储独立用户空间对比 |        |        |       |
> | -------------------------------- | ------ | ------ | ----- |
> | 数据类型                         | 一天   | 一个月 | 一年  |
> | 集合类型                         | 400MB  | 12GB   | 144GB |
> | Bitmaps                          | 12.5MB | 375MB  | 4.5GB |
>
> 
>
> 
>
> 但Bitmaps并不是万金油， 假如该网站每天的独立访问用户很少， 例如只有10万（大量的僵尸用户） ， 那么两者的对比如下表所示， 很显然， 这时候使用Bitmaps就不太合适了， 因为基本上大部分位都是0。
>
> | set和Bitmaps存储一天活跃用户对比（独立用户比较少） |                    |                  |                        |
> | -------------------------------------------------- | ------------------ | ---------------- | ---------------------- |
> | 数据类型                                           | 每个userid占用空间 | 需要存储的用户量 | 全部内存量             |
> | 集合类型                                           | 64位               | 100000           | 64位*100000 = 800KB    |
> | Bitmaps                                            | 1位                | 100000000        | 1位*100000000 = 12.5MB |

### HyperLogLog



能够降低一定的精度来平衡存储空间

HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。

#### 什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

```
pfadd <key>< element> [element ...]   添加指定元素到 HyperLogLog 中

pfcount<key> [key ...] 计算HLL的近似基数

pfmerge<destkey><sourcekey> [sourcekey ...]  将一个或多个HLL合并后的结果存储在另一个HLL中，
```

### Geospatial

元素的2维坐标--经纬度

```
geoadd<key>< longitude><latitude><member> [longitude latitude member...]   添加地理位置（经度，纬度，名称）

geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing

geopos  <key><member> [member...]  获得指定地区的坐标值
geodist<key><member1><member2>  [m|km|ft|mi ]  获取两个位置之间的直线距离
georadius<key>< longitude><latitude>radius  m|km|ft|mi   以给定的经纬度为中心，找出某一半径内的元素
```

> 有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。

## Redis_Jedis_测试

### Jedis所需要的jar包

```
<dependency>
<groupId>redis.clients</groupId>
<artifactId>jedis</artifactId>
<version>3.2.0</version>
</dependency>
```

​                                                                                                                                                                                                                                                                                                                                                      

###  连接到 redis 服务

#### 实例

```
public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        // 如果 Redis 服务设置了密码，需要下面这行，没有就不需要
        // jedis.auth("123456"); 
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```

​                                                                                                                    

>  注意如果连接阿里云要配置你的安全组，并且设置防火墙        （ps：我花了一下午                                

### Redis Java api 实例

略

## springboot整合redis





1、在pom.xml文件中引入redis相关依赖

```
<!-- redis -->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- spring2.X集成redis所需common-pool2-->
<dependency>
<groupId>org.apache.commons</groupId>
<artifactId>commons-pool2</artifactId>
<version>2.6.0</version>
</dependency>
```

2、application.properties配置redis配置

```
#Redis服务器地址
spring.redis.host=192.168.140.136
#Redis服务器连接端口
spring.redis.port=6379
#Redis数据库索引（默认为0）
spring.redis.database= 0
#连接超时时间（毫秒）
spring.redis.timeout=1800000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#最大阻塞等待时间(负数表示没限制)
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=5
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```

3、添加redis配置类

```
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    RedisSerializer<String> redisSerializer = new StringRedisSerializer();
    Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jackson2JsonRedisSerializer.setObjectMapper(om);
    template.setConnectionFactory(factory);

//key序列化方式
        template.setKeySerializer(redisSerializer);
//value序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
//value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;  mxnnm,m,n,mmm       
    }

@Bean
public CacheManager cacheManager(RedisConnectionFactory factory) {
    RedisSerializer<String> redisSerializer = new StringRedisSerializer();
    Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

//解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
// 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```

##  Redis的事务定义        

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
Redis事务的主要作用就是串联多个命令防止别的命令插队。

### Multi、Exec、discard

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。  

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

![image-20220506232640172](resource\image-20220506232640172.png)

组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

### WATCH key [key ...]

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

### unwatch

取消 WATCH 命令对所有 key 的监视。

如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。

## Redis持久化

Redis 提供了2个不同形式的持久化方式。

- RDB（Redis DataBase）

- AOF（Append Of File）

### RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘,它恢复时是将快照文件直接读到内存里

#### 备份

Redis会单独创建一个子进程（fork）来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

> RDB的缺点是最后一次持久化后的数据可能丢失。
>
> 在达到备份标准前宕机，最后一次便没有备份成功

#### Fork

12.2.4.Fork
Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“写时复制技术”



> **写时复制**（**Copy-on-write**，简称**COW**）是一种计算机[程序设计领域的优化策略。其核心思想是，如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上，他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的。此作法主要的优点是如果调用者没有修改该资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。



一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。



#### 恢复

**CONFIG GET dir** 输出的 redis 安装目录为

- 关闭Redis
- 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb
- 启动Redis, 备份数据会直接加载

#### 相关配置

在redis.conf中配置文件名称，默认为dump.rdb

rdb文件的保存路径，也可以修改。默认为Redis启动时命令行所在的目录下

save ：save时只管保存，其它不管，全部阻塞。手动保存。不建议。
bgsave：Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求。
可以通过lastsave 命令获取最后一次成功执行快照的时间

执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义

```
save 秒钟 写操作次数            指定时间内更改次数达标就保存
禁用
不设置save指令，或者给save传入空字符串

stop-writes-on-bgsave-error  
当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes.


rdbcompression 压缩文件
对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。
如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。推荐yes.

rdbchecksum 检查完整性
在存储快照后，还可以让redis使用CRC64算法来进行数据校验，
但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
推荐yes.
```

#### 优势

- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高更适合使用
- 节省磁盘空间
- 恢复速度快

#### 劣势

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
- 虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。
- 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

#### 停止

动态停止RDB：redis-cli config set save ""#save后给空值，表示禁用保存策略

### AOF

以日志的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

#### AOF持久化流程

（1）客户端的请求写命令会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

#### AOF默认不开启

```
appendonly no
```

可以在redis.conf中配置文件名称，默认为 appendonly.aof
AOF文件的保存路径，同RDB的路径一致。

> AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）

####  异常恢复


如遇到AOF文件损坏，通过/usr/local/bin/redis-check-aof--fix appendonly.aof进行恢复

#### AOF同步频率设置

```
appendfsync always

始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

appendfsync everysec

每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

appendfsync no

redis不主动进行同步，把同步时机交给操作系统。
```

#### 压缩

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，redis4.0版本后的重写，是指上就是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。

no-appendfsync-on-rewrite：

如果 no-appendfsync-on-rewrite=yes ,不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）

如果 no-appendfsync-on-rewrite=no,  还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）

触发机制，何时重写

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 

auto-aof-rewrite-percentage：设置重写的基准值，文件达到100%时开始重写（文件是原来重写后文件的2倍时触发）

auto-aof-rewrite-min-size：设置重写的基准值，最小文件64MB。达到这个值开始重写。

例如：文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,

如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。 

#### 优势

- 备份机制更稳健，丢失数据概率更低。
- 可读的日志文本，可以处理误操作。

#### 劣势

- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。
- 每次读写都同步的话，有一定的性能压力。
- 存在个别Bug，造成恢复不能。

l性能建议

因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。 如果使用AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价,一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。

## 主从复制

主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主

l 读写分离，性能扩展

l 容灾快速恢复

![image-20220507172252001](resource\image-20220507172252001.png)

```
slaveof  <主机ip><主机port> 配置主机
```

主机挂掉，重启就行，一切如初
从机重启需重设：slaveof 

##  薪火相传

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。

## 反客为主



当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。
用 slaveof  no one  将从机变为主机。

## 哨兵模式

反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

### 步骤

新建sentinel.conf文件（名字不能改



填写内容

sentinel monitor mymaster  127.0.0.1 6379 1
其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。 



当主机挂掉，从机选举中产生新的主机
(大概10秒左右可以看到哨兵窗口日志，切换了新的主机)
哪个从机会被选举为主机呢？根据优先级别：slave-priority 
原主机重启后会变为从机。

![image-20220507182728877](resource\image-20220507182728877.png)

优先级在redis.conf中默认：slave-priority 100，值越小优先级越高

偏移量是指获得原主机数据最全的

> 由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。



每个redis实例启动后都会随机生成一个40位的runid

## 集群

Redis 集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。

```
cluster-enabled yes    打开集群模式
cluster-config-file nodes-6379.conf  设定节点配置文件名
cluster-node-timeout 15000   设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。
```

> %s/6379/6380     linux搜索替换，  搜索6379， 替换为6380

> cd  /opt/redis-6.2.1/src 先前往你安装目录下的src下‘

redis-cli --cluster create --cluster-replicas 1 192.168.11.101:6379 192.168.11.101:6380 192.168.11.101:6381 192.168.11.101:6389 192.168.11.101:6390 192.168.11.101:6391

一个集群至少要有三个主节点。

选项 --cluster-replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。

分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。



普通登录可能直接进入读主机，存储数据时，会出现MOVED重定向操作。所以，应该以集群方式登录。

-c 采用集群策略连接，设置数据会自动切换到相应的写主机

![image-20220507184435488](resource\image-20220507184435488.png)

```
cluster nodes 查看节点情况
```

### slot

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个.

集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。

集群中的每个节点负责处理一部分插槽。

### 在集群中录入值

在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口。

redis-cli客户端提供了 –c 参数实现自动重定向。

不在一个slot下的键值，是不能使用mget,mset等多键操作。

可以通过{}来定义组的概念，从而使key中{}内相同内容的键值对放到一个slot中去。

### 查询集群中的值

CLUSTER GETKEYSINSLOT <slot><count> 返回 count 个 slot 槽中的键。





### 故障

如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为yes ，那么 ，整个集群都挂掉
如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为no ，那么，该插槽数据全都不能使用，也无法存储。

### 集群的Jedis开发

```
public class JedisClusterTest {
  public static void main(String[] args) { 
     Set<HostAndPort>set =new HashSet<HostAndPort>();
     set.add(new HostAndPort("192.168.31.211",6379));
     JedisCluster jedisCluster=new JedisCluster(set);
     jedisCluster.set("k1", "v1");
     System.out.println(jedisCluster.get("k1"));
  }
}
```

## 缓存穿透

key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

### 解决

一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑， ，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。
解决方案：
（1）对空值缓存：如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟
（2）设置可访问的名单（白名单）：
使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。
（3）采用布隆过滤器：(布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。
布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)
将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被 这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。
（4）进行实时监控：当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务

## 缓存击穿

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

### 解决方案

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。
解决问题：

![image-20220509200404560](resource\image-20220509200404560.png)（1）预先设置热门数据：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长
（2）实时调整：现场监控哪些数据热门，实时调整key的过期时长
（3）使用锁：
（1）就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。
（2）先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key
（3）当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；
（4)当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。

### 缓存雪崩

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。
缓存雪崩与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key

### 解决方案

（1）构建多级缓存架构：nginx缓存 + redis缓存 +其他缓存（ehcache等）
（2）使用锁或队列：
用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况
（3）设置过期标志更新缓存：
记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。
（4）将缓存失效时间分散开：
比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

## 分布式锁

### 背景

单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

#### 解决：

1. 多个客户端同时获取锁（setnx）
2. 获取成功，执行业务逻辑{从db获取数据，放入缓存}，执行完成释放锁（del）
3. 其他客户端等待重试

#### 优化：

setnx获取锁时，设置一个指定的唯一值（例如：uuid）；释放前获取这个值，判断是否自己的锁

#### 优化：LUA脚本保证删除的原子性

为了确保分布式锁可用，我们至少要确保锁的实现同时**满足以下四个条件**：

\- 互斥性。在任意时刻，只有一个客户端能持有锁。

\- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

\- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

\- 加锁和解锁必须具有原子性。

## IO多线程

Redis 6 加入多线程,但跟 Memcached 这种从 IO处理到数据访问多线程的实现模式有些差异。Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。整体的设计大体如下:

![image-20220509205745125](resource\image-20220509205745125.png)

另外，多线程IO默认也是不开启的，需要再配置文件中配置
io-threads-do-reads  yes 
io-threads 4
