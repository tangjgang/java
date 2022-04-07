#### Redis数据结构类型

##### 五种数据结构

![image-20211022120016304](Redis数据结构.assets/image-20211022120016304.png)

**PS: Redis中，所有key都是字符串**

##### 字符串类型(String)

Redis最基本的数据类型。

二进制安全，可包含任何数据，例如：数字，字符串，jpg图片或序列化对象。

常用命令：

```shell
SET KEY_NAME VALUE
MSET KEY_1 VALUE_1 KEY_2 VALUE_2 .. KEY_N VALUE_N

GET KEY_NAME VALUE
MGET KEY_1 KEY_2 .. KEY_N

# 查看字符串长度
STRLEN KEY_NAME

# 为指定的key设置值及其过期时间，如果key已经存在，将会覆盖旧的值
SETEX KEY_NAME TIMEOUT VALUE	# 时间单位-s
PSETEX KEY_NAME TIMEOUT VALUE	# 时间单位-ms

# 指定的key不存在时，为key设置指定的值
SETNX KEY_NAME VALUE
MSETNX KEY_1 VALUE_1 KEY_2 VALUE_2 .. KEY_N VALUE_N

# 将key中储存的数字值加一，并返回修改后的值
# 如果key不存在，那么key的值会先被初始化为0，然后再执行INCR操作。
# 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误
#本操作的值限制在 64 位(bit)有符号数字表示之内
INCR KEY_NAME 
INCRBY KEY_NAME INCR_AMOUNT	# 加上指定的增量值
DECR KEY_NAME	# 减一
DECRBY KEY_NAME DECREMENT_AMOUNT	# 减去指定的增量值

# 为指定的key追加值，拼接到尾部
# 如果key已经存，会将value追加到key原来的值的末尾。
# 如果key不存在，就像执行 SET key value 一样。
APPEND KEY_NAME NEW_VALUE
```

应用场景：

1、缓存： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。

2、计数器：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。

3、session共享：常见方案spring session + redis实现session共享

##### 哈希(Hash)

空间复杂度：2^32 -1   40亿+元素

Redis的hash表结构如下：

![image-20211022171545300](Redis数据结构.assets/image-20211022171545300.png)

常用命令：

**PS:  所有hash命令都是 h 开头**

```shell
HSET KEY_NAME FIELD VALUE
HMSET KEY_NAME FIELD_1 VALUE_1 FIELD_2 VALUE_2 .. FIELD_N VALUE_N

# 只有当field不存在时，才能设置该字段的值
HSETNX KEY_NAME FIELD VALUE

HGET KEY_NAME FIELD
HMGET KEY_NAME FIELD_1 FIELD_2
HKEYS KEY_NAME	# 获取所有key
HVALS KEY_NAME	# 获取所有value
HGETALL KEY_NAME # 获取所有key value
HLEN KEY_NAME  # 获取字段数

# 查看哈希表的指定字段是否存在
HEXISTS KEY_NAME FIELD_NAME
# 删除一个或多个指定字段
HDEL KEY_NAME FIELD_1.. FIELD_N 

```

应用场景：

1、缓存：更直观，相比string更节省空间，例如：用户信息等

##### 列表(List)

Redis使用的是双向链表。从左右两边都能进行插入、删除元素。

List是有序的，value可以重复，通过下标能取出对应的value值。

空间复杂度：2^32 -1   40亿+元素

![image-20211022211716812](Redis数据结构.assets/image-20211022211716812.png)

使用技巧：

- LPUSH + LPOP = STACK 栈
- LPUSH + RPOP =QUEUE 队列
- LPUSH + ITRIM = CAPPED COLLECTION 有限集合
- LPUSH + BRPOP = MESSAGE QUEUE 消息队列

常用命令：

