# Redis notes

## 1. 简介

Redis是什么？

Redis是完全开源免费的，遵守BSD协议，是⼀个⾼性能（NOSQL）的key-value数据库。Redis是⼀个开源的使⽤ANSI C语⾔编写、⽀持⽹络、可基于内存亦可持久化的⽇志型、Key-Value数据库，并提供多种语⾔的API。



Redis是一个内存数据库，也就是Redis是把数据存到内存里面的。这样访问速度就会很快。

[redis中文官网](http://redis.cn/)

[redis英文官网](https://redis.io/)

[开源地址](https://github.com/redis/redis)

## 2. 安装

Redis如何安装呢？

在服务器上有软件安装包

![image-20210430162442886](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210430162442886.png)



直接下载到本地，解压即可

解压需要注意：解压路径不要带中文，不要带空格





解压出来的文件列表：

![image-20210430163017339](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210430163017339.png)

启动：

​	进入对应的目录，打开cmd，执行指令

`redis-server.exe redis.windows.conf`



使用客户端去连接Redis

```cmd
redis-cli.exe -h 192.168.2.100 -p 3306
```

![image-20210430163908189](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210430163908189.png)



连接成功了以后，就可以去操作了

简单使用

![image-20210430164156795](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210430164156795.png)



## 3. 配置

接下来去介绍Redis的配置文件

### 3.1 常规配置

```properties
# 1. Redis 默认不是以守护进程的⽅式运⾏，可以通过该配置项修改，使⽤yes启动守护进程
 daemonize no
#. 当客户端闲置多⻓时间后关闭连接(单位是秒)
 timeout 300
# *3. 指定Redis监听端⼝，默认端⼝为6379，作者在⼀⽚博⽂中解释了为什么选⽤6379作为默认端⼝，因
# 为6379在⼿机按键上MERZ对应的号码，⽽MERZ取⾃意⼤利歌⼿Alessia Merz的名字
port 6379

#*4. 绑定的主机地址 127.0.0.1 意味着这个Redis服务器只有127.0.0.1这个ip才能访问（本机才能访问），如果你的Redis需要被别的远程机器访问，那么应该改成 0.0.0.0 (意味着说有的IP都能访问)
bind 127.0.0.1 
#5. 指定⽇志记录级别，Redis共⽀持四个级别：debug、verbose、notice、warning
 loglevel verbose
#6. 数据库数量(单机环境下)，默认数据库为0，可以使⽤select <dbid>命令在连接上指定数据库id
databases 16

# *10. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis的时候需要通过 AUTH
# <password> 命令提供密码，默认关闭
requirepass foobared
```



### 3.2 持久化配置(面试会问)

Redis给我们提供了两种持久化的方式，一种叫做RDB，一种叫做AOF

#### 3.2.1 RDB

RDB是指我们可以通过给内存快照的方式，来保存内存中的数据，这样去做持久化。

其实就是在某一个时刻，去保存内存里面的全部数据，然后写到磁盘，通过这种方式去做持久化。

配置：

```properties
#save <seconds> <changes>
# Redis默认配置⽂件提供了三个条件
 save 900 1
 save 300 10
 save 60 10000
#持久化⽂件名
 dbfilename dump.rdb
 
# 持久化文件保存路径
dir D:\soft\Redis-x64-3.2.100\tmp
```

RDB这种持久化方式会在快照的时候，保存内存里面所有数据，其实有的时候没有必要保存所有的，我们只需要保存新增的变化即可。

问题：RDB这种持久化方式有没有可能丢失数据？

有可能丢失数据，可能会丢失上一次持久化之后所有写入的数据



#### 3.2.2 AOF

Append only file，其实就是通过追加日志文件的方式来持久化。什么意思呢？其实就是通过把我们输入的命令保存到日志文件中，然后在进行数据恢复的时候，就是通过执行日志文件里面的命令来进行恢复整个数据库的内容。

```properties
# 默认是关闭的，改为yes之后表示开启AOF
appendonly yes

# 文件的名字，保存文件的路径和RDB是一致的
# The name of the append only file (default: "appendonly.aof")
appendfilename "appendonly.aof"

# appendfsync always
appendfsync everysec
# appendfsync no
```

问题：AOF会不会丢失数据呢？

在配置 `appendfsync always` 的前提下，AOF可以做到不丢失数据。但是这种配置方式推不推荐大家使用呢？

因为每一次（update、insert、delete） 操作都会同步持久化到磁盘，速度是比较慢的，甚至和Mysql是一样的了，也就是每一条命令都要持久化到日志文件，都要经过磁盘IO，速度比较慢，所以一般不采用该方式。



我们在以后的工作中，应该是采用AOF还是采用RDB呢？

在公司里面，早期的时候是采用RDB的，但是经过Redis对AOF这种持久化的方式优化了以后，AOF被越来越多的公司所采用，目前而言，一般是两个都用，AOF配置 `appendfsync everysec`



## 4. 命令介绍

Redis支持五种数据结构。哪五种呢？

 ### 4.1 String 字符串

字符串其实就是类似于Map，在Redis里面可以存储Key-value，这个value是单个值的时候，其实就是字符串。

什么叫单个值呢？例如zhangsan、3、3.3 ，其实数字，小数这些都是以字符串的方式来存储的

![image-20210504102229481](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504102229481.png)

- SET key value

- GET key

- INCR 可以对应的key的数值（整型的数值）加⼀( 原⼦操作)

- INCRBY 给数值加上⼀个步⻓

- SETEX expire 过期

- SETNX not exist key不存在的时候再去赋值  不覆盖原来的值
- MSET 设置多个值
- MGET 获取多个值

使用场景：可以利用INCR命令，来统计网站的访问数，也可以统计在线人数。当然String这种数据结构，有很多使用场景，他是我们使用Redis最多的一种数据结构。

```java
String set = jedis.set("1", "一号机");
//返回状态码 OK 成功

Long incr = jedis.incr("3");
Long incr = jedis.incrBy("3", 10);
//返回新值 或 错误：redis.clients.jedis.exceptions.JedisDataException: ERR value is not an integer or out of range

String setex = jedis.setex("3", 3, "33");
//返回状态码 OK 成功

Long kawaii = jedis.setnx("3", "kawaii");
//返回 1 成功 ， 0 不成功

String mset = jedis.mset("4", "四驱车", "5", "五环歌");
//返回状态码 成功 ok

List<String> mget = jedis.mget("4", "5");
//返回 List<String>形式的 值集
```



### 4.2 List

![image-20210504112107121](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504112107121.png)

List这种数据结构其实有点类似于我们之前学过的 `栈`，特点是**有序，可重复**

- LPUSH 后面的元素放在栈顶

- LPOP 返回第⼀个元素，并且在列表上删除该元素 （栈顶）

- LLEN 返回当前的list列表的⻓度
- LINDEX 返回当前的list的指定index下标的元素。没有返回nil，0表示栈顶的元素

- LINSERT 插⼊的位置是按照index的顺序，Before的话得注意 index的值

- LPUSHX 如果list存在，再去push

- LRANGE 可以⽅便的查看某个index范围内的list的值。输⼊的index是从0开始，显示的标号是从1

开始的。

- LREM 删除list⾥的指定的前⼏个（指定value的）元素

  删除指定位置的元素：没有

- LSET 设置指定的位置的元素的值 （修改） 输⼊的index是从0开始，显示的标号是从1开始的。

使用场景：消息队列。消息的最新排行榜

```java
//注：list下标左边从0开始，右边从-1开示

Long lpush = jedis.lpush("老师", "长风", "景甜", "天明");
//返回list更新后的元素数量

Long llen = jedis.llen("老师");
//返回list的元素数量
 
String lpop = jedis.lpop("老师");
//弹出并返回list的首部(左一)

String element0 = jedis.lindex("老师", 0);
//返回index位置的元素，不删除这个元素

List<String> 老师 = jedis.lrange("老师", 0, -1);
//List<String>形式返回list从start到end的元素

Long linsert = jedis.linsert("老师", BinaryClient.LIST_POSITION.BEFORE, "景甜", "大雄");
//返回 -1失败，或 元素个数 插入成功， pivot是指名插入位置参照的元素，不是序号

Long lpushx = jedis.lpushx("老师", "静香");
//返回0 失败，或者 元素数量 成功

Long lrem = jedis.lrem("老师", 2, "大雄");
//返回删除的元素数量 或 0 没有可删的
```



### 4.3 Hash

其实这个是一个二维表

![image-20210504112051612](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504112051612.png)



- **HSET key field value**

  将哈希表 key 中的域 field 的值设为 value 。

  如果 key 不存在，⼀个新的哈希表被创建并进⾏ HSET 操作。

  如果域 field 已经存在于哈希表中，旧值将被覆盖。

- **HGET** 返回哈希表 key 中给定域 field 的值。如果不存在，返回nil

- **HEXISTS key field**

  查看哈希表 key 中，给定域 field 是否存在。

- **HGETALL key**

  返回哈希表 key 中，所有的域和值。

  在返回值⾥，紧跟每个域名(field name)之后是域的值(value)，所以返回值的⻓度是哈希表⼤⼩的两倍。

- **HKEYS key**

  返回哈希表 key 中的所有值。

- HLEN

  返回哈希表 key 中值的数量。

- **HVALS key**

  返回哈希表 key 中所有域的值。HINCRBY

- **HINCRBY key field increment**

  为哈希表 key 中的域 field 的值加上增量 increment 。

  增量也可以为负数，相当于对给定域进⾏减法操作。

  如果 key 不存在，⼀个新的哈希表被创建并执⾏ HINCRBY 命令。

  如果域 field 不存在，那么在执⾏命令前，域的值被初始化为 0 。

  对⼀个储存字符串值的域 field 执⾏ HINCRBY 命令将造成⼀个错误。

  本操作的值被限制在 64 位(bit)有符号数字表示之内。

- **HMGET key field [field ...]**

  返回哈希表 key 中，⼀个或多个给定域的值。

  如果给定的域不存在于哈希表，那么返回⼀个 nil 值。

  因为不存在的 key 被当作⼀个空哈希表来处理，所以对⼀个不存在的 key 进⾏ HMGET 操作将

  返回⼀个只带有 nil 值的表。

- **HMSET key field value [field value ...]**

  同时将多个 field-value (域-值)对设置到哈希表 key 中。

  此命令会覆盖哈希表中已存在的域。

  如果 key 不存在，⼀个空哈希表被创建并执⾏ HMSET 操作。

- **HSETNX key field value**

  将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在。

  若域 field 已经存在，该操作⽆效。

  如果 key 不存在，⼀个新哈希表被创建并执⾏ HSETNX 命令

  其实也就是不覆盖原来的操作

使用场景：

​	二维表可以用来存储我们的Java对象，存对象的时候key：对象的名字，field：对象成员变量的名字，value：成员变量的值

```java
Long hset = jedis.hset("熊猫", "name", "团团");
//返回1 成功

String hget = jedis.hget("熊猫", "name");
//返回 域值 或 null(不存在这个域)

Boolean hexists = jedis.hexists("熊猫", "name");
//返回 true 存在这个域， false 不存在这个域

Long hlen = jedis.hlen("熊猫");
//返回key中储存的field的数量，不存在的key会返回0

Map<String, String> hgetAll = jedis.hgetAll("熊猫");
//以 Map<String, String>的形式返回hashes里的域和值
//key不存在返回空Map {}

Set<String> hkeys = jedis.hkeys("熊猫");
//Set<String>的形式返回key中储存的field的集
//key不存在返回空集 []

 List<String> hvals = jedis.hvals("熊猫");
 // List<String>的形式返回key中所有域的值的列表
 //key不存在返回空列表 []
 
 Long hincrBy = jedis.hincrBy("熊猫", "age", 1);
 //返回这个域修改后的新值 或报错如果域值不是数字 redis.clients.jedis.exceptions.JedisDataException: ERR hash value is not an integer
//field不存在 或 key不存在 都会直接创建

List<String> hmget = jedis.hmget("熊猫", "name", "age");
//List<String>的形式返回多个域的值的列表
//不存在的域返回的列表元素值为null

Map<String, String> hmsetFields = new HashMap<>();
hmsetFields.put("height", "180");
hmsetFields.put("weight", "80");
String hmset = jedis.hmset("熊猫", hmsetFields);
//返回 OK 成功，用Map<String,String>的形式设置多个域

Long hsetnx = jedis.hsetnx("熊猫", "nickname", "圆圆");
//返回0 失败，1 成功
```



### 4.4 SET

这个是一个无序的集合，特点是：**无序、不可重复**

![image-20210504112543612](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504112543612.png)



- **SADD key member [member ...]**

  将⼀个或多个 member 元素加⼊到集合 key 当中，已经存在于集合的 member 元素将被忽略。

  假如 key 不存在，则创建⼀个只包含 member 元素作成员的集合。

  当 key 不是集合类型时，返回⼀个错误。

- **SMEMBERS key**返回集合 key 中的所有成员。

  不存在的 key 被视为空集合。

- **SISMEMBER key member**

判断 member 元素是否集合 key 的成员。

- **SCARD key**

  返回集合 key 的基数(集合中元素的数量)。

- **SPOP key**

  移除并返回集合中的⼀个随机元素。

  如果只想获取⼀个随机元素，但不想该元素从集合中被移除的话，可以使⽤ *SRANDMEMBER*。

- **SRANDMEMBER key [count]**

  如果命令执⾏时，只提供了 key 参数，那么返回集合中的⼀个随机元素。

​	随机取出 count个元素（不删除）

- **SINTER key [key ...]**

  返回⼀个集合的全部成员，该集合是所有给定集合的交集。

  不存在的 key 被视为空集。

- **SINTERSTORE destination key [key ...]**

  这个命令类似于 *SINTER* 命令，但它将结果保存到 destination 集合，⽽不是简单地返回结果集。

  如果 destination 集合已经存在，则将其覆盖。

- **SUNION key [key ...]**

  返回⼀个集合的全部成员，该集合是所有给定集合的并集。

  不存在的 key 被视为空集。

- **SUNIONSTORE destination key [key ...]**

  这个命令类似于 *SUNION* 命令，但它将结果保存到 destination 集合，⽽不是简单地返回结果集。

  如果 destination 已经存在，则将其覆盖。

- **SDIFF key [key ...]**

  返回⼀个集合的全部成员，该集合是所有给定集合之间的差集。

  不存在的 key 被视为空集。

- **SDIFFSTORE destination key [key ...]**

  这个命令的作⽤和 *SDIFF* 类似，但它将结果保存到 destination 集合，⽽不是简单地返回结果集。

  如果 destination 集合已经存在，则将其覆盖。 destination 可以是 key 本身。

- **SMOVE source destination member**

  将 member 元素从 source 集合移动到 destination 集合。SMOVE 是原⼦性操作。

- **SREM key member [member ...]**

  移除集合 key 中的⼀个或多个 member 元素，不存在的 member 元素会被忽略。

  当 key 不是集合类型，返回⼀个错误。

使用场景：

1. 求共同好友
2. 好友推荐
3. 统计网站的独立IP

```java
Long sadd = jedis.sadd("校草", "景甜", "雪茄", "长风");
//返回成功插入到集中的元素的数量

Set<String> smembers = jedis.smembers("校草");
//Set<String>的形式返回这个key中的元素
//keyu不存在返回空集 []

Boolean sismember = jedis.sismember("校草", "天明");
//返回true 是 或 false 不是 这个集的成员

Long scard = jedis.scard("校草");
//返回集合中元素数量

String spop = jedis.spop("校草");
//移除并返回集合中的⼀个随机元素

String srandmember = jedis.srandmember("校草");
List<String> srandmember2 = jedis.srandmember("校草", 2);
//随机返回集中的1个或多个元素（不删除）

Set<String> sinter = jedis.sinter("校草", "学霸");
//Set<String>的形式返回多个集的交集
//没有交集返回空集 []

Long sinterstore = jedis.sinterstore("校草学霸", "校草", "学霸");
//将交集结果保存到目标集，返回值为交集元素数量


Set<String> sunion = jedis.sunion("校草", "学霸");
//返回并集

Long sunionstore = jedis.sunionstore("校草和学霸", "校草", "学霸");
//把并集并保存到目标集，返回值是并集的元素数量

Set<String> sdiff = jedis.sdiff("校草", "学霸");
//返回差集

Long sdiffstore = jedis.sdiffstore("校草非学霸", "校草", "学霸");
//把差集保存到指定集，返回值是差集的元素数量

Long smove = jedis.smove("校草", "学霸", "雪茄");
//将元素从src集移动到dst集，返回1成功，0不成功

Long srem = jedis.srem("学霸", "珠珠", "景甜");
//移除key集中的一个或多个元素，返回值为被移除的元素数量
```



### 4.5 SortSET

有序集合。其实和二维表比较类似

![image-20210504143531369](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504143531369.png)



- **ZADD key score member [[score member] [score member] ...]**

  将⼀个或多个 member 元素及其 score 值加⼊到有序集 key 当中。

- **ZCARD key**

  返回有序集 key 的基数。

- **ZSCORE key member**

  返回有序集 key 中，成员 member 的 score 值。

  如果 member 元素不是有序集 key 的成员，或 key 不存在，返回 nil 。

- **ZCOUNT key min max**

  返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的

  成员的数量。关于参数 min 和 max 的详细使⽤⽅法，请参考 *ZRANGEBYSCORE* 命令。

- **ZINCRBY key increment member**

  为有序集 key 的成员 member 的 score 值加上增量 increment 。

- **ZRANGE key start stop [WITHSCORES]**

  返回有序集 key 中，指定区间内的成员。

  其中成员的位置按 score 值递增(从⼩到⼤)来排序。

  具有相同 score 值的成员按字典序(lexicographical order )来排列。

- **ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]**

  返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有

  序集成员按 score 值递增(从⼩到⼤)次序排列。

  根据指定的分值范围去查找

- ZRANK （排名从0开始）

  **ZRANK key member**

  返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从⼩到⼤)顺序排列。

- ZREVRANGE

  **ZREVRANGE key start stop [WITHSCORES]**

​	返回有序集 key 中，指定区间内的成员。

​	其中成员的位置按 score 值递减(从⼤到⼩)来排列。

​	具有相同 score 值的成员按字典序的逆序(reverse lexicographical order)排列。

- ZREVRANGEBYSCORE

  **ZREVRANGE key start stop [WITHSCORES]**

  返回有序集 key 中，指定区间内的成员。

- ZREVRANK

  **ZREVRANK key member**

  返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从⼤到⼩)排序。

  排名以 0 为底，也就是说， score 值最⼤的成员排名为 0 。

  使⽤ *ZRANK* 命令可以获得成员按 score 值递增(从⼩到⼤)排列的排名。

