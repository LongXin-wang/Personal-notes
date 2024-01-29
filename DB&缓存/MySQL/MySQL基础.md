- [MySQL字段](#mysql字段)
  - [MySQL数据类型](#mysql数据类型)
    - [数值类型](#数值类型)
    - [时间/日期](#时间日期)
    - [字符串](#字符串)
    - [有对象性质的](#有对象性质的)
    - [Emun](#emun)
  - [字段约束](#字段约束)
- [增删改查](#增删改查)
  - [增](#增)
  - [改](#改)
  - [删](#删)
  - [查询](#查询)
    - [条件查询](#条件查询)
    - [多表联查](#多表联查)
  - [SQL优化](#sql优化)
- [索引](#索引)
  - [索引树](#索引树)
  - [索引查询](#索引查询)
- [事务](#事务)
  - [ACID](#acid)
  - [并发-不隔离造成的一致性问题](#并发-不隔离造成的一致性问题)
    - [覆盖](#覆盖)
    - [脏读](#脏读)
    - [不可重复读](#不可重复读)
    - [幻读](#幻读)
  - [隔离级别](#隔离级别)
    - [未提交读(READ UNCOMMITTED)](#未提交读read-uncommitted)
    - [提交读(READ COMMITTED)](#提交读read-committed)
    - [可重复读(REPEATABLE READ)  —mysql做到的](#可重复读repeatable-read--mysql做到的)
      - [可串行化(SERIALIZABLE)](#可串行化serializable)
  - [多版本并发控制MVCC](#多版本并发控制mvcc)
    - [快照读（ReadView）](#快照读readview)
      - [细节](#细节)
    - [RR级别下如何解决幻读的](#rr级别下如何解决幻读的)
      - [记录锁](#记录锁)
      - [间隙锁](#间隙锁)
      - [临键锁](#临键锁)
  - [子事务](#子事务)
      - [session和transcation的区分](#session和transcation的区分)
- [主从复制](#主从复制)
  - [同步复制](#同步复制)
  - [增强半同步复制](#增强半同步复制)
  - [异步复制](#异步复制)

# MySQL字段

## MySQL数据类型

数值、日期/时间、字符串

### 数值类型

| 数据类型        | 描述                                 | 大小                                                                  | SQLAIchemy |
| --------------- | ------------------------------------ | --------------------------------------------------------------------- | ---------- |
| tinyint         | 十分小的数据                         | 1个字节                                                               |            |
| smallint        | 较小的数据                           | 2个字节                                                               |            |
| mediumint       | 中等大小的数据                       | 3个字节                                                               |            |
| int             | 标准的整数                           | 4个字节                                                               | Integer    |
| bigint          | 较大的数据                           | 8个字节                                                               |            |
| float           | 浮点数                               | 4个字节                                                               | Float      |
| double          | 浮点数                               | 8个字节                                                               | DECIMAL    |
| decimal/numeric | 字符串形式的浮点数，一般用于金融计算 | numeric(19,4),19 代表 长度为19位的数，4代表小数的精度为4，保留4位小数 | DECIMAL    |


### 时间/日期

| 数据类型  | 描述                           | 格式                  | SQLAIchemy                  |
| --------- | ------------------------------ | --------------------- | --------------------------- |
| date      | 日期格式                       | YYYY-MM-DD            | Date；datetime.date         |
| time      | 时间格式                       | HH：mm：ss            | Time；datetime.time         |
| datetime  | 最常用的时间格式               | YYYY-MM-DD HH：mm：ss | DateTime；datetime.datetime |
| timestamp | 时间戳，1970.1.1到现在的毫秒数 |                       |                             |
| year      | 年份表示                       |                       |                             |


### 字符串

char在C语言中只能存放一个字符，可以定义char a[5]来存放多个字符；

mysql中的char表示存放固定长度的，char(10)表示可以存10个字符，char最大能存储255个字符.如果该列是utf8编码,则该列所占用的字节数=字符数*3；

vachar长度范围0～65535个字节，编码为utf8,每个字符最多占3个字节,最大字符长度为21845

与其他类型不同，MySQL把每个BLOB和TEXT值当做一个独立的对象去处理。当BLOB和TEXT值太大时，InnoDB会使用专门的“外部”存储区域来进行存储，此时每个值在行内需要1~4个字节存储一个指针，然后在外部存储区域存储实际的值。 

| 数据类型 | 描述              | 大小    |
| -------- | ----------------- | ------- |
| char     | 字符串固定大小    | 0~255   |
| varchar  | 可变字符串        | 0~65535 |
| tinytext | 微型文本          | 2^8-1   |
| text     | 文本串   字符方式 | 2^16-1  |
| BLOB     | 二进制方式        |         |


### 有对象性质的

date、datetime、emun、json  取出来需要转换（sqlaichemy）

### Emun

- [enum](https://so.csdn.net/so/search?q=enum&spm=1001.2101.3001.7020)类型插入时并不区分大小写。并且采用下标的形式插入时类型限制并不严格（string 、int 类型均可）
- 查询不区分大小写
- 使用索引下标查询可以只能使用 int 类型，string 类型会导致失效
- 建议采用tinyint字段

```Python
# 表结构说明：

CREATE TABLE `test_gender` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `gender` enum('M','F') DEFAULT NULL COMMENT '性别',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4;

# 这里分别插入四个不同的状态值
# M 为正常插入、‘1’,‘2’测试 string 类型的索引下标、1 测试 int 类型的索引、f 测试小写写入
insert INTO test_gender (gender) VALUES ('M'),('1'),('2'),(1),('f')

select * from test_gender where gender=1
select * from test_gender where gender='m'
select * from test_gender where gender='M'
select * from test_gender where gender='1'  #不能用，查不出来的
```

## 字段约束

| 约束条件       | 说明                                                       |
| -------------- | ---------------------------------------------------------- |
| PRIMARY KEY    | 主键约束用于唯一标识对应的记录。 自建index                 |
| FOREIGN KEY    | 外键约束                                                   |
| NOT NULL       | 非空约束                                                   |
| UNIQUE         | 唯一性约束   自建index<br>**unique约束列可以含有多个null** |
| AUTO_INCREMENT | 主键自增加约束                                             |
| DEFAULT        |                                                            |

# 增删改查

## 增

```Bash
#向表中添加数据
insert into 表名 values(值1，值2，值3);

#对应字段添加
insert into 表名(字段, 字段, 字段...)  values(值1，值2，值3);
```

## 改

```Bash
#修改表中的某一字段全部的记录 
update  表名 set 字段 = 值;

#修改指定的记录
update  表名 set 字段 = 值 where 条件;
```

## 删

```Bash
#删除表中所有数据 区别drop
delete from 数据表名;

#删除指定数据
delete from 数据表名 where 条件;
```

## 查询

FROM --> join on—> WHERE --> GROUP BY --> HAVING -->SELECT --> ORDER BY --> LIMIT  
on是对中间结果进行筛选，where是对最终结果筛选。   
1.from  
  先选择一个表，或者说源头，构成一个结果集。从硬盘上读取表  
2.where  
  然后用where对结果集进行筛选。筛选出需要的信息形成新的结果集。  
3.group by  
  对新的结果集分组。  
4.having  
  筛选出想要的分组。跟着group by一起     
5.select    #单纯的select不会产生临时表  
  选择列。  
6.order by  
  当所有的条件都弄完了。最后排序。  
7.limit

**执行顺序：**   
先进行on的过滤, 而后才进行join  
在MySQL中执行查询命令时会自动创建临时表  
什么是临时表:
由[查询命令]在内存中生成虚拟表,被称为临时表  
order_by  group by  join  子查询、dictinct会出临时表

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401221922291.png)

### 条件查询

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition
GROUP BY column_name
HAVING aggregate_function(condition)
ORDER BY column_name
LIMIT offset, count;
```

**where**
```SQL
#使用关系运算发
select * from students where age<25;

#使用in关键字，为这几个值的
select * from students where stuid in (18,20);

#在范围之内的
select * from students where age between 10 and 18 limit 3

#使用null，null不是0，也不是空字符串
select * from students where name is not null and name like "%r%";
```

**group by...having...**  

分组字段执行顺序对于最终结果没有任何影响,GROUP BY SEX,HOME与GROUP BY HOME,SEX得到的结果完全一样  
从第二个分组字段开始,每一个分组字段操作的临时表时上一个分组字段生成的临时表
如果GROUP BY是主键或者 unique NOT NULL 时是可以查询非聚合的列的，原因是此时分组的key是主键，则每一个分组只有一条数据，因此是可以进行查询非聚合的列的。  

![group by查](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401221934606.png)

![分组虚拟表](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401221934550.png)

**dictinct**  
distinct 会对数据去重，所以一般distinct用于单字段的查询，计数等  
distinct执行时会对查询的记录进行去重，产生一张虚拟的临时表；  

### 多表联查

比如A表有a, b,c 三行，B表有b,c,d三行  

**交叉查询**  
a b c d 全有，涉及的字段补null  

**外连接**  
左查询：a,b,c均有，a没有的字段补null  
右查询：b,c,d均有，d没有的字段补null

**内连接**
b，c有

**子查询**  
```sql
select * from students where age<(select avg(age) from students);
```

**组合查询**  
使用 UNION 来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果一般为 M+N 行。  
```sql
SELECT col FROM mytable WHERE col = 1
UNION
SELECT col FROM mytable WHERE col =2;
```

## SQL优化
核心就是要利用索引，减少全表查询
![SQL优化](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401221951032.png)

# 索引

## 索引树

进行查找操作时，首先在根节点进行二分查找，找到一个 key 所在的指针，然后递归地在指针所指向的节点进行查找。直到查找到叶子节点，然后在叶子节点上进行二分查找，找出 key 所对应的 data。  
插入删除操作记录会破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。

B+树比B树的优势：多了一个叶子节点形成链表，所以查询可能是从根开始，也可能是顺着叶子链表；而B树每次都需要从根开始


## 索引查询

主键索引； 唯一索引、组合索引，普通索引（都是二级索引，辅助索引）

**Innodb 聚簇索引**  
- 辅助索引叶子节点存储的不再是行数据记录，而是主键值（二级索引的叶子节点存放的是主键值或指向数据行的指针）。首先通过辅助索引找到主键值，然后到主键索引树中通过主键值找到数据行。（两次查找）
- 联合索引：假如现在真的对name+age建立联合索引，此时 MySQL 根据会 name+age 维护一个单独的 B+ 树结构，数据依旧是存放在数据页中的，只不过是原来数据中的每条记录写的是 id=xx，现在写的是name=xx，age=xx，id=xx，不管怎么样，主键肯定会存放的。

**MylSAM 非聚簇索引**  
- 无论哪种索引都是一次查询

![索引查询](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401222014375.png)

# 事务

## ACID

![ACID](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401231632732.png)

AUTOCOMMIT  
MySQL 默认采用自动提交模式。也就是说，如果不显式使用`START TRANSACTION`语句来开始一个事务，那么每个查询都会被当做一个事务自动提交。

**在一个事物内，没有隔离性的问题，只有并发才有隔离性问题**  
**mysql默认可重复读，并发下事务隔离，互不影响，但是并发-后提交的事务会覆盖前面事务的修改结果，如果不希望这样，可以开启行锁upsert for update 这样以来相当于这条记录同一时刻只能被一个并发事务写，其它写的并发操作被阻塞**
**MVCC解决的是读写冲突，实现可重复读；mysql默认多个事务可以对同一数据并发操作（覆盖），需要行级锁实现写写冲突（写被覆盖）**  
**读写冲突：不隔离的话，脏读，不可重复读等**

## 并发-不隔离造成的一致性问题

### 覆盖

T2的修改覆盖了T1的修改，

### 脏读

T2读取T1未提交的事务，T1回滚，则就是脏数据

### 不可重复读

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同（已经commit）。  
不可重复读和脏读却别在于：脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了另一事务提交的数据。  
在某些情况下，不可重复读并不是问题，比如我们多次查询某个数据当然以最后查询得到的结果为主。但在另一些情况下就有可能发生问题，例如同一个事物前后两次查询同一个数据，期望两次读的内容是一样的，但是因为读的过程中，因为令一个数据写了该数据，导致不可重复读。

### 幻读

T1 读取某个范围的数据，T2 在这个范围内(必须是在这个范围内)插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。  
幻读和不可重复读的区别在于，不可重复是针对记录的update操作，只要在记录上加写锁，就可避免；幻读是对记录的insert操作，要禁止幻读必须加上全局的写锁(比如在表上加写锁)。

## 隔离级别

并发下，不同的隔离级别会造成不同的并发问题，需要用锁来解决

### 未提交读(READ UNCOMMITTED)

造成脏读、不可重复读、幻读  
事务中的修改，即使没有提交，对其它事务也是可见的。

###  提交读(READ COMMITTED)

造成不可重复读、幻读  
一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

### 可重复读(REPEATABLE READ)  —mysql做到的

会造成幻读  
**如果 一个事物在读的时候，禁止任何事物写**  
保证在同一个事务中多次读取同样数据的结果是一样的。

#### 可串行化(SERIALIZABLE)

强制事务串行执行。

## 多版本并发控制MVCC

**而未提交读隔离级别总是读取最新的数据行，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。**

### 快照读（ReadView）

基本思想是：在每次【更新操作】时,旧版本的数据保存在【undo log日志中】并用【回滚指针】形成一条【版本链】,读数据的时候通过【readview】，拿到【版本链】中的旧数据,解决了读-写加锁效率低的问题;快照读本质上读取的是历史数据（原理是回滚段），属于无锁查询  
版本链是来自undo log中的日志：update（update和delete） undlogo和insert undolog，insert undo log会在commit之后删除，只给未commit的提供回滚机会；

- RC： MVCC通过禁止未提交读做到，每次select建立新快照
- RR：MVCC第一次select建立新快照

#### 细节

![ReadView](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401231942899.png)

事务id是递增分配的。ReadView的机制就是在生成ReadView时确定了以下几种信息：  
- m_ids：表示在生成ReadView时当前系统中活跃的读写事务的事务id列表。
- min_trx_id：表示在生成ReadView时当前系统中活跃的读写事务中最小的事务id，也就是m_ids中的最小值。
- max_trx_id：表示生成ReadView时系统中将要分配给下一个事务的id值。
- creator_trx_id：表示生成该ReadView的事务的事务id。

**举例**  
A事务 1:00--2:00； B事务 0:00--3:00  
- 若B事务0:30时select了一次，建立了快照，后续读该版本链，A的改动B永远无法知道（即使A提交了）
- 若A事务在1:10insert了，B事务在1:30第一次select，此时B也读取不到A的改动；但若是B在2:30才第一次select，B是能读取到A得改动的（A已经提交，且在A提交之前B还未快照）



### RR级别下如何解决幻读的
**行级锁：for update**
- Record Lock，记录锁，也就是仅仅把一条记录锁上；
- Gap Lock，间隙锁，锁定一个范围，但是不包含记录本身；
- Next-Key Lock：Record Lock + Gap Lock 的组合，锁定一个范围，并且锁定记录本身。

#### 记录锁
其它事务无法修改

```SQL
mysql > begin;
mysql > select * from t_test where id = 1 for update;

```

#### 间隙锁
假设，表中有一个范围 id 为（3，5）间隙锁，那么其他事务就无法插入 id = 4 这条记录了，这样就有效的防止幻读现象的发生。

```SQL
-- 开启事务
START TRANSACTION;

-- 在 product_id 范围内放置间隙锁
SELECT * FROM products WHERE product_id > 100 and product_id < 200 FOR UPDATE;

-- 在这之后，其他事务无法在 product_id 为 100 到 200 之间的间隙插入新记录

-- 提交事务
COMMIT;
```

#### 临键锁
假设，表中有一个范围 id 为（3，5] 的 next-key lock，那么其他事务即不能插入 id = 4 记录，也不能修改 id = 5 这条记录。

```SQL
-- 开启事务
START TRANSACTION;

-- 临键锁示例
SELECT * FROM orders WHERE order_id >= 100 FOR UPDATE;

-- 在这之后，其他事务无法在 order_id 大于等于 100 的记录和其之间的间隙插入新记录

-- 提交事务
COMMIT;
```

## 子事务
#### session和transcation的区分

- session是建立会话的一种交互状态，而transaction是一组数据库操作
- Session.commit() 方法用于提交当前 Session 中的所有修改到数据库，即将所有待提交的 SQL 语句发送给数据库执行，以实现对数据库的持久化操作
- Session.close() 方法用于关闭当前 Session，即释放与数据库的连接资源，并清除 Session 中的所有缓存数据。调用 Session.close() 方法后，Session 对象将不再可用

```Python
Session = sessionmaker()
session = Session()
session.add(u1)
session.add(u2)

session.begin_nested() # establish a savepoint
session.add(u3)
session.rollback()  # rolls back u3, keeps u1 and u2

session.commit() # commits u1 and u2
```

# 主从复制

主要涉及三个线程: binlog 线程、I/O 线程和 SQL 线程。

 binlog 线程 : 负责将主服务器上的数据更改写入二进制日志中。  
 I/O 线程 : 负责从主服务器上读取二进制日志，并**写入从服务器的中继日志**relay log中。请求主库的 binlog，并将得到的 binlog 写到本地的 relay-log（中继日志）文件中；主库会生成一个 log dump 线程,用来给从库 I/O 线程传 binlog；  
 SQL 线程 : 负责读取**中继日志并重放**其中的 SQL 语句。会读取 relay log 文件中的日志，并解析成sql语句逐一执行。

1). Master 服务器会把数据变更产生的二进制日志通过 Dump 线程发送给 Slave 服务器；  
2). Slave 服务器中的 I/O 线程负责接受二进制日志，并保存为中继Relay日志；  
3). SQL/Worker 线程负责并行执行中继日志，即在 Slave 服务器上回放 Master 产生的日志

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401232032672.png)

## 同步复制

在 MySQL cluster 中特有的复制方式。  
当主库执行完一个事务，然后所有的从库都复制了该事务并成功执行完才返回成功信息给客户端。  
因为需要等待所有从库执行完该事务才能返回成功信息，所以全同步复制的性能必然会收到严重的影响。

## 增强半同步复制

**为了保证主库上的每一个BINLOG事务都能够被可靠地复制到从库**上，主库在每次事务成功提交时，并不及时反馈给前端应用用户，而是**等待至少一个从库也接收到BINLOG事务并成功写入中继日志后，主库才返回Commit**操作成功给客户端  
半同步复制保证了事务成功提交后，至少有两份日志记录，一份在主库的BINLOG日志上，另一份在至少一个从库的中继日志Relay Log上，从而更进一步保证了数据的完整性。


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401232034827.png)

## 异步复制

主节点不会主动推送数据到从节点，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理。  
主库和从库的数据之间难免会存在一定的**延迟**，这样存在一个隐患：**当在主库上写入一个事务并提交成功，而从库尚未得到主库的BINLOG日志时，主库由于磁盘损坏、内存故障、断电等原因意外宕机**，导致主库上该事务BINLOG丢失，此时从库就会损失这个事务，从而造成主从不一致。  
在异步复制时，**主库执行Commit提交操作并写入BINLOG日志后即可成功返回客户端，无需等待BINLOG日志传送给从库**