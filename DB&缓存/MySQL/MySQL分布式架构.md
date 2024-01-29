- [分布式数据库](#分布式数据库)
- [数据分片](#数据分片)
  - [选出分片键](#选出分片键)
    - [RANG分片算法（不建议）](#rang分片算法不建议)
    - [HASH分片算法 （取模/余）](#hash分片算法-取模余)
  - [分库分表](#分库分表)
    - [扩缩容](#扩缩容)
  - [多活架构](#多活架构)
- [索引设计](#索引设计)
  - [查询如何定位分片](#查询如何定位分片)
  - [全局表](#全局表)
- [访问分布式数据库模式](#访问分布式数据库模式)
    - [分库分表直接访问](#分库分表直接访问)
  - [使用中间件技术](#使用中间件技术)
- [分布式事务](#分布式事务)
  - [2PC（两阶段提交）的分布式事务实现](#2pc两阶段提交的分布式事务实现)
  - [柔性事务](#柔性事务)

# 分布式数据库

分布式数据库本身分为计算层、元数据层和存储层：

- 计算层就是之前单机数据库中的 SQL 层，用来对数据访问进行权限检查、路由访问，以及对计算结果等操作。
- 元数据层记录了分布式数据库集群下有多少个存储节点，对应 IP、端口等元数据信息是多少。当分布式数据库的计算层启动时，会先访问元数据层，获取所有集群信息，才能正确进行 SQL 的解析和路由等工作。另外，因为元数据信息存放在元数据层，那么分布式数据库的计算层可以有多个，用于实现性能的扩展。
- 存储层（分片）用来存放数据，但存储层要和计算层在同一台服务器上，甚至不求在同一个进程中。

![](https://secure2.wostatic.cn/static/2RkuZYoEtWTyRiaz3SN8vY/image.png?auth_key=1705912810-ek21EMaF3Eu742VFg8SJMD-0-f1e9e70f50ecc04d73eeee8991015c1b)

从可用性的角度看，如果存储层发生宕机，那么只会影响 1/N 的数据，N 取决于数据被打散到多少台服务器上。所以，分布式数据库的可用性对比单机会有很大提升，单机数据库要实现99.999% 的可用性或许很难，但是分布式数据库就容易多了。

当然，分布式数据库也存在缺点：正因为数据被打散了，分布式数据库会引入很多新问题，比如自增实现、索引设计、分布式事务等（这些将在后面的内容中具体介绍）。

对于元数据来说，虽然它的数据量不大，但数据非常关键，一旦宕机则可能导致中间件无法工作，所以，元数据要通过副本技术保障高可用。

**最后，每个分片存储本身都有副本，通过我们之前学习的高可用技术，保证分片的可用性。也就是说，如果分片 1 的 MySQL 发生宕机，分片 1 的从服务器会接替原先的 MySQL 主服务器，继续提供服务。**

但由于使用了分布式架构，那么即使分片 1 发生宕机，需要 60 秒的时间恢复，这段时间对于业务的访问来说，只影响了 1/N 的数据请求。

# 数据分片

- 分布式数据库数据分片要先选择一个或多个字段作为分片键；
- 分片键的要求是业务经常访问的字段，且业务之间的表大多能根据这个分片键进行单元化；
- 如果选不出分片键，业务就无法进行分布式数据库的改造；
- 选择完分片键后，就要选择分片算法，通常是 RANGE 或 HASH 算法；
- 海量 OLTP 业务推荐使用 HASH 算法，强烈不推荐使用 RANGE 算法；
- 分片键和分片算法选择完后，就要进行分库分表设计，推荐不同库名表名的设计，这样能方便后续对分片数据进行扩缩容；
- 实际进行扩容时，可以使用过滤复制，仅复制需要的分片数据。

## 选出分片键

布式数据库架构设计的原则是：**选择一个适合的分片键和分片算法，把数据打散，并且业务的绝大部分查询都是根据分片键进行访问**。绝大部分操作需要在一个分片中完成

比如用自增键，订单的年份键等

多表查询，用关联字段做分片键

### RANG分片算法（不建议）

对于表orders，采用 RANGE 算法进行分片后，表 orders 中，1992 年的订单数据存放在分片 1 中、1993 年的订单数据存放在分片 2 中、1994 年的订单数据存放在分片 3中，依次类推，如果要存放新年份的订单数据，追加新的分片即可。

### HASH分片算法 （取模/余）

对于订单号除以 4，余数为 0 的数据存放在分片 1 中，余数为 1 的数据存放在分片 2 中，余数为 2 的数据存放在分片 3 中，以此类推。

![](https://secure2.wostatic.cn/static/72X7waQQDX9Q1PsVRusR89/image.png?auth_key=1705912810-ftN2P9L92kHPQCwawpECZz-0-210f564f93c89c37a7b60e0591e980eb)

## 分库分表

每个分片的库名不一样，表名也不一样，如分片 1 的表在库名 tpch01下，表名为oders01；分片 2 的表名在库名 tpch02，表名为 orders02；分片 3 的表名在库名tpch03，表名为 orders03；分片 3 的表名在库名 tpch04，表名为 orders04。

```Python
分片 = 实例 + 库 + 表 = ip@port:db_name:table_name

```

### 扩缩容

在 HASH 分片的例子中，我们把数据分片到了 4 个节点，然而在生产环境中，为了方便之后的扩缩容操作，推荐一开始就把分片的数量设置为不少于 1000 个。

不用担心分片数量太多，因为分片 1 个还是 1000 个，管理方式都是一样的，但是 1000 个，意味着可以扩容到 1000 个实例上，对于一般业务来说，1000 个实例足够满足业务的需求了（BTW，网传阿里某核心业务的分布式数据库分片数量为 10000个）。

如果到了 1000 个分片依然无法满足业务的需求，这时能不能拆成 2000 个分片呢？从理论上来说是可以的，但是这意味着需要对一张表中的数据进行逻辑拆分，这个工作非常复杂，通常不推荐。

那么扩容在 MySQL 数据库中如何操作呢？其实，本质是搭建一个复制架构，然后通过设置过滤复制，仅回放分片所在的数据库就行，这个数据库配置在从服务器上大致进行如下配置：

先过滤复制 —> 业务低峰期，将业务的请求转向新的分片，完成最终的扩容操作：

![](https://secure2.wostatic.cn/static/4inrfcrtUFVUFqVNXxXJ9q/image.png?auth_key=1705912810-2rE8EFE1NP8ttdBHV4bemf-0-668879537518d33f8ba2c665d9f9cacf)

## 多活架构

订单服务和订单数据分片，通过将其部署在不同的机房，使得订单服务1 部署在机房 1，可以对分片1进行读写；订单服务 2 部署在机房 1，可以对分片 2 进行读写；订单服务 3 部署在机房 3，可以对分片 3 进行读写。

![](https://secure2.wostatic.cn/static/6QVkpCsuSM1rApDfopBrGL/image.png?auth_key=1705912810-4sfZgEGDNhTG4oPEPTH4V2-0-58d2c3cd3914bd2602f473c95acc50bd)

![](https://secure2.wostatic.cn/static/u9eLs4asGhPizyQrgV53CR/image.png?auth_key=1705912811-2ZCg3GqGHxEbGt34rTh37g-0-bd9950fcd5ac78dca3b40599b758995f)

# 索引设计

- 分布式数据库主键设计使用有序 UUID，全局唯一；
- 分布式数据库唯一索引设计使用 UUID 的全局唯一设计，避免局部索引导致的唯一问题；
- 分布式数据库唯一索引若不是分片键，则可以在设计时保存分片信息，这样查询直接路由到一个分片即可；（订单号-用户id）
- 对于分布式数据库中的全局表，可以采用冗余机制，在每个分片上进行保存。这样能避免查询时跨分片的查询。

## 查询如何定位分片

因为自增并不能在插入前就获得值，而是要通过填 NULL 值，然后再通过函数 last_insert_id()获得自增的值。所以，如果在每个分片上通过自增去实现主键，可能会出现同样的自增值存在于不同的分片上。

**所以，在分布式数据库架构下，尽量不要用自增作为表的主键**，这也是我们在第一模块“表结构设计”中强调过的：自增性能很差、安全性不高、不适用于分布式架构。

说明白了“自增主键”的所有问题，那么该如何设计主键呢？依然还是用全局唯一的键作为主键，比如 MySQL 自动生成的有序 UUID；业务生成的全局唯一键（比如发号器）；或者是开源的 UUID 生成算法，比如雪花算法（但是存在时间回溯的问题）。

总之，**用有序的全局唯一替代自增，是这个时代数据库主键的主流设计标准**，如果你还停留在用自增做主键，或许代表你已经落后于时代发展了。

对于电商的订单表 orders，其表结构如下（分片键是o_custkey，表的主键是o_orderkey）：

```SQL
CREATE TABLE `orders` (

  `O_ORDERKEY` int NOT NULL auto_increment, # 订单号，唯一，uuid

  `O_CUSTKEY` int NOT NULL, # 客户唯一标识id，可以做分片键，这样做的好处是：
  #根据用户维度进行查询时，可以在单个分片上完成所有的操作，不用涉及跨分片的访问

  `O_ORDERSTATUS` char(1) NOT NULL,   # 订单状态

  `O_TOTALPRICE` decimal(15,2) NOT NULL, # 订单价格

  `O_ORDERDATE` date NOT NULL,  # 订单日期

  `O_ORDERPRIORITY` char(15) NOT NULL,  # 订单优先级

  `O_CLERK` char(15) NOT NULL,   # 订单处理人员

  `O_SHIPPRIORITY` int NOT NULL,  # 订单发货优先级

  `O_COMMENT` varchar(79) NOT NULL, # 订单备注

  PRIMARY KEY (`O_ORDERKEY`), # 主键

  KEY (`O_CUSTKEY`) # 索引

  ......

) ENGINE=InnoDB
```

通过分片键可以把 SQL 查询路由到指定的分片，但是在现实的生产环境中，业务还要通过其他的索引访问表。

还是以前面的表 orders 为例，如果业务还要根据 o_orderkey 字段进行查询，比如查询订单 ID 为 1 的订单详情：

```Python
SELECT * FROM orders WHERE o_orderkey = 1

```

我们可以看到，由于分片规则不是分片键，所以需要查询 4 个分片才能得到最终的结果，如果下面有 1000 个分片，那么就需要执行 1000 次这样的 SQL，这时性能就比较差了。

改进的做法之一是实现一个索引表，表中只包含 o_orderkey 和分片键 o_custkey，如：

```Sql
CREATE TABLE idx_orderkey_custkey （ # **实现主键到分片键的映射，这样比如修改主键为1的记录，通过映射关系找到分片键去定位**

  o_orderkey INT

  o_custkey INT,

  PRIMARY KEY (o_orderkey)

)
```

如果这张索引表很大，也可以将其分库分表，但是它的分片键是 o_orderkey，如果这时再根据字段 o_orderkey 进行查询，可以进行类似二级索引的回表实现：先通过查询索引表得到记录 o_orderkey = 1 对应的分片键 o_custkey 的值，接着再根据 o_custkey 进行查询，最终定位到想要的数据，如

```Sql
SELECT * FROM orders WHERE o_orderkey = 1

=>

# step 1

SELECT o_custkey FROM idx_orderkey_custkey 

WHERE o_orderkey = 1

# step 2

SELECT * FROM orders 

WHERE o_custkey = ? AND o_orderkey = 1
```

这个例子是将一条 SQL 语句拆分成 2 条 SQL 语句，但是拆分后的 2 条 SQL 都可以通过分片键进行查询，这样能保证只需要在单个分片中完成查询操作。不论有多少个分片，也只需要查询 2个分片的信息，这样 SQL 的查询性能可以得到极大的提升。

通过索引表的方式，虽然存储上较冗余全表容量小了很多，但是要根据另一个分片键进行数据的存储，依然显得不够优雅。

因此，最优的设计，不是创建一个索引表，而是将分片键的信息保存在想要查询的列中，这样通过查询的列就能直接知道所在的分片信息。

如果我们将订单表 orders 的主键设计为一个字符串，这个字符串中最后一部分包含分片键的信息，如：

```Python
o_orderkey = string（o_orderkey + o_custkey）

```

那么这时如果根据 o_orderkey 进行查询：查询分片1，订单号为1000-1的记录

```Sql
SELECT * FROM Orders WHERE o_orderkey = '1000-1';
```

由于字段 o_orderkey 的设计中直接包含了分片键信息，所以我们可以直接知道这个订单在分片1 中，直接查询分片 1 就行。

同样地，在插入时，由于可以知道插入时 o_custkey 对应的值，所以只要在业务层做一次字符的拼接，然后再插入数据库就行了

当然，这里我们谈的设计都是针对于唯一索引的设计，如果是非唯一的二级索引查询，那么非常可惜，依然需要扫描所有的分片才能得到最终的结果，如：

```Sql
SELECT * FROM Orders

WHERE o_orderate >= ? o_orderdate < ?
```

## 全局表

每个分片存全部信息，等于不分片

在分布式数据库中，有时会有一些无法提供分片键的表，但这些表又非常小，一般用于保存一些全局信息，平时更新也较少，绝大多数场景仅用于查询操作。

例如 tpch 库中的表 nation，用于存储国家信息，但是在我们前面的 SQL 关联查询中，又经常会使用到这张表，对于这种全局表，可以在每个分片中存储，这样就不用跨分片地进行查询了。

# 访问分布式数据库模式

- 业务直接根据分库分表访问 MySQL 数据库节点；
- 根据中间件访问；

### 分库分表直接访问

在设计分片时，我们已经明确了每张表的分片键信息，所以业务或服务可以直接根据分片键对应的数据库信息，直接访问底层的 MySQL 数据节点，比如在代码里可以做类似的处理：

```Sql
void InsertOrders(String orderKey, int userKey...) {

  

  int shard_id = userKey % 4;

  if (shard_id == 0) {

    conn = MySQLConncetion('shard1',...);

    conn.query(...);

  } else if (shard_id == 1) {

    conn = MySQLConncetion('shard2',...);

    conn.query(...);   

  } else if (shard_id == 2) {

    conn = MySQLConncetion('shard3',...);

    conn.query(...);   

  } else if (shard_id == 3) {

    conn = MySQLConncetion('shard4',...);

    conn.query(...);   

  }

}
```

又因为业务比较多，需要访问分布式数据库分片逻辑的地方也比较多。所以，可以把分片信息存储在缓存中，当业务启动时，自动加载分片信息。比如，在 Memcached 或 Redis 中保存如下的分片信息，key 可以是分库分表的表名，value通过 JSON 或字典的方式存放分片信息：

```Python
{

  'key': 'orders',

  'shard_info' : {

    'shard_key' : 'o_custkey',

    'shard_count' : 4，

    'shard_host' : ['shard1.xxx.com','shard2.xxx.com','...']，

    ‘shard_table' : ['tpch00/orders01','tpch01/orders02','...'],

  }

}
```

## 使用中间件技术

通过分布式 MySQL 中间件，用户只需要访问中间件就行，下面的数据路由、分布式事务的实现等操作全部交由中间件完成。所以，分布式数据库中间件变成一个非常关键的核心组件。

业界比较知名的 MySQL 分布式数据库中间件产品有：ShardingShpere、DBLE、TDSQL 等。

![](https://secure2.wostatic.cn/static/qX3MZZHQnuVWRj5M1jAmGK/image.png?auth_key=1705912810-dkAPXWFu6iEvy12hLv8QPx-0-09f0ce498f571dbee979a9ce36ba39c8)

你要注意，使用数据库中间件虽好，但其存在一个明显的缺点，即多了一层中间层的访问，单个事务的访问耗时会有上升，对于性能敏感的业务来说，需要有这方面的意识和考虑。

# 分布式事务

前面我们讲了分布式数据库架构设计的一个原则，即大部分的操作要能单元化。即在一个分片中完成。如对用户订单明细的查询，由于分片键都是客户ID，因此可以在一个分片中完成。那么他能满足事务的ACID特性。

但是，如果是下面的一个电商核心业务逻辑，那就无法实现在一个分片中完成，即用户购买商品，其大致逻辑如下所示：

```Sql
START TRANSATION;

INSERT INTO orders VALUES (......);

INSERT INTO lineitem VALUES (......);

UPDATE STOCK SET COUNT = COUNT - 1 WHERE sku_id = ?

COMMIT;
```

可以看到，在分布式数据库架构下，表orders、linitem的分片键是用户ID。但是表stock是库存品，是商品维度的数据，没有用户ID的信息。因此stock的分片规则肯定与表orders和lineitem不同。

所以，上述的事务操作大部分情况下并不能在一个分片中完成单元化，因此就是一个分布式事务，它要求用户维度的表 orders、lineitem 和商品维度的表 stock 的变更，要么都完成，要么都完成不了。

## 2PC（两阶段提交）的分布式事务实现

- prepare —> binlog —> commit
- 直接先commit 在binlog 这样，binlog中断了，就无法回滚，binlog和redolog数据就不一致了
- 先保证在不同分片的事务都prepare，再开启分片事务的commit，这样就能保证不在一个分片的事物提交

![](https://secure2.wostatic.cn/static/fKczpfkcUKHofwqpbuSXy/image.png?auth_key=1705912816-wXy3HDGponXZGBAkCzwxME-0-48a2019861cb0d4f7733a5eb0dd5f44b)

2PC 要求第一段 prepare 的操作都成功，那么分布式事务才能提交，这样最终能够实现持久化，2PC 的代码逻辑如下所示：

2PC 的 Java 代码实现，可以看到只有2个参与者第一阶段 prepare 都成功，那么分布式事务才能提交。

![](https://secure2.wostatic.cn/static/ek8eYJvHbXWmPbKvm35ahN/image.png)

但是 2PC 的一个难点在于 prepare 都成功了，但是在进行第二阶段 commit 的时候，其中一个节点挂了。这时挂掉的那个节点在恢复后，或进行主从切换后，节点上之前执行成功的prepare 事务需要人为的接入处理，这个事务就称之为悬挂事务。

然而，他的缺点是，事务的提交开销变大了，从 1 次 COMMIT （本来代码只需要写一次commit）变成了两次 PREPARE 和COMMIT。而对于海量的互联网业务来说，2PC 的性能是无法接受。因此，这就有了业务级的分布式事务实现，即柔性事务。

## 柔性事务

柔性事务是指分布式事务由业务层实现，通过最终一致性完成分布式事务的工作。可以说，通过牺牲了一定的一致性，达到了分布式事务的性能要求。

业界常见的柔性事务有 TCC、SAGA、SEATA 这样的框架、也可以通过消息表实现。它们实现原理本身就是通过补偿机制，实现最终的一致性。柔性事务的难点就在于对于错误逻辑的处理。

为了讲述简单，这里用消息表作为柔性事务的案例分享。对于上述电商的核心电商下单逻辑，用消息表就拆分为 3 个阶段：

```Sql
阶段1：

START TRANSACTION;

# 订单号，订单状态

INSERT INTO orders VALUES (...) 

INSERT INTO lineitem VALUES (...)

COMMIT;

阶段2：

START TRANSACTION;

UPDATE stock SET count = count -1 WHERE sku_id = ?

# o_orderkey是消息表中的主键，具有唯一约束

INSERT INTO stock_message VALUES (o_orderkey, ... )  

COMMIT;

阶段3：

UPDATE orders SET o_orderststus = 'F' WHERE o_orderkey = ?
```

上面的柔性事务中，订单表中的列 o_orderstatus 用于记录柔性事务是否完成，初始状态都是未完成。表 stock_message 记录对应订单是否已经扣除过相应的库存。若阶段 2 完成，则柔性事务必须完成。阶段 3 就是将柔性事务设置为完成，最终一致性的确定。

接着我们来下，若阶段 2 执行失败，即执行过程中节点发生了宕机。则后台的补偿逻辑回去扫描订单表中 o_orderstatus 为未完成的超时订单有哪些，然后看一下是否在对应的表stock_message 有记录，若有，则执行阶段 3。若无，可选择告知用户下单失败。

若阶段 3 执行失败，处理逻辑与阶段 2 基本一致，只是这时 2 肯定是完成的，只需要接着执行阶段 3 即可。

对于海量的互联网业务来说，柔性事务性能更好，因此支付宝、淘宝等互联网业务都是使用柔性事务完成分布式事务的实现。