- ZREM

  **ZREM key member [member ...]**

  移除有序集 key 中的⼀个或多个成员，不存在的成员将被忽略。

  当 key 存在但不是有序集类型时，返回⼀个错误。ZREMRANGEBYRANK

- **ZREMRANGEBYRANK key start stop**

  移除有序集 key 中，指定排名(rank)区间内的所有成员。

  区间分别以下标参数 start 和 stop 指出，包含 start 和 stop 在内。

- ZREMRANGEBYSCORE 根据分数区间去删除

  **ZREMRANGEBYSCORE key min max**



使用场景：

1. 可以很方便的去统计排名（实时积分排行榜）
2. 主播排行榜



```java
Long zadd = jedis.zadd("积分", 1, "LGD");
Map<String, Double> params = new HashMap<>();
params.put("IG", 2D);
params.put("VG", 3D);
Long zaddmulti = jedis.zadd("积分", params);
//将⼀个或多个 member 元素及其 score 值加⼊到有序集 key 当中

Long zcard = jedis.zcard("积分");
//返回key的基数（member数）

Set<String> zrange = jedis.zrange("积分", 0, -1);
//Set<String>形式返回有序集 key 中，指定区间内的成员

Double zscore = jedis.zscore("积分", "LGD");
//返回指定memeber的score值

Long zcount = jedis.zcount("积分", 1, 3);
//返回score在[min,max]之间的member的数量

Double zincrby = jedis.zincrby("积分", 10, "LGD");
//为这个member的score值加上增量，返回这个score的新值

Set<String> zrangeByScore = jedis.zrangeByScore("积分", 1, 3);
//Set<String>形式返回score值在[min,max]之间的member
//参数还可以设置偏移量限制结果集

Long zrank = jedis.zrank("积分", "LGD");
//返回这个member的排名，从0开始


Set<String> zrevrange = jedis.zrevrange("积分", 0, -1);
//返回有续集指定区间的member，按score从大到小排列

Set<String> zrevrangeByScore = jedis.zrevrangeByScore("积分", 3, 2);
//按score在[max,min]区间的member，从大到小排列

Long zrevrank = jedis.zrevrank("积分", "LGD");
//返回这个member的排名，从大到小的排名

Long zrem = jedis.zrem("积分", "LGD", "OG");
//返回成功移除的元素的数量，忽略不存在的移除目标

Long zremrangeByRank = jedis.zremrangeByRank("积分", 0, 2);
//移除指定排名区间内的所有member，返回移除的member数量

Long zremrangeByScore = jedis.zremrangeByScore("积分", 2, 3);
//移除指定score区间的所有member，返回移除的数量
```



