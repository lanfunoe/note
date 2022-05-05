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

   单实例关闭：redis-cli shut也可以进入终端后再关闭

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

需要更改config中requirepass

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