```shell
# 将一个或多个值插入到列表。
# 如果key不存在，先创建一个空列表，并执行PUSH操作
LPUSH KEY_NAME VALUE_1 .. VALUE_N	# 从头部插入
RPUSH KEY_NAME VALUE_1 .. VALUE_N	# 从尾部插入
# 如果key不存在，则PUSH操作无效
LPUSHX KEY_NAME VALUE_1 .. VALUE_N	# 从头部插入
RPUSHX KEY_NAME VALUE_1 .. VALUE_N	# 从尾部插入

# 通过索引来设置元素的值。list不存在，index越界，均会出错
LSET KEY_NAME INDEX VALUE
# 想pivot前/后插入value
LINSERT KEY_NAME BEFORE|AFTER pivot VALUE	# pivot存在多个，则为列表中第一个出现的值

# 移除并返回列表的第一个元素
LPOP KEY_NAME
RPOP KEY_NAME
# 移除列表中count个与value相等的元素
# count > 0 : 从表头开始向表尾搜索
# count < 0 : 从表尾开始向表头搜索
# count = 0 : 移除表中所有
LREM key count VALUE

# 获取索引的value
LINDEX KEY_NAME INDEX
# 获取列表长度
LLEN KEY_NAME

# 从L1到LN依次判断，返回第一个不为空的list的第一个元素(移除并返回), 如果都没有，则阻塞，
# 直到list有元素插入，返回顺序在前面的list的第一个元素
# 返回数据：返回两个数据，第一个，list名称，第二个，list的第一个元素
# BLPOP-从队头  BRPOP-从队尾
BLPOP L_1 L_2 .. LIST_N TIMEOUT # timeout 过期时间，过期时间内会阻塞。
BRPOP L_1 L_2 .. LIST_N TIMEOUT # timeout 过期时间，过期时间内会阻塞。

# 让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
# 0 第一个；1 第二个；-1 倒数第一；-2 倒数第二
# LPUSH + ITRIM 有限集合
LTRIM KEY_NAME START STOP
```

使用场景：

1、timeline：例如微博的时间轴，有人发布微博，用lpush加入时间轴，展示新的列表信息

##### 集合(Set)

集合类型用于保存多个字符串的元素，并满足如下规则：

1、不允许有重复的元素	2、集合中的元素是无序的，无法通过index获取元素	3、支持集合见的操作，多集合的交集、并集、差集

时间复杂度：O1

空间复杂度：2^32 -1   40亿+元素

![image-20211023200551321](Redis数据结构.assets/image-20211023200551321.png)

常用命令：

**PS:  集合的命令都是以 S 开头的**

```shell
# 添加元素，如果是重复元素，则忽略
SADD KEY_NAME VALUE_1 .. VALUE_N
# 返回集合中元素个数
SCARD KEY_NAME
# 返回第一个集合与其他集合之间的差异
SDIFF KEY_1 KEY_2 .. KEY_N
SDIFFSTORE TARGET_KEY KEY_1 KEY_2 .. KEY_N  # 将差异集合的元素存入TARGET_KEY
# 返回交集
SINTER KEY_1 KEY_2 .. KEY_N
SINTER TARGET_KEY KEY_1 KEY_2 .. KEY_N
# 返回并集
SUNION KEY_1 KEY_2 .. KEY_N
SUNIONSTORE TARGET_KEY KEY_1 KEY_2 .. KEY_N
# 判断元素是否存在
SISMEMBER KEY_NAME member
# 返回所有元素
SMEMBERS KEY_NAME
# 移动元素到另一个集合
SMOVE SOURCE TARGET member
# 随机返回元素
SPOP KEY_NAME
# 随机返回多个元素
SRANDMEMBER KEY_NAME COUNT
# 移除元素
SREM KEY_NAME member_1 .. member_n
```

应用场景：

1、标签（tag），给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人

2、点赞，或点踩，收藏等，可以放到set中实现

##### 有序集合(sorted Set)

保留集合不能有重复成员的特性。

增加有序性：有序集合元素可以排序，增加了分数属性，作为排序依据。（元素不能重复，分数能重复，类似：班级学生和成绩）

时间复杂度：O1

空间复杂度：2^32 -1   40亿+元素

![image-20211023211616432](Redis数据结构.assets/image-20211023211616432.png)

常用命令：

**PS:  有序集合的命令都是以 Z 开头的**

```shell
# 添加元素
ZADD KEY_NAME S1 M1 .. SN MN
# 获取元素个数
ZCARD KEY_NAME
# 通过字典区间，获取元素个数
ZLEXCOUNT KEY_NAME MIN MAX
# 获取分数区间的元素个数
ZCOUNT KEY_NAME MIN MAX
# 为元素增加分数
ZINCRBY KEY_NAME incrment member
# 取多个集合的交集，num_key 指定集合数，TARGET 返回的结果集
ZINTERSTORE TARGET num_key KEY_1 .. KEY_N
....
```

