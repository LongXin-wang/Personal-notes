- [分布式 ID 类型](#分布式-id-类型)
- [UUID](#uuid)
- [基于数据库的自增ID](#基于数据库的自增id)
- [基于数据库集群模式](#基于数据库集群模式)
- [基于数据库的号段模式](#基于数据库的号段模式)
- [雪花算法](#雪花算法)
  - [时重回拨问题](#时重回拨问题)

# 分布式 ID 类型

- 全局唯⼀：必须保证ID是全局性唯⼀的，基本要求
- ⾼性能：⾼可⽤低延时，ID⽣成响应要块，否则反倒会成为业务瓶颈
- ⾼可⽤：100%的可⽤性是骗⼈的，但是也要⽆限接近于100%的可⽤性
- 好接⼊：要秉着拿来即⽤的设计原则，在系统设计和实现上要尽可能的简单
- 趋势递增：最好趋势递增，这个要求就得看具体业务场景了，⼀般不严格要求

- UUID
- 数据库⾃增ID
- 数据库多主模式
- 号段模式
- Redis
- 雪花算法（SnowFlake）
- 滴滴出品（TinyID）
- 百度 （Uidgenerator）
- 美团（Leaf）

# UUID

UUID 的⽣成简单到只有⼀⾏代码，输出结果 c2b8c2b9e46c47e3b30dca3b0d447718 ，但UUID却并不适⽤于实际
的业务需求。像⽤作订单号 UUID 这样的字符串没有丝毫的意义，看不出和订单相关的有⽤信息；⽽对于数据库来说⽤作业务 主键ID ，它不仅是太⻓还是字符串，存储性能差查询也很耗时，所以不推荐⽤作 分布式ID 。

优点：
* ⽣成⾜够简单，本地⽣成⽆⽹络消耗，具有唯⼀性

缺点：
- ⽆序的字符串，不具备趋势⾃增特性
- 没有具体的业务含义
- ⻓度过⻓ 16字节128位，36位⻓度的字符串，存储以及查询对MySQL的性能消耗较⼤，MySQL官⽅明确建议主键要尽
- 量越短越好，作为数据库主键 UUID 的⽆序性会导致数据位置频繁变动，严重影响性能。

```java
public static void main(String[] args) {
    String uuid = UUID.randomUUID().toString().replaceAll("-","");
    System.out.println(uuid);
}
```

# 基于数据库的自增ID

当我们需要⼀个ID的时候，向表中插⼊⼀条记录返回 主键ID ，但这种⽅式有⼀个⽐较致命的缺点，访问量激增时MySQL
本身就是系统的瓶颈，⽤它来实现分布式服务⻛险⽐较⼤，不推荐！

优点
- 实现简单，ID单调⾃增，数值类型查询速度快

缺点：
- DB单点存在宕机⻛险，⽆法扛住⾼并发场景

```bash
CREATE DATABASE `SEQ_ID`;
CREATE TABLE SEQID.SEQUENCE_ID (
     id bigint(20) unsigned NOT NULL auto_increment,
     value char(10) NOT NULL default '',
     PRIMARY KEY (id),
) ENGINE=M
```

# 基于数据库集群模式

前边说了单点数据库⽅式不可取，那对上边的⽅式做⼀些⾼可⽤优化，换成主从模式集群。害怕⼀个主节点挂掉没法⽤，那就做双主模式集群，也就是两个Mysql实例都能单独的⽣产⾃增ID。
那这样还会有个问题，两个MySQL实例的⾃增ID都从1开始，

**解决⽅案：设置 起始值 和 ⾃增步⻓**

MySQL_1 配置：
```bash
set @@auto_increment_offset = 1; "# 起始值
set @@auto_increment_increment = 2; "# 步⻓
```

MySQL_2 配置：
```bash
set @@auto_increment_offset = 1; "# 起始值
set @@auto_increment_increment = 2; "# 步⻓
```

那如果集群后的性能还是扛不住⾼并发咋办？就要进⾏MySQL扩容增加节点，这是⼀个⽐较麻烦的事。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403061112467.png)

增加第三台 MySQL 实例需要⼈⼯修改⼀、⼆两台 MySQL实例 的起始值和步⻓，把 第三台机器的ID 起始⽣成位置设定在⽐现有 最⼤⾃增ID 的位置远⼀些，但必须在⼀、⼆两台 MySQL实例 ID还没有增⻓到 第三台MySQL实例 的 起始ID值的时候，否则 ⾃增ID 就要出现重复了，必要时可能还需要停机修改

优点：
- 解决DB单点问题

缺点
- 不利于后续扩容，⽽且实际上单个数据库⾃身压⼒还是⼤，依旧⽆法满⾜⾼并发场景。

# 基于数据库的号段模式

号段模式是当下分布式ID⽣成器的主流实现⽅式之⼀，号段模式可以理解为从数据库批量的获取⾃增ID，每次从数据库取出⼀个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，⽣成1~1000的⾃增ID并加载到内存。表结构如下：

```python
CREATE TABLE id_generator (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的长度',
  biz_type    int(20) NOT NULL COMMENT '业务类型',
  version int(20) NOT NULL COMMENT '版本号,是一个乐观锁，每次都更新version，保证并发时数据的正确性',
  PRIMARY KEY (`id`)
)

update id_generator set max_id = "$max_id+step}, version = version + 1 where version = #{version} and biz_type = XXX
```

具体地，当发号服务请求数据库时，即会获取一个号段范围内的 ID，例如 [1,2000] 号段表示 1~2000 的 ID。发号服务会把这 2000 个 ID 缓存到本地。直到该业务第 2001 次向发号服务请求 ID 时，由于发号服务对于该业务的本地 ID 都已经被使用完毕了。故其会再次请求数据库，获取下一个号段范围的 ID—— 即 [2001,4000] 号段

*假设有10个号段（1-10），需要在3个数据库中分配存储。可以按照一定规则，比如对号段进行取模操作，将1-3号段分配到数据库1，4-6号段分配到数据库2，7-10号段分配到数据库3。这样，当查询特定号段数据时，可以根据号段和规则直接定位到对应的数据库，实现分布式存储和查询。*

# 雪花算法

- 1bit，不⽤，因为⼆进制中最⾼位是符号位，1表示负数，0表示正数。⽣成的id⼀般都是⽤整数，所以最⾼位固定为
0。
- 41bit时间戳，⽤来记录时间戳，毫秒级。41 bit 可以表示的数字多达 2^41 - 1，也就是可以标识 2 ^ 41 -1 个毫秒值，换算成年就是表示 69 年的时间。
- 10bit工作机器，⽤来记录⼯作机器id，代表的是这个服务最多可以部署在 2^10 台机器上，也就是 1024 台机器。10 bit ⾥ 5 个 bit 代表机房 id，5 个 bit 代表机器 id。意思就是最多代表 2 ^ 5 个机房（32 个机房），每个机房⾥可以代表 2 ^ 5 个机器（32 台机器），这⾥可以随意拆分，⽐如拿出4位标识业务号，其他6位作为机器号。可以随意组合。
- 12bit序列号，⽤来记录同毫秒内产⽣的不同id。也就是说可以⽤这个 12 bit 代表的数字来区分同⼀个毫秒内的 4096 个不同的 id。也就是同⼀毫秒内同⼀台机器所⽣成的最⼤ID数量为4096

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403061416944.png)

> 所以从理论上来说， Snowflake 在同⼀毫秒内可以⽣成的全局唯⼀ID数量为：1024 X 4096 = 4194304

优点：
- 快，吞吐量⾼
- 没有啥依赖，实现也简单
- 知道原理之后可以根据实际情况调整各位段，⽅便灵活。

缺点：
- 依赖机器时间，如果发⽣回拨会导致可能⽣成id重复
- workerId 需要在部署时⼿动配置，并且不可重复。如果实例数量达到⼀定量级，管理 workerId 将会变得复杂

```python
import time

class Snowflake:
    def __init__(self, datacenter_id, worker_id):
        self.datacenter_id = datacenter_id
        self.worker_id = worker_id
        self.sequence = 0
        self.last_timestamp = -1

    def _gen_timestamp(self):
        return int(time.time() * 1000)

    def get_id(self):
        timestamp = self._gen_timestamp()

        if timestamp < self.last_timestamp:
            raise ValueError("Clock moved backwards. Refusing to generate id")

        if timestamp == self.last_timestamp:
            self.sequence = (self.sequence + 1) & 4095
            if self.sequence == 0:
                timestamp = self._til_next_millis(self.last_timestamp)
        else:
            self.sequence = 0

        self.last_timestamp = timestamp

        return ((timestamp - 1288834974657) << 22) | (self.datacenter_id << 17) | (self.worker_id << 12) | self.sequence

    def _til_next_millis(self, last_timestamp):
        timestamp = self._gen_timestamp()
        while timestamp <= last_timestamp:
            timestamp = self._gen_timestamp()
        return timestamp
```

## 时重回拨问题

由于强依赖时钟，对时间的要求⽐较敏感，如果系统时间被回调，或者改变，则有可能会造成ID冲突。

造成时钟回拨的原因多种多样，可能是闰秒回拨（最近⼀次闰秒在北京时间2017年1⽉1⽇7时59分59秒，时钟显示07:59:60 出现），可能是NTP同步，还可能是服务器时间⼿动调整。

⼀般有以下⼏种解决⽅案：

1. 少量服务器部署ID⽣成器实例，关闭NTP服务器，严格管理服务器。这种⽅案不需要从代码层⾯解决，完全⼈治。

2. 针对回退时间断的情况，如闰秒回拨仅回拨了1s，可以在代码层⾯通过判断暂停⼀定时间内的ID⽣成器使⽤。虽然少了⼏秒钟可⽤时间，但时钟正常后，业务即可恢复正常。
   
3. 实例启动后，改⽤内存⽣成时间。该⽅案为 baidu 开源的 UidGenerator 使⽤的⽅案，由于实例启动后，时间不再从服务器获取，所以不管服务器时钟如何回拨，都影响不了 Snowflake 的执⾏。如下代码中 lastSecond 变量是⼀个 AtomicLong 类型，⽤以代替系统时间