## 5. Java客户端

Jedis客户端

如何使用Jedis客户端去操作Redis呢？

- 导包

  ```xml
  <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.9.0</version>
  </dependency>
  ```

- 配置

- 使用

  ```java
  @Test
  public void test01(){
  
      // 配置
      Jedis jedis = new Jedis("127.0.0.1",6379);
  
      // 密码
      jedis.auth("password");
  
  
      jedis.set("zhangfei","张飞");
  
  }
  ```



我们发现，这个Java客户端，基本上API的名字和我们使用Redis命令行操作的命令是一致的

## 6. 图形化界面客户端

[下载地址](https://gitee.com/qishibo/AnotherRedisDesktopManager/releases)

![image-20210504160141648](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\image-20210504160141648.png)



## 7. 内存淘汰策略

我们的Redis是把数据存储在内存中，但是内存资源是有限的，所以当内存不足的时候，我们Redis需要去存储新的数据的时候该怎么办呢？

这就得需要知道不同的Redis的内存淘汰策略

Redis默认给我们提供了8中不同的内存淘汰策略（5.0这个版本以后）

- volatile-lru：从已设置过期时间的数据集中挑选最近最少使⽤的数据淘汰

- lru：least recently used 最近最少使用

- volatile-lfu：从已设置过期的Keys中，删除⼀段时间内使⽤次数最少使用的key

- volatile-ttl：从已设置过期时间的数据集中挑选最近将要过期的数据进⾏淘汰

- volatile-random：从已设置过期时间的数据集中随机选择数据淘汰

- allkeys-lru：从数据集中挑选最近最少使⽤的数据淘汰

- allkeys-lfu：从所有的keys中，删除⼀段时间内使⽤次数最少的key

- allkeys-random：从数据集中随机选择数据淘汰

- no-enviction（驱逐）：禁⽌驱逐数据（不采⽤任何淘汰策略。默认即此配置），内存不⾜时，针对写操作，返回错误信息

  

建议：了解了Redis的淘汰策略之后，在平时使⽤时应尽量主动设置/更新key的expire时间，主动剔除

不活跃的旧数据，有助于提升查询性能

## <mark>事务</mark>

WATCH : 监视的keys在EXEC之前一旦有一个被修改了，整个事务被取消，EXEC返回nil-reply表示事务失败。

MULTI : 开启事务

EXEC : 提交事务

DISCARD : 清空事务队列，结束事务状态为常规状态，释放被监视的keys如果有的话。

UNWATCH : 释放监视的keys，如果执行了EXEC或DISCARD就没有必要在手动释放了。当客户端断开连接时， 该客户端对键的监视也会被取消。



WATCH 乐观锁：

```
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

如果在 WATCH 执行之后， EXEC 执行之前， 有其他客户端修改了被监视一个或多个key的值， 那么当前客户端的事务就会失败。 程序需要做的， 就是不断重试这个操作， 直到没有发生碰撞为止。

这种形式的锁被称作乐观锁， 它是一种非常强大的锁机制。 并且因为大多数情况下， 不同的客户端会访问不同的键， 碰撞的情况一般都很少， 所以通常并不需要进行重试。





关于回滚问题

在exec之前出错其实是不执行，exec之后有命令失败了其他的照旧执行。算是有回滚还是没回滚呢。

当然这种设计的原因是不要以拖累数据库性能为代价来替程序员错误的编程背锅是有道理的。

# Jedis

[Jedis wiki]([Home · redis/jedis Wiki (github.com)](https://github.com/redis/jedis/wiki))

## 简单用法

导包 :  Jedis 和 Apache Commons Pool2

Maven配置

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

基本示例：

Jedis资源连接池，可以静态保存这个池子，它是线程安全的。

连接池配置见[Commons Pool2's configuration]( http://commons.apache.org/proper/commons-pool/apidocs/org/apache/commons/pool2/impl/GenericObjectPoolConfig.html)

```java
JedisPool pool = new JedisPool(new JedisPoolConfig(), "localhost");

// Jedis implements Closeable. Hence, the jedis instance will be auto-closed after the last statement.
try (Jedis jedis = pool.getResource()) {
    /// ... do stuff here ... for example
    jedis.set("foo", "bar");
    String foobar = jedis.get("foo");
    jedis.zadd("sose", 0, "car"); jedis.zadd("sose", 0, "bike"); 
    Set<String> sose = jedis.zrange("sose", 0, -1);
}
//池中的jedis资源的close方法是归还连接
/// ... when closing your application:
pool.close();

//也可以传统地在finally中关闭

//Jedis jedis = new Jedis(参数或没有);//这样获得的jedis的close方法是真的关闭连接了。
```



## 主从派发设置

Redis的设计天然应对主从派发。

Redis网络种可以后很多Redis服务器，“主”或者“从”。但对于客户端没什么区别，"从的确"也能接受写入请求，不同在于"从"的写入变动不会复制派发到网络中，但所有的"从"都会同步"主"的数据并最终都被"主"的数据覆盖。如此一来把"读去"派给"从"，"主"负责"写入"就有意义了。一个Redis服务器有"主"了之后也不妨碍自己可以成为其他服务器的"主"。

从属身份设置

- 在每个Redis服务器的配置文件中分别设置

  ```properties
  # slaveof <masterip> <masterport>
  ```

- 对现存Jedis实例调用slaveOf方法，传入IP(或"localhost")和端口号为参数

  ```java
  jedis.slaveof("localhost", 6379);  //  if the master is on the same PC which runs your code
  jedis.slaveof("192.168.1.35", 6379); 
  ```

注意：自Redis2.6，从属默认为只读，对其写入的请求会报错。可以修改这个配置让从属也能写入且不报错，但从属内的变动不会复制派发，这些变动可能被过度写入，Jedis实例管理混乱会有种隐藏风险。



易主设置

主体宕机，从现有从属种提拔一个作为新主体，新主体先解除自身从属身份，其他从属变更从属关系到新主体

```java
slave1jedis.slaveofNoOne();//解除自身从属身份
slave2jedis.slaveof("192.168.1.36", 6379); //变更从属关系到新主体
```



## 事务

```java
jedis.watch (key1, key2, ...);//开启监视
Transaction t = jedis.multi();//开启事务
t.set("foo", "bar");//执行队列
...
t.exec();//提交事务
```

执行队列的命令有返回值的话

```java
Transaction t = jedis.multi();
t.set("fool", "bar"); 
Response<String> result1 = t.get("fool");

t.zadd("foo", 1, "barowitch"); 
t.zadd("foo", 0, "barinsky"); 
t.zadd("foo", 0, "barikoviev");
Response<Set<String>> sose = t.zrange("foo", 0, -1);   // get the entire sortedset
t.exec();                                              // dont forget it

String foolbar = result1.get();                       // use Response.get() to retrieve things from a Response
int soseSize = sose.get().size();                      // on sose.get() you can directly call Set methods!

// List<Object> allResults = t.exec();                 // you could still get all results at once, as before
```

注意:

exect()提交前Response还没有结果（是一种Future任务），提交前Response.get()结果会报错。可以使用最后一行的方式获取所有的结果列表（包括返回的结果或状态码）。

Redis不允许在事务过程中使用事务中间状态的结果，会报错，原因同上。

```java
// this does not work! Intra-transaction dependencies are not supported by Redis!
jedis.watch(...);
Transaction t = jedis.multi();
if(t.get("key1").equals("something"))
    t.set("key2", "value2");
else 
    t.set("key", "value");
```

但是类似setnx等条件执行语句在事务间是支持的。可以使用eval/ LUA脚本自定义命令。

<mark>TODO : 这里有疑问，事务间t.get()不行，就不能用jedis.get()吗？</mark>



# 中文乱码

redis-cli客户端SET输入和GET输出中文乱码

redis-cli --raw 能让GETshu'c

# 常见面试题



## 什么是Redis持久化？Redis有哪几种持久化方式？优缺点是什么？

答：

​    Redis的持久化就是把redis中的数据从内存中写入到磁盘中去，防止服务宕机了内存数据丢失。

持久化策略：

1， RDB：（默认开启）

RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。数据恢复的时候会将文件中的数据通过rdbload函数读取到内存。

​    

2， AOF

AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。

 

二者的比较：

​    RDB：

​       优点: 

1 一旦采用该方式，那么你的整个Redis数据库将只包含一个文件，这对于文件备份而言是非常完美的。比如，你可能打算每个小时归档一次最近24小时的数据，同时还要每天归档一次最近30天的数据。通过这样的备份策略，一旦系统出现灾难性故障，我们可以非常容易的进行恢复。

2 性能最大化。对于Redis的服务进程而言，在开始持久化时，它唯一需要做的只是fork出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行IO操作了。

3 如果数据量很大，RDB启动效率会更高

​       缺点：

1， 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么RDB将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。如果在持久化的时候宕机，那么可能整个数据都可能丢失![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\clip_image001.png)

2， 由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器压力过大，影响到redis主进程的业务相应速度

 

AOF：

​    优点：

1， 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。

2， 由于该机制对日志文件的写入操作采用的是append模式，因此在写入的过程中即使出现服务宕机的情况，那么也不会破坏日志文件中之前的内容。

3， AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

缺点：

1， 对于相同数量的数据集而言，AOF文件 通常大于RDB文件，RDB在恢复大数据集时的速度比AOF要快

2， 根据同步策略不同，AOF在运行效率上往往会慢于RDB，

二者选择的标准：

​    二者选择的标准，就是看系统是愿意牺牲一些性能，换取更高的缓存一致性（aof），还是愿意写操作频繁的时候，不启用备份来换取更高的性能，待手动运行save的时候，再做备份（rdb）。rdb这个就更有些 eventually consistent的意思了。不过生产环境其实更多都是二者结合使用的。 

 

 

## Redis支持的数据类型？

​    String，Hash，Set，List，Zset(SortSet)

建议了解各自的应用场景

 

 

 

## Redis的架构模式有哪些？讲讲各自的特点

1， 单机版

缺点：1、内存容量有限 2、处理能力有限 3、无法高可用（只有一台机器，假如这台机器挂了，那么相当于整个缓存服务都挂了，所以无法高可用）

 

主从复制：

Redis 的复制（replication）功能允许用户根据一个 Redis 服务器来创建任意多个该服务器的复制品，其中被复制的服务器为主服务器（master），而通过复制创建出来的服务器复制品则为从服务器（slave）

特点：

​    主机master中的内容和slave中的内容完全一致，那么这种模式下可以做读写分离，master负责写，slave负责读，这样可以缓解master的压力，

 

问题：

1， 无法高可用，一旦master挂了，那么整个写功能都挂了

2， 没有解决master写的压力

 

 

哨兵模式：如图所示

 ![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\clip_image002.png)

​    

​    可以理解为主从架构的升级版

​    

多了一个sentinel角色（哨兵），它和redis的主机之间保持着心跳链接机制，一旦发现redis 的master服务挂了，那么会切换master到一台slave上，并且可以通过API向管理员或者其他应用发送通知

 

​    

​    

 

 

 

 

 

集群<Proxy型>：如图所示

 ![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\clip_image003.png)

 

 相当于哨兵模式的升级版，master升级为集群，哨兵升级为集群，但是要通过twemproxy去映射请求，然后分发到master上

 

映射有多种hash算法：

   MD5、CRC16、CRC32、CRC32a、hsieh、murmur、Jenkins

 

优点：  

对比哨兵模式又有了进一步的提升，处理能力更强

支持失败节点自动删除

后端集群存储对于开发使用人员来说透明，和操作单个redis基本没区别

缺点：

 新增了Proxy，维护困难

 

​     

 

 

 

 

 

 

 

 

 

 

 

 

集群<直连型>

 

![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\clip_image004.png)

 

优点:

当前比较流行的“去中心化”架构，一台服务的宕机不会立马影响到整个服务

可拓展性，拓展节点很方便

高可用性，部分节点不可用时，集群仍然可用

 

缺点：

 数据通过异步复制，所以不能保证数据的强一致性

 资源隔离性差，容易互相影响

 

## 什么是缓存穿透？如何避免？什么是缓存雪崩？何如避免？

### 缓存穿透：

​    

一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。

如何避免：

​    去后面DB查找之后如果还是没有，那么就把这个key缓存起来，值可以设为空值或者一个特定的值，设置一个合理的缓存时间（5分钟左右）那么在下一个请求过来之后，先去查找redis，如果拿到的这个值是我们之前设置的那么值，那么就直接返回空

 

### 缓存雪崩：

​    缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

1， 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。

2， 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。

3， 设置热点数据永远不过期。

 

 

### 缓存击穿

​     缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

 

解决方案：

1， 设置热点数据永不过期，对于秒杀这种活动应用场景，可以使用缓存预热，也就是在真正上线之前就把缓存加入到redis中，这样等到活动一开始的时候，就可以直接去查缓存了，从而不会造成大量的并发请求去查询数据库

2， 增加互斥锁（Mutex key）

 

 

## Mysql中有2000w条数据，而redis中只有20w条数据，如何保证redis中存的都是热点数据？

相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略（回收策略）。redis 提供 6种数据淘汰策略：

volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

 

 

## Redis是单线程还是单进程? Redis如何控制并发问题？

Redis为单进程单线程模式，采用队列模式将并发访问变为串行访问。Redis本身没有锁的概念，Redis对于多个客户端连接并不存在竞争，但是在Jedis客户端对Redis进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是

由于客户端连接混乱造成。对此有2种解决方法：

1.客户端角度，为保证每个客户端间正常有序与Redis进行通信，对连接进行池化，同时对客户端读写Redis操作采用内部锁synchronized。

2.服务器角度，利用setnx实现锁。

　　　注：对于第一种，需要应用程序自己处理资源的同步，可以使用的方法比较通俗，可以使用synchronized也可以使用lock；第二种需要用到Redis的setnx命令，但是需要注意一些问题。

 

## Redis有哪些常用的命令?了解Watch命令吗？

watch 用于在进行事务操作的最后一步也就是在执行exec 之前对某个key进行监视

如果这个被监视的key被改动，那么事务就被取消，否则事务正常执行.

一般在MULTI 命令前就用watch命令对某个key进行监控.如果想让key取消被监控，可以用unwatch命令

## 如何使用Redis实现分布式锁?思路是什么？

最简单的一种：

![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Database\Redis-notes.assets\clip_image006.jpg)

这种是有问题的，大家可以思考一下，附件有有demo

 

这些是频率比较高的面试，可以看下下面这个链接

redis面试题参考链接https://msd.misuland.com/pd/3065794831805580868