应用场景：

1、排行榜：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行。

##### 位图（bitmap)

通过一个bit位来表示某个元素对应的值或者状态，其中的key就是对应元素本身。

bitmap本身会极大的节省储存空间。

常用命令：

```shell
# 设置key在offset处的bit值(只能是0或者1)
setbit key offset value
# 获得key在offset处的bit值
getbit key offset
# 获得key的bit位为1的个数
bitcount key
# 返回第一个被设置为bit值的索引值
bitpos key value
# 对多个key 进行逻辑运算后存入destkey中
bitop and[or/xor/not] destkey key [key …]
```

应用场景： 

1、用户每月签到，用户id为key ， 日期作为偏移量 1表示签到 

2、统计活跃用户, 日期为key，用户id为偏移量 1表示活跃 

3、查询用户在线状态， 日期为key，用户id为偏移量 1表示在线

##### HyperLongLog

HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

HyperLogLog 只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身

**PS: 基数。比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5**

##### GEO

Redis 3.2 版本新增数据结构。

**Z阶曲线**

在x轴和y轴上将十进制数转化为二进制数，采用x轴和y轴对应的二进制数依次交叉后得到一个六位数编 码。把数字从小到大依次连起来的曲线称为Z阶曲线，Z阶曲线是把多维转换成一维的一种方法。

![image-20211024000202333](Redis数据结构.assets/image-20211024000202333.png)

**Base32编码**

Base32这种数据编码机制，主要用来把二进制数据编码成可见的字符串，其编码规则是：任意给定一 个二进制数据，以5个位(bit)为一组进行切分(base64以6个位(bit)为一组)，对切分而成的每个组进行编 码得到1个可见字符。Base32编码表字符集中的字符总数为32个（0-9、b-z去掉a、i、l、o），这也是 Base32名字的由来。

![image-20211024000242394](Redis数据结构.assets/image-20211024000242394.png)

**geohash算法**

Gustavo在2008年2月上线了geohash.org网站。Geohash是一种地理位置信息编码方法。 经过 geohash映射后，地球上任意位置的经纬度坐标可以表示成一个较短的字符串。可以方便的存储在数据 库中，附在邮件上，以及方便的使用在其他服务中。以北京的坐标举例，[39.928167,116.389550]可以 转换成 wx4g0s8q3jf9 。 

Redis中经纬度使用52位的整数进行编码，放进zset中，zset的value元素是key，score是GeoHash的 52位整数值。在使用Redis进行Geo查询时，其内部对应的操作其实只是zset(skiplist)的操作。通过zset 的score进行排序就可以得到坐标附近的其它元素，通过将score还原成坐标值就可以得到元素的原始坐 标。

用于存储地理位置信息，并对存储的信息进行操作。

- geoadd：添加地理位置的坐标。
- geopos：获取地理位置的坐标。
- geodist：计算两个位置之间的距离。
- georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- geohash：返回一个或多个位置对象的 geohash 值。

应用场景： 

1、记录地理位置

2、计算距离

3、查找"附近的人"

##### Stream

Redis 5.0 版本新增加数据结构。

主要用于消息队列，Redis 本身是有一个 Redis 发布订阅 (pub/sub) 来实现消息队列的功能，但它有个缺点就是消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃。

几乎满足了消息队列具备的全部内容，包括： 

- 消息ID的序列化生成 
- 消息遍历 
- 消息的阻塞和非阻塞读取 
- 消息的分组消费 
- 未完成消息的处理 
- 消息队列监控 

每个Stream都有唯一的名称，它就是Redis的key，首次使用 xadd 指令追加消息时自动创建。

发布订阅 (pub/sub) 可以分发消息，但无法记录历史消息。

 Redis Stream 提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

结构如下：

![image-20211023233158805](Redis数据结构.assets/image-20211023233158805.png)

每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。

上图解析：

- **Consumer Group** ：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer)。
- **last_delivered_id** ：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。
- **pending_ids** ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack (Acknowledge character：确认字符）。