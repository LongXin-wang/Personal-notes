- [Redis线程模型](#redis线程模型)
- [使用场景](#使用场景)
  - [分布式锁](#分布式锁)
  - [消息队列](#消息队列)
    - [发布订阅模式 实现 消息队列](#发布订阅模式-实现-消息队列)
    - [List/Zset 实现消息队列](#listzset-实现消息队列)
- [数据结构](#数据结构)
  - [String](#string)
  - [List](#list)
  - [Sets](#sets)
  - [Sorted Sets](#sorted-sets)
  - [Hash](#hash)
- [高可用/可扩展](#高可用可扩展)
  - [过期策略](#过期策略)
    - [持久化的过期键](#持久化的过期键)
    - [过期策略](#过期策略-1)
    - [内存淘汰机制](#内存淘汰机制)
  - [持久化](#持久化)
    - [RDB快照](#rdb快照)
    - [AOF 文件追加](#aof-文件追加)
    - [RDB和AOF的混合](#rdb和aof的混合)
  - [主从复制](#主从复制)
    - [全量复制](#全量复制)
    - [增量复制](#增量复制)
  - [哨兵模式](#哨兵模式)
  - [Redis Cluster](#redis-cluster)
- [Redis事务](#redis事务)
  - [Redis事务相关命令和使用](#redis事务相关命令和使用)
    - [标准的事务执行](#标准的事务执行)
    - [事物取消](#事物取消)
  - [事物出错的处理](#事物出错的处理)
    - [CAS操作实现乐观锁](#cas操作实现乐观锁)

# Redis线程模型
- Redis 的大部分操作**都在内存中完成**，并且采用了高效的数据结构，因此 Redis 瓶颈可能是机器的内存或者网络带宽，而并非 CPU，既然 CPU 不是瓶颈，那么自然就采用单线程的解决方案了；
- Redis 采用单线程模型可以**避免了多线程之间的竞争**，省去了多线程切换带来的时间和性能上的开销，而且也不会导致死锁问题。**CPU 并不是制约 Redis 性能表现的瓶颈所在**，更多情况下是受到内存大小和网络I/O的限制
- Redis 采用了 **I/O 多路复用机制**处理大量的客户端 Socket 请求

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402041106804.png)

# 使用场景

* 缓存
* 限时业务 expire
* 计数器和限流
* 分布式锁 setnx
* 消息队列

## 分布式锁

```python
# NX 表示仅在键不存在时设置值，EX 表示设置键的过期时间
set key value ex 10 nx
do something
del key
```

*问题：线程1占用锁过期后，线程2拿到锁，线程1执行完会继续删除，删除的是线程2的锁*

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402041522924.png)

**怎么解决：每个线程加一个value**

```python
set key random_value ex 10 nx
do something
if random_value == key.value:
    del key 
```

*问题：if判断和del不是原子操作，中间发生了宕机，再判断完宕机了，就有可能是删除线程2的锁*

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402041532869.png)

*lua脚本解决原子性问题*

```java
import org.apache.commons.lang3.StringUtils;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.params.SetParams;
import utils.JedisUtils;

import java.util.Collections;
public class LockExample {
    static final String _LOCKKEY = "REDISLOCK"; // 锁 key
    static final String _FLAGID = "UUID:6379";  // 标识（UUID）
    static final Integer _TimeOut = 90;     // 最大超时时间

    public static void main(String[] args) {
        Jedis jedis = JedisUtils.getJedis();
        // 加锁
        boolean lockResult = lock(jedis, _LOCKKEY, _FLAGID, _TimeOut);
        // 逻辑业务处理
        if (lockResult) {
            System.out.println("加锁成功");
        } else {
            System.out.println("加锁失败");
        }
        // 手动释放锁
        if (unLock(jedis, _LOCKKEY, _FLAGID)) {
            System.out.println("锁释放成功");
        } else {
            System.out.println("锁释放成功");
        }
    }
    /**
     * @param jedis       Redis 客户端
     * @param key         锁名称
     * @param flagId      锁标识（锁值），用于标识锁的归属
     * @param secondsTime 最大超时时间
     * @return
     */
    public static boolean lock(Jedis jedis, String key, String flagId, Integer secondsTime) {
        SetParams params = new SetParams();
        params.ex(secondsTime);
        params.nx();
        String res = jedis.set(key, flagId, params);
        if (StringUtils.isNotBlank(res) && res.equals("OK"))
            return true;
        return false;
    }
    /**
     * 释放分布式锁
     * @param jedis   Redis 客户端
     * @param lockKey 锁的 key
     * @param flagId  锁归属标识
     * @return 是否释放成功
     */
    public static boolean unLock(Jedis jedis, String lockKey, String flagId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(flagId));
        if ("1L".equals(result)) { // 判断执行结果
            return true;
        }
        return false;
    }
}
```

## 消息队列

### 发布订阅模式 实现 消息队列

PSUBSCRIBE pattern [pattern ...] # 订阅一个或多个符合给定模式的频道。
UNSUBSCRIBE news_updates # 取消订阅
PUBLISH news_updates "Latest news: Redis 6.0 released!" # 像频道发送消息

```bash
# 客户端1 - 订阅
redis 127.0.0.1:6379> SUBSCRIBE runoobChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "runoobChat"
3) (integer) 1

# 客户端2 - 发布
redis 127.0.0.1:6379> PUBLISH runoobChat "Redis PUBLISH test"

(integer) 1

redis 127.0.0.1:6379> PUBLISH runoobChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
 1) "message"
2) "runoobChat"
3) "Redis PUBLISH test"
 1) "message"
2) "runoobChat"
3) "Learn redis by runoob.com"
```

```python
# 普通订阅模式 每个人订阅，每个人都能收到
import redis
import time

# 连接到本地Redis服务器
r = redis.StrictRedis(host='localhost', port=6379, db=0)

# 创建一个发布者
def publish_message():
    while True:
        message = input("Enter a message to publish: ")
        r.publish('my_channel', message)

# 创建一个订阅者
def subscribe_channel():
    pubsub = r.pubsub()
    pubsub.subscribe('my_channel')
    # 订阅所有频道
    # pubsub.psubscribe('__keyspace@*__:*', '__keyevent@*__:*')
    for message in pubsub.listen():
        if message['type'] == 'message':
            print(f"Received message: {message['data'].decode('utf-8')}")

# 在两个不同的线程中分别运行发布者和订阅者
import threading
t1 = threading.Thread(target=publish_message)
t2 = threading.Thread(target=subscribe_channel)

t1.start()
t2.start()

t1.join()
t2.join()

```

**频道和队列的结合使用**

```python
# 
PUBLISH channel1 "Message 1"

# 添加消息到队列
LPUSH queue1 "Message 2"

# 使用列表命令如LPOP或RPOP从队列中取出消息，然后使用PUBLISH命令将其发布到频道
RPOP queue1
PUBLISH channel1 "Message 3"
```

### List/Zset 实现消息队列

**List 和 ZSet 的方式解决了发布订阅模式不能持久化的问题**

- List不能被重复订阅，被一个人消费，另外的就没了，没有主题订阅功能
- 任务发布到队列中

```java
import redis.clients.jedis.Jedis;

public class ListMQExample {
    public static void main(String[] args) throws InterruptedException {
        // 消费者 改良版
        new Thread(() -> bConsumer()).start();
        // 生产者
        producer();
    }
    /**
     * 生产者
     */
    public static void producer() throws InterruptedException {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        // 推送消息
        jedis.lpush("mq", "Hello, List.");
        Thread.sleep(1000);
        jedis.lpush("mq", "message 2.");
        Thread.sleep(2000);
        jedis.lpush("mq", "message 3.");
    }
    /**
     * 消费者（阻塞版）
     */
    public static void bConsumer() {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        while (true) {
            // 阻塞读
            for (String item : jedis.brpop(0,"mq")) {
                // 读取到相关数据，进行业务处理
                System.out.println(item);
            }
        }
    }
}
```

Zset消息队列

ZSet 版消息队列相比于之前的两种方式，List 和发布订阅方式在实现上要复杂一些，但 ZSet 因为多了一个 score（分值）属性，从而使它具备更多的功能，例如我们可以用它来存储时间戳，以此来实现延迟消息队列等。

延迟队列场景：

1. 超过 30 分钟未支付的订单，将会被取消
2. 外卖商家超过 5 分钟未接单的订单，将会被取消
3. 在平台注册但 30 天内未登录的用户，发短信提醒


# 数据结构

| 结构类型         | 结构存储的值                               | 结构的读写能力                                                                                                                           | 使用场景                             |
| ---------------- | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| **String字符串** | 可以是字符串、整数或浮点数                 | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作；                                                                 | 存储、计数器、缓存                   |
| **List列表**     | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行push和pop操作，读取单个或多个元素；根据值查找或删除元素；                                                                | 队列、推动列表                       |
| **Set集合**      | 包含字符串的无序集合                       | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等                                                   | 共同好友、标签、点赞用户             |
| **Hash散列**     | 包含键值对的无序散列表                     | 包含方法有添加、获取、删除单个元素                                                                                                       | 存储对象，用户信息、商品信息         |
| **Zset有序集合** | 和散列一样，用于存储键值对                 | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 | 排行榜、优先级队列、按分数排序的数据 |


## String

```python
SET key value：设置指定键的值。
GET key：获取指定键的值。
GETRANGE key start end：获取指定键值的子字符串。
APPEND key value：将指定值追加到键值的末尾。
INCR key：将存储的键值递增1。
DECR key：将存储的键值递减1。
INCRBY key increment：将存储的键值递增指定的值。
DECRBY key decrement：将存储的键值递减指定的值。

SET mykey "Hello"
GET mykey
APPEND mykey " World"
GET mykey
INCR counter
DECR counter
INCRBY counter 5
DECRBY counter 3
```


## List

```python
LPUSH key value1 [value2]：将一个或多个值从左边推入列表。
RPUSH key value1 [value2]：将一个或多个值从右边推入列表。
LPOP key：从左侧弹出一个元素。
RPOP key：从右侧弹出一个元素。
LLEN key：获取列表的长度。
LRANGE key start stop：获取列表中指定范围内的元素。

LPUSH mylist "world" # 左侧推入
LPUSH mylist "hello"
RPUSH mylist "goodbye" # 右侧推入
RPUSH mylist "!"
LRANGE mylist 0 -1 # 获取制定范围元素
```

## Sets

```python
SADD key member1 [member2]：向集合添加一个或多个成员。
SMEMBERS key：获取集合中的所有成员。
SISMEMBER key member：检查成员是否存在于集合中。
SCARD key：获取集合中成员的数量。
SREM key member1 [member2]：从集合中移除一个或多个成员。
SUNION key1 key2：返回给定集合的并集。
SINTER key1 key2：返回给定集合的交集。
SDIFF key1 key2：返回给定集合的差集。

SADD myset "apple"
SADD myset "banana"
SADD myset "orange"
SMEMBERS myset
SISMEMBER myset "banana"
SCARD myset
SREM myset "orange"
SMEMBERS myset
```

## Sorted Sets

```python
ZADD key score1 member1 [score2 member2]：向有序集合添加一个或多个成员，同时指定它们的分数。
ZRANGE key start stop [WITHSCORES]：按照索引范围获取有序集合中的成员。
ZREVRANGE key start stop [WITHSCORES]：按照索引范围逆序获取有序集合中的成员。
ZRANK key member：获取成员在有序集合中的排名。
ZREVRANK key member：获取成员在有序集合中的逆序排名。
ZSCORE key member：获取成员在有序集合中的分数。
ZREM key member1 [member2]：从有序集合中移除一个或多个成员。
ZCARD key：获取有序集合中成员的数量。
ZCOUNT key min max：获取给定分数范围内的成员数量。
ZINCRBY key increment member：将成员的分数增加指定的值。

ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZRANGE myzset 0 -1 WITHSCORES
ZREVRANGE myzset 0 -1 WITHSCORES
ZSCORE myzset "two"
ZREM myzset "one"
ZCARD myzset
ZCOUNT myzset 2 3
ZINCRBY myzset 5 "two"
ZRANGE myzset 0 -1 WITHSCORES
```

## Hash

```python
HSET key field value：为哈希表中的字段设置值。
HGET key field：获取哈希表中指定字段的值。
HMSET key field1 value1 [field2 value2]：为哈希表设置多个字段的值。
HMGET key field1 [field2]：获取哈希表中一个或多个字段的值。
HGETALL key：获取哈希表中所有字段和值。
HDEL key field1 [field2]：删除哈希表中一个或多个字段。
HEXISTS key field：检查哈希表中是否存在指定的字段。
HKEYS key：获取哈希表中的所有字段。
HVALS key：获取哈希表中的所有值。
HLEN key：获取哈希表中字段的数量。

HSET myhash field1 "value1"
HSET myhash field2 "value2"
HGET myhash field1
HMSET myhash field3 "value3" field4 "value4"
HMGET myhash field1 field2 field3
HGETALL myhash
HDEL myhash field2
HEXISTS myhash field2
HKEYS myhash
HVALS myhash
HLEN myhash
```


# 高可用/可扩展

## 过期策略

Redis 中设置过期时间主要通过以下四种方式：

- expire key seconds：设置 key 在 n 秒后过期；
- pexpire key milliseconds：设置 key 在 n 毫秒后过期；
- expireat key timestamp：设置 key 在某个时间戳（精确到秒）之后过期；
- pexpireat key millisecondsTimestamp：设置 key 在某个时间戳（精确到毫秒）之后过期；

字符串中几个直接操作过期时间的方法，如下列表：

- set key value ex seconds：设置键值对的同时指定过期时间（精确到秒）；
- set key value px milliseconds：设置键值对的同时指定过期时间（精确到毫秒）；
- setex key seconds valule：设置键值对的同时指定过期时间（精确到秒）

### 持久化的过期键

RDB模式下，会自动检查，过期键不会被载入，从库北载入，下次RDB的时候被清除了

AOF模式下，会向 AOF 文件追加一条 DEL 命令来显式地删除该键值

### 过期策略

Redis 使用的是惰性删除加定期删除的过期策略。Redis 的定期删除策略并不会遍历删除每个过期键，而是采用随机抽取的方式删除过期键。

### 内存淘汰机制

Redis的运行内存达到某个阈值，触发淘汰机制

默认的策略是 noeviction 当内存超出时不淘汰任何键值，只是新增操作会报错。

LRU（最久未使用）和 LFU（一段时间访问频率最低）

## 持久化

### RDB快照

一方面Redis主进程会fork一个新的快照进程专门来做这个事情，这样保证了Redis服务不会停止对客户端包括写请求在内的任何响应。另一方面这段时间发生的数据变化会以副本的方式存放在另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。从库接收到 RDB 文件后，会先清空当前数据库，然后加载 RDB 文件

### AOF 文件追加

AOF日志采用写后日志，即先写内存，后写日志。

### RDB和AOF的混合

RDB以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。

**每次主库全量生成RDB文件，从库清空数据库；全量RDB - AOF增量 - 全量RDB - AOF增量**

## 主从复制

### 全量复制

采用RDB持久化发给从节点，然后在这个阶段内接收到的新命令，会再发给从库

### 增量复制

RDB和AOF混合复制

## 哨兵模式

```python
# 普通链接
import redis

# 连接 Redis
r = redis.Redis(host='localhost', port=6379, db=0)
# 设置键值对
r.set('name', 'Alice')


# 哨兵模式
sentinel = redis.RedisSentinel(
    [('localhost', 26379)],  # 哨兵节点列表
    socket_timeout=5000,    # 超时时间
    password='password'     # Redis 密码
)

# 获取 Redis 主节点连接
master = sentinel.master_for('mymaster', socket_timeout=5000, password='password')

# 设置键值对
master.set('name', 'Alice'

```
**故障转移**

首先哨兵是一个独立于主从服务之外的服务，它也是一个集群服务。哨兵实例会不断给主服务器发送Ping命令，主服务器在收到命令后，返回一个有效回复，这样哨兵实例认为服务器是正常的。

投票决定

从节点升级主节点：排除不符合条件的节点，选择优先级高的节点

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402041418948.png)

## Redis Cluster

Redis Cluster 是一个去中心化架构，每个节点记录全部 slot 的拓扑分布。这样 Client 如果把 key 分发给了错误的 Redis 节点，Redis 会检查请求 key 所属的 slot，如果发现 key 属于其他节点的 slot，会通知 Client 重定向到正确的 Redis 节点访问。


# Redis事务

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

## Redis事务相关命令和使用

- MULTI ：开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。 
- EXEC：执行事务中的所有操作命令。 
- DISCARD：取消事务，放弃执行事务块中的所有命令。
- WATCH：监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。 
- UNWATCH：取消WATCH对所有key的监视。 ¶

### 标准的事务执行

```Python
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"22"
127.0.0.1:6379>

```

### 事物取消

```Python
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 33
QUEUED
127.0.0.1:6379> set k2 34
QUEUED
127.0.0.1:6379> DISCARD
OK

```

## 事物出错的处理

- **语法错误（编译器错误）**

在开启事务后，修改k1值为11，k2值为22，但k2语法错误，最终导致事务提交失败，k1、k2保留原值。

```Python
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> sets k2 22
(error) ERR unknown command `sets`, with args beginning with: `k2`, `22`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>

```

- **Redis类型错误（运行时错误）**

在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行， 结果k1值改变、k2保留原值

```Python
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> lpush k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>

```

### CAS操作实现乐观锁

被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回nil-reply来表示事务已经失败。

```Python
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC

```